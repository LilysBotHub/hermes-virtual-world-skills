---
name: lsl-core-scripting
description: "Second-Life LSL scripting: states, events, permissions."
version: 1.0.0
author: Nisaba
license: CC-BY-SA
metadata:
  hermes:
    tags: [lsl, second-life, scripting, states, events, permissions, optimization, prims, llSetLinkPrimitiveParamsFast, llRequestPermissions]
    homepage: https://wiki.secondlife.com/wiki/LSL_Portal
    related_skills: [ossl-opensim-scripting, lsl-ossl-porting-and-hud-systems]
---

# LSL Core Scripting (Second Life)

Authoritative reference for **pure Second-Life LSL**. Scope: state-driven architecture, event handlers, strict typing, the 64 KB Mono script limit, the permission model, and optimization patterns that keep simulator script time low.

## When to Use

Reach for this skill whenever the task is a Second-Life-native LSL script: building or fixing state-driven objects, event handlers, permissions, prim/link work, listeners, timers, or optimizing LPS. Use it as the source of truth for pure LSL. Do NOT use it for OpenSim `os*` functions — load `ossl-opensim-scripting` for that, or `lsl-ossl-porting-and-hud-systems` when moving code between platforms.

## Hard Constraint — Strict LL Compliance

- **Never use any `os*` function.** No `osSetPrimitiveParams`, `osNpcCreate`, etc. If a task needs OSSL, load `ossl-opensim-scripting` instead.
- Target the **Second Life** runtime, not OpenSim. Do not rely on OSSL-only behavior.
- Do not ship script stubs or `// TODO` placeholders — always a complete, working script.

## State Model

- Every script has exactly one `default` state and zero or more custom states.
- Execution is **event-driven**; code runs only when an event fires — there is no main loop.
- Entering a state runs `state_entry` once. `default`'s `state_entry` also fires after `llResetScript` / a script reset.
- `state <name>;` switches states; the old state's remaining events stop firing.
- Put setup in `state_entry`; tear down side effects (listeners, timers) in the leaving state's `state_exit` (OpenSim/older viewers may not fire `state_exit` — clean up in `state_entry` of the next state as a guard).

## Event Handler Quick Reference

| Event | Fires when | Must set up in |
|---|---|---|
| `state_entry` | State entered (script init) | `llListen`, `llSetTimerEvent`, `llRequestPermissions` |
| `state_exit` | State left | Stop listeners/timers, release controls |
| `touch_start(integer n)` | Avatar touches a prim of the linkset | Read `llDetectedKey(0)`, `llDetectedLinkNumber(0)` |
| `touch_end` / `touch_cancel` | Touch released / cancelled | Cleanup |
| `listen(integer chan, string name, key id, msg)` | Message on an open `llListen` channel | — |
| `timer()` | `llSetTimerEvent(s)` elapsed | — |
| `dataserver(key qid, string data)` | `llGetNotecardLine`/`llLoadURL` query done | Track the notecard line index |
| `http_response(key request_id, integer status, list metadata, string body)` | `llHTTPRequest` completes | Match `request_id` to the outstanding request |
| `changed(integer change)` | Object/attachment/permissions change | `CHANGE_ATTACHMENT`, `CHANGE_OWNER`, `CHANGE_TRANSFER` |
| `attach(key owner_id)` | Object attached/detached from avatar | Attach on `ATTACHMENT_FLAG` bits, reset on 0 |
| `run_time_permissions(key id, integer perm)` | Permission request answered | Check `perm & PERMISSION_*` |
| `collision(integer n)` / `collision_start` / `collision_end` | Collision with a physical object | — |
| `sensor(integer n)` / `no_sensor()` | Sensor detects / finds nothing | — |
| `link_message(integer sender, integer num, string str, key id)` | Another prim `llMessageLinked` | — |
| `object_changed` | Linked-prim change (perms/texture) | Refresh cached data |
| `rezzed` | Object rezzed from inventory | Re-init |

## Strict Typing & Data Types

- Types: `integer` (32-bit), `float` (32-bit IEEE), `string`, `key` (UUID), `vector` (x,y,z), `rotation` (x,y,z,w quaternion), `list`.
- **Implicit conversions to watch:** `float -> integer` truncates toward zero; `string -> list` splits on `\n`; `list -> string` joins with `\n`.
- `llDumpList2String(list, "\n")` / `llParseString2List(str, ["\n"], [])` round-trip cleanly.
- Prefer integer link numbers over string name lookups (`llGetLinkNumber` vs `llGetLinkName`); integer compares are cheaper.

## Permission Model (never assume; always ask)

```lsl
// Ask once, in state_entry, before you need the capability.
llRequestPermissions(owner, PERMISSION_TRIGGER_ANIMATION | PERMISSION_CHANGE_LINKS);

run_time_permissions(integer perm)
{
    if (perm & PERMISSION_TRIGGER_ANIMATION)
        llStartAnimation("sit");
    if (perm & PERMISSION_CHANGE_LINKS)
        llCreateLink(TARGET, FALSE);
}
```
- `PERMISSION_TRIGGER_ANIMATION` — `llStartAnimation`/`llStopAnimation`.
- `PERMISSION_CONTROL` — `llStartControl`/`llStopControl`.
- `PERMISSION_ATTACH` — `llAttachToAvatar`.
- `PERMISSION_CHANGE_LINKS` — `llCreateLink`/`llBreakLink`.
- `PERMISSION_TELEPORT` — `llTeleportAgent`.
- A user can **deny** permissions; `run_time_permissions` returns 0 — never hard-fail, degrade gracefully.

## Memory & Script Limits

- **Mono script limit: 64 KB per script.** Exceeding it throws `SCRIPT_MEMORY_EXCEEDED` and halts the script.
- Watch stack/heap limits; avoid deep recursion and giant in-memory lists.
- Offload big data to **notecards** (`llGetNotecardLine`) or remote storage (`llHTTPRequest`) — never hold megabytes in a list.
- Cache immutable values in globals (owner key, region name, link count) instead of recomputing.

## Optimization Patterns (keep LPS low)

- **Batch prim changes** with `llSetLinkPrimitiveParamsFast(LINK_SET, [PRIM_COLOR, ALL_SIDES, color]);` — never call per-face or per-prim in a loop.
- Use **strided lists**: keep parallel fields in one list so a change updates one call, e.g. `[name0,x0,y0,z0,name1,x1,y1,z1]`; iterate with `llList2ListStrided`.
- **Close idle listeners**: open only the `llListen` channels you need; use unique **negative channels** per script and `llListenControl(chan, FALSE)` when idle. Never leave an open `llListen` on channel 0.
- **Kill idle timers**: `llSetTimerEvent(0.0)` when work is done; reserve `timer()` for genuine polling.
- **Avoid `llSleep`** in event handlers — it freezes the whole script. Use a `timer` event for delays.
- Prefer `llOwnerSay`/`llRegionSayTo` over `llSay(0,...)` — channel-0 broadcasts cost every nearby viewer.
- Never spam `llHTTPRequest`; coalesce requests and cache responses.

## Common Pitfalls

- **Channel 0 is public**: any script listening on 0 hears everything; collisions and cross-talk. Use `~1000 + random` negative channels.
- `llListen` returns a handle; you must store it and pass it to `llListenControl`. A lost handle = listener leaks.
- `llDialog` buttons `"Close"` and `"No Permission"` must be handled or the menu hangs.
- Every `llHTTPRequest` **must** be matched in `http_response` by `request_id`.
- Don't trust `llListen` data as an avatar key for privileged actions — verify ownership.

## Routing — load references for detail

| Need | Load |
|---|---|
| Function signatures by category | `references/functions-cheatsheet.md` |
| Deep optimization (before/after) | `references/optimization.md` |
| Canonical working scripts | `references/examples.md` |

### Key References

- LSL Portal: https://wiki.secondlife.com/wiki/LSL_Portal
- Category:LSL Functions: https://wiki.secondlife.com/wiki/Category:LSL_Functions
