---
name: ossl-opensim-scripting
description: "OpenSimulator os* scripting: NPCs, textures, teleport."
version: 1.0.0
author: Nisaba
license: CC-BY-SA
metadata:
  hermes:
    tags: [ossl, opensimulator, os-functions, npc, dynamic-texture, notecard, teleport, threatlevel, osslEnable, prim, parcel]
    homepage: http://opensimulator.org/wiki/OSSL
    related_skills: [lsl-core-scripting, lsl-ossl-porting-and-hud-systems]
---

# OSSL Scripting (OpenSimulator)

Authoritative reference for OpenSimulator scripting with `os*` OSSL extensions on top of LSL. Scope: NPCs, dynamic/in-world textures, notecard read/write, instant teleport, avatar animation, link management, and parcel/grid administration — with the required `ThreatLevel` and `osslEnable.ini` permission for every `os*` function used.

## When to Use

Use this skill for any OpenSimulator script that needs capabilities beyond pure LSL: NPCs, dynamic/in-world textures, notecards, agent teleport, avatar animation without permission, link management, inventory, and parcel/grid info. Load `lsl-core-scripting` for pure LSL, or `lsl-ossl-porting-and-hud-systems` when porting between Second Life and OpenSim.

## Hard Constraint — Document ThreatLevel & osslEnable.ini

For **every** `os*` function in a script, the comment block must state:
- the `ThreatLevel` (None / VeryLow / Low / Moderate / High / Severe), and
- the `osslEnable.ini` requirement (whether the function is enabled and under which section).

If the target grid disables a function in `osslEnable.ini`, the script throws at runtime — so always check the destination grid's config and provide an LSL fallback where feasible.

## Prerequisites (server-side; not settable in-script)

- `OpenSim.ini` `[Scripting]`: `AllowOSFunctions = true`. Without it, **no** `os*` function runs regardless of threat level.
- `OSFunctionThreatLevel` in `[Scripting]` sets the minimum allowed threat (e.g. `OSFunctionThreatLevel = Moderate`).
- `osslEnable.ini` (region/config dir) lists each `os*` function and whether it is enabled; individual functions can be toggled.
- Each call is gated by `CheckThreatLevel(ThreatLevel, name)` at runtime (JustinCC's OSSL model).

## Threat Level Scale (lowest → highest)

- **None** — no abuse risk.
- **VeryLow / Low** — minor abuse potential.
- **Moderate** — intentional abuse may cause denial of service.
- **High** — significant risk; restrict on public grids.
- **Severe** — even casual use may cause region instability and/or data loss.

## os* Catalog with Threat Levels (condensed)

| Function | Threat | Purpose |
|---|---|---|
| `osGetLinkPrimitiveParams` | None | Multi-link constants (LINK_SET etc.) |
| `osSetPrimitiveParams` / `osGetPrimitiveParams` | None | Operate on out-of-linkset prim **by UUID** |
| `osGetPrimCount()` / `osGetPrimCount(uuid)` | None | Prim count (ignores seated avatars, works on attachments) |
| `osRegionX` / `osRegionY` / `osRegionSize` / `osRegionName` | None | Region info |
| `osPlaySound` / `osLoopSound` / `osTriggerSound` | None | Sounds (take a linknum arg) |
| `osKey2Name` | None | Key→name (empty string for objects) |
| `osRegionSay` / `osRegionSayTo` | None | Regional chat |
| `osSetDynamicTextureData` / `osSetDynamicTextureURL` | VeryLow | Dynamic / external textures |
| `osAgentIsSimStanding` | VeryLow | Whether an avatar is sim-standing |
| `osNpcGetOwner` | Low | NPC owner key |
| `osNpcSay` / `osNpcSendChat` | Low | NPC chat |
| `osNpcMoveToTarget` / `osNpcMoveAndStandAt` | Low | NPC movement |
| `osNpcLoadAppearance` / `osNpcSaveAppearance` | Low | NPC appearance notecards |
| `osOwnerSaveAppearance` / `osAgentSaveAppearance` | Low | Save avatar appearance to notecard |
| `osAvatarPlayAnimation` / `osAvatarStopAnimation` | Low | Animate avatars **without PERMISSION_TRIGGER_ANIMATION** |
| `osGetAgentList` / `osGetAgents` / `osGetAvatarList` / `osGetNPCList` | Low | Agent/NPC lists |
| `osInventory*` (add/remove/move/key/name) | Low | Object inventory manipulation |
| `osAgentGetRole` / `osUUIDToAgent` | Low | Avatar role / UUID→agent |
| `osForceCreateLink` / `osForceBreakLink` | Low | Link/unlink **without PERMISSION_CHANGE_LINKS** |
| `osParticleSystem` / `osLinkParticleSystem` | Low | Particle systems |
| `osSetStandTarget` / `osSetLinkStandTarget` | Low | Stand position when unsitting |
| `osParcelGetFlags` / `osParcelInfo` | Low | Parcel flags/info |
| `osTerrainSetHeight` | High | Terrain editing |
| `osNpcCreate` / `osNpcRemove` | High | Create / delete NPC |
| `osLocalTeleportAgent` | High | In-region teleport |
| `osTeleportAgent` | **Severe** | Cross-region agent teleport |

> Canonical authority: the destination grid's `osslEnable.ini` + `[Scripting] OSFunctionThreatLevel` in OpenSim.ini. Threat levels vary by OpenSim version; verify against the target server before shipping a script that depends on a High/Severe function on a public grid.

## NPCs (summary)

`osNpcCreate(firstname, lastname, vector pos, string appearance, integer options)` returns the NPC key; **High** threat. `appearance` is a notecard name (from `osOwnerSaveAppearance`/`osAgentSaveAppearance`) or an in-region avatar UUID. Options: `OS_NPC_OBJECT_GROUP` (needs `[NPC] NoNPCGroup = false` in OpenSim.ini), `OS_NPC_CREATOR_OWNED`, `OS_NPC_NOT_OWNED`, `OS_NPC_SENSE_AS_AGENT`. See `references/npcs.md`.

## Dynamic Textures (summary)

`key osSetDynamicTextureData(string dynamicID, string contentType, string data, string extraParams, integer timer)` — **VeryLow**. `contentType` = `"vector"` means `data` is a list of drawing instructions. `dynamicID` and `timer` params are NOT implemented (pass `""` and `0`; refresh via a timer event). `extraParams`: `width:256,height:256,alpha:0-255,bgcolor:Red,altdatadelim,setalpha,lossless:true|false`. See `references/textures.md`.

## Notecards (summary)

`osMakeNotecard(name, list lines)` (**Moderate**), `osWriteNotecard`, `osGetNotecardLine` (async → `dataserver`, like `llGetNotecardLine`). See `references/admin.md`.

## Teleport (summary)

`osTeleportAgent(agent, regionName, vector pos, vector lookat)` and overloads — **Severe**. Delays: 0.5 s if disallowed or same-region, 5 s cross-region. Use `osLocalTeleportAgent` for in-region (no cross-region delay). Restrict on public grids. See `references/admin.md`.

## Key LSL-vs-OSSL Divergences

- `osGetPrimCount` ignores seated avatars and works on attachments; `llGetNumberOfPrims` includes them.
- `osAvatarPlayAnimation` needs no permission; `llStartAnimation` needs `PERMISSION_TRIGGER_ANIMATION`.
- `osKey2Name` returns `""` for objects; `llKey2Name` works on both.
- `osGetInventoryItemKey` returns asset id; `llGetInventoryKey` returns item id.
- `osSetPrimitiveParams` acts by UUID (out-of-linkset); `llSetPrimitiveParams` acts by linknum/this.
- `osForceCreateLink`/`osForceBreakLink` need no `PERMISSION_CHANGE_LINKS`.

## Pitfalls

- Three independent gates for any `os*`: `AllowOSFunctions = true` **and** the function enabled in `osslEnable.ini` **and** the threat level satisfied. Missing any one → runtime throw.
- `osTeleportAgent` on a public grid is Severe — usually blocked. Offer an OSSL-free workaround or a sit-to-teleport.
- `osMakeNotecard`/`osGetNotecardLine` need the same line-index/EOF bookkeeping as `llGetNotecardLine`.
- Never hardcode a threat level as guaranteed — confirm on the destination grid.

## Routing — load references for detail

| Need | Load |
|---|---|
| Full ThreatLevel table + `osslEnable.ini` keys | `references/threat-levels.md` |
| NPC management + example | `references/npcs.md` |
| Dynamic texture drawing commands | `references/textures.md` |
| Notecards, teleport, prim/parcel/grid admin | `references/admin.md` |

### Key References

- OSSL Overview: http://opensimulator.org/wiki/OSSL
- OSSL vs LSL: http://opensimulator.org/wiki/OSSL_vs_LSL
- OSSL Wiki: https://www.ossl.wiki/