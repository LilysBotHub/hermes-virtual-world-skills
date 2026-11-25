---
name: lsl-ossl-porting-and-hud-systems
description: "Port LSL<->OSSL scripts; build HUDs, dialogs, HTTP bridges."
version: 1.0.0
author: Nisaba
license: CC-BY-SA
metadata:
  hermes:
    tags: [porting, hypergrid, hud, llDialog, llHTTPRequest, json, ossl, lsl, compatibility, xengine, yengine]
    homepage: http://opensimulator.org/wiki/OSSL_vs_LSL
    related_skills: [lsl-core-scripting, ossl-opensim-scripting]
---

# LSL↔OSSL Porting & HUD Systems

Authoritative guide for cross-platform LSL/OSSL work: translating scripts between Second Life and OpenSim (including hypergrid), and building interactive HUDs, dialog menus, and external HTTP/JSON web bridges.

## When to Use

Use when: porting a script between Second Life and OpenSim (or across a hypergrid); choosing between `ll*` and `os*` calls; building a HUD, a dialog-menu system, or an external HTTP/JSON data bridge; or diagnosing platform-specific behavioral differences.

## Hard Constraint

- **Always apply the OSSL-vs-LSL behavioral distinctions** (see `references/porting-differences.md`). Never assume `ll*` and `os*` behave identically.
- For a script that must run on **both** platforms, guard `os*` calls behind a platform check or provide an LSL fallback — a pure Second Life target has no `os*` and will throw on any `os*` call.
- Document the ThreatLevel / `osslEnable.ini` requirement for every `os*` call (see `ossl-opensim-scripting`).

## Porting Strategy

1. **Decide the target platform first.** Pure SL → `lsl-core-scripting` (no `os*`). OpenSim → `ossl-opensim-scripting`.
2. **For dual-platform scripts**, wrap each `os*` call with an LSL fallback or a runtime platform test, and require `AllowOSFunctions` on the OpenSim target.
3. **Compiler quirks:** XEngine (Mono) vs YEngine — most `os*` functions require the XEngine; a script using `os*` will not run on a YEngine-only region. Verify per-function availability.

## OSSL vs LSL Behavioral Differences (core)

| `ll*` (Second Life) | `os*` (OpenSim) | Key behavioral difference |
|---|---|---|
| `llGetLinkPrimitiveParams` | `osGetLinkPrimitiveParams` | Multi-link constants; returns by-side list |
| `llGetPrimitiveParams` / `llSetPrimitiveParams` | `osGetPrimitiveParams` / `osSetPrimitiveParams` | Act on out-of-linkset prim **by UUID** |
| `llCreateLink` / `llBreakLink` | `osForceCreateLink` / `osForceBreakLink` | No `PERMISSION_CHANGE_LINKS` needed |
| `llSitTarget` | `osSetStandTarget` | SitTarget = sit pos; StandTarget = unsit pos |
| `llAvatarOnSitTarget` | `osGetSittingAvatarsCount()` | OSSL counts all sitters on a linkset |
| `llPlaySound` | `osPlaySound` | OSSL sound takes a **linknum** arg |
| `llGetNumberOfPrims` | `osGetPrimCount()` | OSSL ignores seated avatars; works on attachments |
| `llStartAnimation` | `osAvatarPlayAnimation` | **No permission needed**; can animate multiple avatars |
| `llAttachToAvatar` | `osForceAttachToAvatar` | No `PERMISSION_ATTACH`; implicit target |
| `llGetAgentList` | `osGetAgents` / `osGetAvatarList` / `osGetNPCList` | Key vs name vs strided (key/pos/name) |
| `llGetInventoryKey` | `osGetInventoryItemKey` | Item id vs **asset id** |
| `llKey2Name` | `osKey2Name` | `osKey2Name` returns `""` for objects |

Full table + notes: `references/porting-differences.md`.

## HUD Architecture

- A HUD is an attachment; pin to a fixed `ATTACHPOINT` and keep its script lean (low LPS).
- **Channel discipline:** one unique negative channel per HUD + listen group; never channel 0.
- **Dialog flow:** open `llDialog` on the channel, read `listen`, handle `Close` / `No Permission`, and page long option lists.
- Keep state minimal; use `http_response` / `dataserver` queues rather than blocking.

## Dialog Menus

`llDialog` needs an open `llListen` on the same channel; responses arrive via `listen`. See `references/hud-systems.md` for paging and multi-page menus.

## HTTP/JSON Bridges

`llHTTPRequest` → `http_response`; parse with `llJsonGetValue` / `llJsonSetValue`. Watch cross-platform JSON path syntax and CORS on the server side. See `references/http-json.md`.

## Hypergrid Caveats

- Cross-region/cross-grid teleport: `osTeleportAgent` is Severe; plan for 5 s delays and grid-name resolution.
- Region coordinates: `osRegionX` / `osRegionY` vs `llGetRegionCorner`; prim UUIDs are grid-global.
- Asset copies: texture/asset UUIDs may differ between grids; prefer inventory notecards for portable assets.

## Pitfalls

- An `os*` call on a YEngine-only or `AllowOSFunctions=false` region throws — always provide an LSL fallback for dual-platform scripts.
- `llGetNumberOfPrims` counts seated avatars; `osGetPrimCount` does not — pick per platform.
- `osKey2Name` returns `""` for objects; use `llKey2Name` for cross-platform name lookups.
- JSON path syntax and `llHTTPRequest` behavior can differ between XEngine and YEngine — test round-trips on the target.

## Routing — load references for detail

| Need | Load |
|---|---|
| Full OSSL-vs-LSL distinction table | `references/porting-differences.md` |
| HUD + dialog-menu system | `references/hud-systems.md` |
| HTTP/JSON web bridge | `references/http-json.md` |

### Key References

- OSSL vs LSL: http://opensimulator.org/wiki/OSSL_vs_LSL
- OSSL Wiki: https://www.ossl.wiki/