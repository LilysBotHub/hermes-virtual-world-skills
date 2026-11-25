# OSSL Threat Levels & osslEnable.ini

## Threat Level Scale (lowest → highest)

| Level | Meaning |
|---|---|
| None | No abuse risk. |
| VeryLow | Minimal abuse potential. |
| Low | Low abuse potential. |
| Moderate | Intentional abuse may cause denial of service. |
| High | Significant risk; restrict on public grids. |
| Severe | Even casual use may cause region instability and/or data loss. |

> **Canonical authority:** the destination grid's `osslEnable.ini` plus `[Scripting] OSFunctionThreatLevel` in OpenSim.ini. Threat levels vary by OpenSim version and module configuration. Always verify against the target server before shipping a script that depends on a High or Severe function on a public grid. The values below are the common OpenSim defaults.

## None

| Function | Notes |
|---|---|
| `osGetLinkPrimitiveParams(linknum, [flags])` | Returns a by-side property list; supports LINK_SET / LINK_ALL_CHILDREN |
| `osSetPrimitiveParams(uuid, [params])` | Operates on an out-of-linkset prim by UUID |
| `osGetPrimitiveParams(uuid, [flags])` | Read params by UUID |
| `osGetPrimCount()` / `osGetPrimCount(uuid)` | Ignores seated avatars; works on attachments |
| `osRegionX` / `osRegionY` / `osRegionSize` / `osRegionName` | Region info |
| `osGetRegionHandle` | Region handle |
| `osPlaySound` / `osLoopSound` / `osTriggerSound` | Take a linknum argument |
| `osKey2Name(key)` | Empty string for objects (use `llKey2Name`) |
| `osRegionSay` / `osRegionSayTo` | Regional chat |
| `osBreakAllLinks` | Break all links in the set |
| `osUUIDToRegion` | UUID→region |

## VeryLow

| Function | Notes |
|---|---|
| `osSetDynamicTextureData` | Render a generated texture |
| `osSetDynamicTextureURL` | Show an external URL image |
| `osSetDynamicTextureURLBlend` / `BlendFace` | Blended external images |
| `osSetDynamicTextureDataBlend` / `DataBlendFace` | Blended generated textures |
| `osAgentIsSimStanding` | Whether an avatar is sim-standing |

## Low

| Function | Notes |
|---|---|
| `osNpcGetOwner` | Owner key of an NPC |
| `osNpcSay` / `osNpcSendChat` | NPC chat |
| `osNpcMoveToTarget` / `osNpcMoveAndStandAt` | NPC movement |
| `osNpcLoadAppearance` / `osNpcSaveAppearance` | NPC appearance notecards |
| `osOwnerSaveAppearance` / `osAgentSaveAppearance` | Save avatar appearance to a notecard |
| `osAvatarPlayAnimation` / `osAvatarStopAnimation` | Animate avatars **without** PERMISSION_TRIGGER_ANIMATION |
| `osGetAgentList` / `osGetAgents` / `osGetAvatarList` / `osGetNPCList` | Agent/NPC lists (osGetAvatarList is strided key/pos/name) |
| `osInventory` / `osInventoryAddItem` / `osInventoryRemoveItem` / `osInventoryMoveItem` | Object inventory manipulation |
| `osGetInventoryItemKey` / `osGetInventoryName` | Asset-id inventory lookups |
| `osAgentGetRole` / `osUUIDToAgent` | Avatar role / UUID→agent |
| `osForceCreateLink` / `osForceBreakLink` | Link/unlink **without** PERMISSION_CHANGE_LINKS |
| `osParticleSystem` / `osLinkParticleSystem` | Particle systems |
| `osSetStandTarget` / `osSetLinkStandTarget` | Stand position when unsitting |
| `osSetProjectionParams` | Camera projection |
| `osParcelGetFlags` / `osParcelInfo` / `osParcelGetName` | Parcel flags/info |
| `osTerrainGetHeight` | Read terrain height |

## Moderate

| Function | Notes |
|---|---|
| `osMakeNotecard(name, list lines)` | Create a notecard from a list of strings |

## High

| Function | Notes |
|---|---|
| `osNpcCreate` | Create an NPC (High) |
| `osNpcRemove` | Delete an NPC (High) |
| `osLocalTeleportAgent` | In-region teleport |
| `osInstantTeleport` | Bypasses normal teleport (High) |
| `osTerrainSetHeight` | Edit terrain |

## Severe

| Function | Notes |
|---|---|
| `osTeleportAgent` | Cross-region agent teleport — **Severe**; restrict on public grids |

## osslEnable.ini

- **Location:** the region's configuration directory (alongside `OpenSim.ini` / `Hypergrid.ini`).
- **Format:** one line per function, `true` to enable, `false` to disable, e.g.:
  ```ini
  osTeleportAgent = true
  osNpcCreate = true
  osSetDynamicTextureData = true
  ```
- **Global gate:** `OpenSim.ini` `[Scripting]` `AllowOSFunctions = true` is required first; without it no `os*` runs.
- **Minimum threat:** `[Scripting] OSFunctionThreatLevel = Moderate` (or None/Low/High/Severe) sets the floor; functions above the floor are rejected.
- **Enable a disabled function:** set `osFunctionName = true` in `osslEnable.ini` and **restart the region** for it to take effect.
- **Safety:** on public grids, keep High/Severe functions (`osTeleportAgent`, `osNpcCreate`, `osInstantTeleport`) disabled to avoid abuse and DoS.