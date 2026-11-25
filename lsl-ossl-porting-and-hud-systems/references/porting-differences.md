# OSSL vs LSL Behavioral Differences

The canonical distinction table (from the OpenSimulator OSSL-vs-LSL guide) plus cross-engine notes. **Never assume `ll*` and `os*` behave identically** — verify each before shipping a dual-platform script.

## Comparison Table

| `ll*` (Second Life) | `os*` (OpenSim) | Key behavioral difference |
|---|---|---|
| `llGetLinkPrimitiveParams` | `osGetLinkPrimitiveParams` | Allows `LINK_SET` / `LINK_ALL_CHILDREN`; returns a by-side list of properties |
| `llSetLinkPrimitiveParams` | *(none)* | No OSSL version needed — `llSetLinkPrimitiveParams` accepts all `LINK_*` flags |
| `llGetPrimitiveParams` / `llSetPrimitiveParams` | `osGetPrimitiveParams` / `osSetPrimitiveParams` | Act on an out-of-linkset primitive **by UUID** instead of link number |
| *(none)* | `osSetProjectionParams` | Same as `llSetLinkPrimitiveParams(PRIM_PROJECTOR)` but works outside the linkset |
| `llCreateLink` / `llBreakLink` / `llBreakAllLinks` | `osForceCreateLink` / `osForceBreakLink` / `osForceBreakAllLinks` | Do **not** require `PERMISSION_CHANGE_LINKS` |
| `llParticleSystem` / `llLinkParticleSystem` | `osParticleSystem` / `osLinkParticleSystem` | OSSL variants take link arguments |
| `llSitTarget` / `llLinkSitTarget` | `osSetStandTarget` / `osSetLinkStandTarget` | `SitTarget` = sit position; `StandTarget` = position when unsitting |
| `llAvatarOnSitTarget` / `llAvatarOnLinkSitTarget` | `osGetSittingAvatarsCount()` / `osGetSittingAvatarsCount(uuid)` | OSSL returns the **total** sitters on a linkset; LSL applies only to the sittarget and can't operate outside the linkset |
| `llPlaySound` / `llLoopSound` / `llTriggerSound` | `osPlaySound` / `osLoopSound` / `osTriggerSound` | OSSL sound functions take a **linknum** argument |
| `llLinkPlaySound` | *(universal)* | `SOUND_PLAY` / `SOUND_LOOP` / `SOUND_TRIGGER` / `SOUND_SYNC` |
| `llGetNumberOfPrims` / `llGetObjectPrimCount` | `osGetPrimCount()` / `osGetPrimCount(uuid)` | OSSL **does not count seated avatars** and works on attachments |
| `llStartAnimation` / `llStopAnimation` | `osAvatarPlayAnimation` / `osAvatarStopAnimation` | **No `PERMISSION_TRIGGER_ANIMATION` needed**; OSSL can animate multiple avatars |
| `llAttachToAvatar` / `llDetachFromAvatar` | `osForceAttachToAvatar` / `osForceDetachFromAvatar` | **No `PERMISSION_ATTACH` needed**; target avatar is implicit |
| `llGetAgentList` | `osGetAgents` / `osGetAvatarList` / `osGetNPCList` | `llGetAgentList` = keys; `osGetAgents` = names; `osGetAvatarList` = strided (key, pos, name); `osGetNPCList` = same but NPCs |
| `llGetInventoryKey` / `llGetInventoryName` | `osGetInventoryItemKey` / `osGetInventoryName` | `llGetInventoryKey` = item id, `osGetInventoryItemKey` = **asset id**; `llGetInventoryName` from (index,type), `osGetInventoryName` from item id |
| `llKey2Name` | `osKey2Name` | `llKey2Name` works on avatars **and** objects; `osKey2Name` returns `""` for objects |

## Additional Divergences

| Capability | `ll*` (Second Life) | `os*` (OpenSim) |
|---|---|---|
| NPC management | *(none)* | `osNpcCreate` / `osNpcSay` / `osNpcMoveToTarget` / `osNpcRemove` |
| Instant agent teleport | `llTeleportAgent` (LL grid only) | `osTeleportAgent` (Severe) / `osLocalTeleportAgent` |
| Notecard write | *(none)* | `osMakeNotecard` / `osWriteNotecard` (read: `osGetNotecardLine`) |
| Dynamic texture | *(none)* | `osSetDynamicTextureData` / `osSetDynamicTextureURL` |
| Avatar role | *(none)* | `osAgentGetRole` / `osUUIDToAgent` |
| Parcel flags | `lsGetParcelFlags` | `osParcelGetFlags` |

## Compiler Quirks: XEngine vs YEngine

- **XEngine** is the newer Mono-based engine; **YEngine** is the older engine. Most `os*` functions require the **XEngine** — a script using `os*` will not run on a YEngine-only region.
- Verify per-function availability on the target engine before relying on an `os*` call.
- `llHTTPRequest` / `http_response` parity differs subtly between engines — test the round-trip on the target.
- Some `os*` functions were added only in later OpenSim releases; pin the minimum OpenSim version in the script header comment.

## JSON Path Notes

- `llJsonGetValue(json, [path])` and `llJsonSetValue(json, [path], value)` use a **list path** (e.g. `["data", "name"]` or `["items", 0, "id"]`).
- Path syntax and escaping can differ between XEngine and YEngine — always round-trip-test JSON on the target grid.
- Prefer `llJsonKeys` to enumerate object keys before indexing by index.