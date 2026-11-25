# Hypergrid-Specific LSL/OSSL Scripting (ExampleGrid Gates & HUDs)

Hypergrid teleporters, Stargates, portals, and visitor-detection HUDs. Covers cross-region /
cross-grid `osTeleportAgent`, `llMapDestination`, foreign-visitor detection, and the
XEngine/YEngine global-variable trap. See `ossl-opensim-scripting` for the full ThreatLevel
model — **`osTeleportAgent` is Severe** and is usually blocked on public grids.

## Engine Selection: XEngine vs YEngine (globals survive jumps)

OpenSim runs two script engines:
- **XEngine (Mono)** — where almost all `os*` functions live. A script using `os*` will **not
  run on a YEngine-only region**.
- **YEngine (legacy Mono / SL-compatible)** — pure LSL.

Both engines persist **global variables across a Hypergrid jump** (the script is not reset when
the agent leaves/returns), but XEngine and YEngine keep **separate** global namespaces. If a
gate must remember a counter or a foreign-visitor allow-list across jumps, **declare the
variable in both engines' global scope** (or re-derive it on `state_entry`) — relying on a
variable that only exists in one engine's memory will silently reset after a teleport.

## 1. HG Stargate / Portal (cross-grid teleport)

`osTeleportAgent` is **Severe** — requires `AllowOSFunctions = true`, the function enabled in
`osslEnable.ini`, and threat level satisfied on the destination grid. Delays: 0.5 s if
disallowed or same-region, **5 s cross-region**. This script guards the call and falls back to
`llMapDestination` (a viewer-initiated portal) if OSSL is unavailable.

```lsl
// ExampleGrid Stargate — cross-grid teleporter
// ThreatLevel: osTeleportAgent = SEVERE (osslEnable.ini + [Scripting] OSFunctionThreatLevel)
// Requires: AllowOSFunctions = true in OpenSim.ini [Scripting]
default
{
    state_entry()
    {
        // hop://<grid-host>:<port>/<Region Name>  — the destination on another grid.
        string dest = "hop://example.grid.org:8002/Home Island";
        vector  lander = <128.0, 128.0, 25.0>;   // where to drop the visitor
        vector  lookAt = <128.0, 128.0, 30.0>;    // where they face on arrival

        llListen(0, "", llGetOwner(), "");
        llOwnerSay("[Porta] touch to open the Stargate.");
    }

    touch_start(integer total_number)
    {
        key who = llDetectedKey(0);
        if (who != llGetOwner() && !llSameGroup(who))
        {
            llOwnerSay("The Stargate rejects " + llKey2Name(who) + ".");
            return;
        }

        // Prefer OSSL cross-grid teleport; the 5s delay is on the caller.
        if (osTeleportAgent(who, dest, lander, lookAt))
        {
            llOwnerSay("Porta open for " + llKey2Name(who) + " — crossing to the ExampleGrid.");
        }
        else
        {
            // OSSL blocked/threat-restricted: fall back to a viewer-initiated portal.
            llOwnerSay("OSSL unavailable here — offering a map portal instead.");
            llMapDestination("Porta Landing", lander, lander);
        }
    }
}
```

> If your grid sets `osTeleportAgent` to Severe and blocks it, ship the `llMapDestination`
> fallback (viewer chooses the portal) or a **sit-to-teleport** that asks the owner to confirm.

## 2. Foreign-Visitor Detection (for gates / UDs)

Detect whether a presence is a Hypergrid visitor from another grid, and read their home grid.
These `os*` calls are **Low** threat. `llGetAgentInfo` reports presence state; the `osGetGrid*`
calls reveal the home grid domain.

```lsl
// ExampleGrid — foreign visitor detector
// ThreatLevel: osGetGridName / osGetGridHomeURI = LOW
// Requires: AllowOSFunctions = true
default
{
    state_entry()
    {
        llListen(0, "", "", "");
        llOwnerSay("[Porta] Listening for foreign visitors.");
    }

    listener(integer channel, string name, key id, string msg)
    {
        if (msg == "!grid")
        {
            // Is this presence a Hypergrid foreign visitor?
            // osGetGridName returns "" for local agents; non-empty => foreign grid.
            string homeGrid = osGetGridName(id);
            string homeURI  = osGetGridHomeURI(id);   // e.g. "hop://grid:8002"

            if (homeGrid == "")
                llRegionSayTo(id, 0, "Welcome, local traveler of the ExampleGrid.");
            else
            {
                // Extract just the host:port from the hop:// home URI.
                string host = llDumpList2String(
                    llList2ListStrided(llParseString2List(homeURI, ["//", ":", "/"], []), 1, 1, ""),
                    "");
                llRegionSayTo(id, 0,
                    "Greetings from " + homeGrid + " (" + host + ").");
            }
        }
    }
}
```

Notes:
- `osGetGridName(key)` → home **grid display name** (`""` for local agents).
- `osGetGridHomeURI(key)` → the visitor's `hop://` home-grid URI; parse `//host:port` to get
  the domain.
- `llGetAgentInfo(agent)` → presence-state bitmask (e.g. `AGENT_SITTING`, `AGENT_AWAY`). Use it
  to gate actions on whether a foreign avatar is actually present and interactive.
- **Never trust a name/key from chat for privileged actions** — verify ownership/role first.

## 3. `llMapDestination` vs `llTeleportAgentGlobalCoords`

| Function | Scope | Hypergrid |
|---|---|---|
| `llMapDestination(name, pos, lookAt)` | Opens the viewer's map/teleport dialog at `pos`. | Viewer-initiated; works cross-grid if the destination is a valid HG map link. Preferred for public grids. |
| `llTeleportAgentGlobalCoords(agent, pos, lookAt)` | **In-region only** (SL-native). | Not a cross-grid teleport — use only to move someone within the current region. |
| `osTeleportAgent(agent, "hop://...", pos, lookAt)` | Cross-region **and** cross-grid. | **Severe** — the HG portal primitive. See script #1. |

`llTeleportAgentGlobalCoords` takes **global** region coordinates (the same X/Y/Z as
`Location` in `Regions.ini`), not local prim positions — pass `osRegionX + localX`, etc.

## HUD Integration

- Put the detector/portal logic in a **HUD** for grid-wide gating; pin to a fixed
  `ATTACHPOINT` and keep the script lean (low LPS).
- Use a **unique negative channel** per HUD + a listen group; never channel 0.
- Page long option lists; handle `llDialog` "Close" / "No Permission" or the menu hangs.
- Keep engine state minimal; re-derive cross-jump globals in `state_entry` (see Engine
  Selection above).