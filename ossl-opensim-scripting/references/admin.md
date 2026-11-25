# Notecards, Teleport, Prim / Parcel / Grid Admin

## Notecards

| Function | Threat | Purpose |
|---|---|---|
| `osNotecardExists(string name)` | None | Whether a notecard exists |
| `osMakeNotecard(string name, list lines)` | Moderate | Create a notecard from a list of strings |
| `osGetNotecardLine(string name, integer line)` | None | Async read → `dataserver` (like `llGetNotecardLine`) |
| `osWriteNotecard(string name, list lines)` | Low | Overwrite a notecard |

**Bookkeeping:** `osGetNotecardLine` works exactly like `llGetNotecardLine` — request line 0 in `state_entry`, track a line index, accumulate in `dataserver`, and stop on `EOF`.

```lsl
default
{
    string  gCard;
    integer gLine;
    list    gLines;

    state_entry()
    {
        osMakeNotecard("greetings", ["Alpha", "Beta", "Gamma"]);   // Moderate
        gCard = "greetings";
        gLine = 0;
        llGetNotecardLine(gCard, gLine);
    }

    dataserver(key qid, string data)
    {
        if (data == EOF)
        {
            llOwnerSay("Read " + (string)llGetListLength(gLines) + " lines.");
            return;
        }
        gLines += data;
        gLine += 1;
        llGetNotecardLine(gCard, gLine);
    }
}
```

## Teleport

| Function | Threat | Purpose |
|---|---|---|
| `osTeleportAgent(agent, regionName, vector pos, vector lookat)` | Severe | Teleport to another region by name |
| `osTeleportAgent(agent, integer regionX, integer regionY, vector pos, vector lookat)` | Severe | By grid coordinates |
| `osTeleportAgent(agent, vector pos, vector lookat)` | Severe | In the same region |
| `osLocalTeleportAgent(agent, vector pos, vector lookat)` | High | In-region teleport (no cross-region delay) |

- `osTeleportAgent` delays: **0.5 s** if the teleport is disallowed or stays in the same region; **5 s** for cross-region teleports.
- For in-region / NPC teleports, prefer `osLocalTeleportAgent` (High) to avoid the cross-region delay.
- `osTeleportAgent` is **Severe** — usually blocked on public grids; provide an LSL fallback (sit-pad, notecard teleporter) or a manager-approved workflow.

```lsl
// Teleport the toucher to a point west of this object, facing west.
default
{
    touch_start(integer num)
    {
        key avatar = llDetectedKey(0);
        vector pos = llGetPos();
        osTeleportAgent(avatar, <-10, pos.y, pos.z + 1>, <-1, 0, 0>);   // Severe
    }
}
```

## Prim Manipulation

| Function | Threat | Purpose |
|---|---|---|
| `osSetPrimitiveParams(uuid, [params])` | None | Set params on an out-of-linkset prim by UUID |
| `osGetPrimitiveParams(uuid, [flags])` | None | Read params by UUID |
| `osGetLinkPrimitiveParams(linknum, [flags])` | None | Multi-link constants (LINK_SET, LINK_ALL_CHILDREN) |
| `osForceCreateLink` / `osForceBreakLink` | Low | Link/unlink **without** PERMISSION_CHANGE_LINKS |
| `osBreakAllLinks` | None | Break all links in the set |
| `osSetProjectionParams` | Low | Camera projection params |
| `osSetStandTarget` / `osSetLinkStandTarget` | Low | Stand position when unsitting |
| `osGetPrimCount()` / `osGetPrimCount(uuid)` | None | Prim count (ignores seated avatars; works on attachments) |

## Avatar Animation (no permission needed)

| Function | Threat | Purpose |
|---|---|---|
| `osAvatarPlayAnimation(agent, anim)` | Low | Play an animation **without** PERMISSION_TRIGGER_ANIMATION |
| `osAvatarStopAnimation(agent, anim)` | Low | Stop an animation |
| `osAvatarGetRole` | Low | Avatar's role on the parcel |

## Inventory

| Function | Threat | Purpose |
|---|---|---|
| `osInventory` | Low | List object inventory |
| `osInventoryAddItem` / `osInventoryRemoveItem` / `osInventoryMoveItem` | Low | Manipulate object inventory |
| `osGetInventoryItemKey(key)` | Low | Asset id (**vs** `llGetInventoryKey` returns item id) |
| `osGetInventoryName(key)` | Low | Name from an asset id |

## Grid / Parcel / Region Info

| Function | Threat | Purpose |
|---|---|---|
| `osRegionX` / `osRegionY` | None | Region grid coords |
| `osRegionSize` | None | Region size vector |
| `osRegionName` | None | Region name |
| `osGetAgentList` / `osGetAgents` / `osGetAvatarList` / `osGetNPCList` | Low | Agent/NPC lists (osGetAvatarList is strided key/pos/name) |
| `osParcelGetFlags` / `osParcelInfo` / `osParcelGetName` | Low | Parcel flags/info |
| `osAgentGetRole` / `osUUIDToAgent` | Low | Avatar role / UUID→agent |
| `osTerrainGetHeight` | None | Read terrain height |
| `osTerrainSetHeight` | High | Edit terrain |

## Quick LSL-vs-OSSL Divergences

- `osGetPrimCount` ignores seated avatars and works on attachments; `llGetNumberOfPrims` includes them.
- `osAvatarPlayAnimation` needs no permission; `llStartAnimation` needs PERMISSION_TRIGGER_ANIMATION.
- `osKey2Name` returns `""` for objects; `llKey2Name` works on both.
- `osGetInventoryItemKey` returns asset id; `llGetInventoryKey` returns item id.
- `osSetPrimitiveParams` acts by UUID (out-of-linkset); `llSetPrimitiveParams` acts by linknum/this.