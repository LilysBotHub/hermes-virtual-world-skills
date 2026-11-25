---
name: opensim-hypergrid-ops
description: "OpenSimulator Hypergrid ops: exact .ini file/section mapping (ROBUST grid vs Standalone), hop:// addressing & map linking, HG-aware LSL/OSSL teleporter/gate/HUD scripting, and teleport/asset/visitor troubleshooting."
version: 1.0.0
author: Nisaba
license: CC-BY-SA
metadata:
  hermes:
    tags: [hypergrid, opensimulator, robust, standalone, ini, gatekeeper, teleport, hop, uui, ossl, xengine, yengine, troubleshooting, 4096, map-link]
    homepage: http://opensimulator.org/wiki/Hypergrid
    related_skills: [ossl-opensim-scripting, lsl-ossl-porting-and-hud-systems, lsl-core-scripting]
---

# OpenSimulator Hypergrid Ops (ExampleGrid)

Authoritative operational reference for **Hypergrid** connectivity in OpenSimulator: exact
`.ini` file & section placement, `hop://` addressing and map linking, Hypergrid-aware
LSL/OSSL scripting for gates/portal/UDs, and a decision-tree troubleshooter for teleport,
asset, and foreign-visitor failures. Scope is **Hypergrid only** — this skill does NOT cover
general simulator setup, terrain editing, or non-Hypergrid grid architecture. Load
`ossl-opensim-scripting` for `os*` threat levels, or `lsl-ossl-porting-and-hud-systems` for
cross-platform porting caveats.

## When to Use

Reach for this skill when a task involves any of: enabling or debugging Hypergrid in grid
(ROBUST) or Standalone mode; resolving `hop://` addresses or map-link failures; building or
fixing HG teleporters, Stargates, portals, or visitor-detection HUDs; or diagnosing
"Region Not Found", black-screen teleports, inventory/texture loss on jump, or "Access
Denied" for foreign visitors on the **ExampleGrid**.

## Hard Constraints

- **Always name the exact file and section** for every Hypergrid setting. Hypergrid lives in
  different files depending on architecture: `bin/Robust.HG.ini` +
  `bin/config-include/GridCommon.ini` for grid mode, `bin/OpenSim.ini` +
  `bin/config-include/StandaloneCommon.ini` for standalone. Mixing these up is the #1 cause
  of "hypergrid is not working".
- **Document ThreatLevel + `osslEnable.ini` for every `os*` call** (see `ossl-opensim-scripting`).
  `osTeleportAgent` is **Severe** — usually blocked on public grids; prefer `llMapDestination`
  or a sit-to-teleport, and always give an LSL fallback.
- **Never ship script stubs or `// TODO` blocks.** Provide complete, working scripts.
- **Distinguish standard LSL from OpenSim-specific OSSL/HG behavior.** `llTeleportAgentGlobalCoords`
  is in-region only; cross-region/cross-grid needs `osTeleportAgent` with a `hop://` URI.
- **4096-region rule:** no viewer can teleport beyond 4096 grid units from the origin region.
  Map links placed farther than 4096 units fail the viewer's teleport-distance check — this is
  a viewer limitation, not a server bug. See `references/addressing-maplink.md`.

## Routing — load the matching reference for the task

| Need | Load |
|---|---|
| Exact `.ini` file & section mapping (grid vs standalone vs region) | `references/config-mapping.md` |
| `hop://` URI parsing, `link-region`/`unlink-region`, the 4096 rule | `references/addressing-maplink.md` |
| HG teleporter / Stargate / portal / HUD scripts, foreign-visitor detection, XEngine vs YEngine | `references/scripting.md` |
| Decision tree for teleport / map-link / inventory / visitor failures | `references/troubleshooting.md` |

### Key References

- Installing & Running Hypergrid: http://opensimulator.org/wiki/Installing_and_Running_Hypergrid
- Hypergrid overview & addressing: http://opensimulator.org/wiki/Hypergrid
- Hypergrid 2.0 & Security: http://opensimulator.org/wiki/Hypergrid_2.0 , http://opensimulator.org/wiki/Hypergrid_Security
- Teleports & protocol: http://opensimulator.org/wiki/Teleports , http://opensimulator.org/wiki/Teleport_Protocol
- OSSL reference: http://opensimulator.org/wiki/OSSL