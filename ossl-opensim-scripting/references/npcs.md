# NPC Management (osNpc*)

NPC = Non-Player Character. Requires `AllowOSFunctions = true`, the function enabled in `osslEnable.ini`, and a satisfied threat level.

## Catalog

| Function | Threat | Signature / Purpose |
|---|---|---|
| `osNpcCreate` | High | `key osNpcCreate(string first, string last, vector pos, string appearance, integer options)` — returns NPC key |
| `osNpcRemove` | High | `integer osNpcRemove(key npc)` — delete an NPC |
| `osNpcGetOwner` | Low | `key osNpcGetOwner(key npc)` — owner key |
| `osNpcSay` | Low | `integer osNpcSay(key npc, string msg)` — NPC says on channel 0 |
| `osNpcSendChat` | Low | `integer osNpcSendChat(key npc, string msg, int channel, ...)` — NPC chat on a channel |
| `osNpcMoveToTarget` | Low | `integer osNpcMoveToTarget(key npc, vector target, float radius)` — walk to a position |
| `osNpcMoveAndStandAt` | Low | `integer osNpcMoveAndStandAt(key npc, vector pos, vector lookat)` — move and face |
| `osNpcLoadAppearance` | Low | `integer osNpcLoadAppearance(key npc, string appearance)` — load notecard/UUID |
| `osNpcSaveAppearance` | Low | `integer osNpcSaveAppearance(key npc, string name)` — save to notecard |
| `osOwnerSaveAppearance` | Low | `integer osOwnerSaveAppearance(key owner, string name)` — save owner appearance |
| `osAgentSaveAppearance` | Low | `integer osAgentSaveAppearance(key agent, string name)` — save a logged-in agent |

## Options flags (osNpcCreate `options` arg)

- `OS_NPC_OBJECT_GROUP` — NPC takes the object's group (requires `[NPC] NoNPCGroup = false` in OpenSim.ini); enables parcel group access.
- `OS_NPC_CREATOR_OWNED` — NPC responds only to scripts owned by the creator.
- `OS_NPC_NOT_OWNED` — NPC responds to any script with OSSL permissions.
- `OS_NPC_SENSE_AS_AGENT` — NPC is detectable by `llSensor` as type AGENT.

## Example (from OpenSim wiki)

```lsl
key npc;
default
{
    state_entry()
    {
        llSay(PUBLIC_CHANNEL, "Touch to see osNpcCreate usage.");
    }

    touch_start(integer number)
    {
        key toucher = llDetectedKey(0);
        vector npcPos = llGetPos() + <1.0, 0.0, 1.0>;
        osAgentSaveAppearance(toucher, "appearance");   // save the toucher's look
        npc = osNpcCreate("ImYour", "Clone", npcPos, "appearance", OS_NPC_OBJECT_GROUP);
        state hasNPC;
    }
}

state hasNPC
{
    state_entry()
    {
        llSetTimerEvent(5.0);
    }

    timer()
    {
        llSetTimerEvent(0.0);
        key npc_owner = osNpcGetOwner(npc);
        osNpcSay(npc, "Hello world!");
    }

    touch_start(integer number)
    {
        osNpcSay(npc, "Goodbye!");
        llSetTimerEvent(0.0);
        osNpcRemove(npc);          // High threat
        npc = NULL_KEY;
        state default;
    }
}
```

## Notes

- `appearance` argument: a notecard name saved via `osOwnerSaveAppearance`/`osAgentSaveAppearance`, or the UUID of an avatar logged into the same region.
- From OpenSim 0.7.2+ (commit c4972e77), the no-options `osNpcCreate` creates an **owned** NPC; use `OS_NPC_NOT_OWNED` if you need an unowned one.
- For in-region NPC movement/teleport, prefer `osLocalTeleportAgent` (no cross-region delay).