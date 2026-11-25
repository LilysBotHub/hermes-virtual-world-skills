# Hypergrid Troubleshooting Decision Tree

Grounded in the OpenSim teleport protocol (see `Teleport_Protocol`). Timeouts that matter:
the viewer **gives up after ~140 s** if the source sim never responds to `TeleportLocationRequest`
("Requesting Teleport..."); the source sim **cancels after 20 s** when the destination's
`UpdateAgent`/`ReleaseAgent` callback never completes (protocol stages **T4** `UpdateAgent` and
**T7** `ReleaseAgent`). Same-sim `UpdateAgent` waits for the viewer's child connection;
cross-sim it does not delay.

## A. Teleport Fails / Black Screen / Timeout

Symptom: progress bar stuck ("Requesting Teleport…"), black screen, or viewer returns to source.

1. **`ExternalHostName` DNS / NAT loopback** (`Regions.ini`)
   - The destination sim's `ExternalHostName` must be a **public FQDN** that the *source* sim
     and viewer can resolve. A LAN IP or `127.0.0.1` → source can't reach destination → black
     screen / 20 s cancel.
   - On a NAT box, confirm **hairpinning/loopback** works (both sims reachable via the same FQDN).
2. **Firewall TCP/UDP ports**
   - ROBUST/Gatekeeper **8002** (TCP), region **UDP port** (viewer circuit), region **HTTP/TCP
     9000+** (capabilities, event queue, `UpdateAgent`). Any blocked port = teleport stalls.
   - Verify with `nc -vz <host> <port>` from the *other* sim, not just localhost.
3. **Protocol mismatch: `SIMULATION/0.2` vs `SIMULATION/0.1`**
   - Default is `SIMULATION/0.2` (0.7.6+). If the destination only speaks `0.1`, force interop:
     - Outgoing: `[EntityTransfer] MaxOutgoingTransferVersion = "SIMULATION/0.1"` in `OpenSim.ini`.
     - Incoming: `[SimulationService] ConnectorProtocolVersion = "SIMULATION/0.1"` in
       `GridCommon.ini` / `StandaloneCommon.ini`.
4. **`UpdateAgent` / `ReleaseAgent` 20 s timeout**
   - Log `[ENTITY TRANSFER MODULE]` / `[SCENE PRESENCE]` for "Set release callback URL" and the
     matching `ReleaseAgent`. Missing `ReleaseAgent` from the destination → 20 s cancel.
   - A `ReleaseAgent` that never arrives (destination unreachable from source, region refusing
     users) leaves the viewer "teleported" but unmoveable — relog from the source.
5. **`AllowHypergridMapSearch = false`** (`[GridService]`) blocks map/address-bar resolution —
   but pre-existing bookmarks still resolve. Not a transport failure; distinguish the two.

## B. "Region Not Found" / Map Link Greyed Out

1. **4096-region distance rule** — the destination `Location X/Y` must be **within 4096 grid
   units** of the origin region's X/Y. Beyond that the viewer's teleport-distance check fails
   regardless of connectivity. Chain hubs every ≤4096 units.
2. **`GatekeeperService` unreachable externally** — from a foreign host, the grid's gatekeeper
   `hop://host:8002` must resolve and accept the login handshake. Test the FQDN+port directly.
3. Confirm the map link was actually placed (`link-region` in the sim console) and the region
   name/port in the `hop://` URI match exactly.
4. `AllowHypergridMapSearch = false` greys out map search — re-enable if map links must resolve
   via the main map.

## C. Inventory / Outfit / Texture Loss on Jump

1. **`HGAssetService` / `HGInventoryService` URL mismatch** — `HomeURI` in `Robust.HG.ini` /
   `GridCommon.ini` must point at the correct ROBUST HTTP endpoint. A wrong/mismatched URL →
   assets fail to pull → missing textures/outfit after the jump.
2. **UUI resolution failure** — the Universal Unique ID for the visitor's assets/inventory can't
   be resolved; check the login/asset service URLs and DNS.
3. **Creator / export permission restrictions**
   - `[HGAssetService] DisallowExport = "LSLText, LSLBytecode"` (and `DisallowImport`) can block
     certain asset types.
   - Texture/asset **UUIDs may differ between grids**; prefer inventory notecards for portable
     assets (see `lsl-ossl-porting-and-hud-systems`).
4. Verify inventory is pulled **on demand** from the grid inventory service (post-legacy-model
   behavior) — a stale full-load assumption is not valid.

## D. Visitor Rejected — "Access Denied"

Hypergrid 2.0 security lives in `Robust.HG.ini`. The real parameter names:

| Intended control (task term) | Actual OpenSim parameter | Location | Effect |
|---|---|---|---|
| "DisallowForeigners" | `[GatekeeperService] ForeignAgentsAllowed = false` | `Robust.HG.ini` | Blocks **all** foreign visitors; local users can still leave. Whitelist with `AllowExcept`. |
| Local users allowed to leave | `[UserAgentService] ForeignTripsAllowed_Level_x` | `Robust.HG.ini` | Per-user-level allow for outbound trips (`_Level_0`, `_Level_10`). |
| Allowlist / blacklist | `[GatekeeperService] AllowExcept` / `DisallowExcept` | `Robust.HG.ini` | Comma-separated `http://host:port` lists, applied when the above is true/false. |
| Outbound allowlist per level | `[UserAgentService] AllowExcept_Level_10` / `DisallowExcept_Level_0` | `Robust.HG.ini` | Same allow/blacklist concept, scoped by user level. |

- `AllowHypergridMapSearch` (`[GridService]`) is separate — it controls map/address resolution,
  not the access decision.
- **User-level IP verification** is enforced by the `UserAgentService`/`GatekeeperService`
  allow-except lists; there is no single "AllowUsage" switch — the access decision is the
  combination of `ForeignAgentsAllowed` + the `*Except` allowlists.
- Malicious-host considerations (copybots, piracy) are discussed in `Hypergrid_Security`;
  restrict on public grids and monitor `UserAgentService` logs.

## Fast Path Summary

| Symptom | First thing to check |
|---|---|
| Black screen / timeout | `ExternalHostName` DNS+NAT + gatekeeper port 8002 reachable from the other sim |
| 20 s cancel mid-teleport | Destination `ReleaseAgent` missing → destination sim unreachable/refusing |
| Map link greyed / "Not Found" | 4096-unit X/Y distance **and** `GatekeeperService` reachability |
| Missing textures/outfit | `HGAssetService`/`HGInventoryService` `HomeURI` mismatch |
| "Access Denied" | `ForeignAgentsAllowed` + `*Except` allowlists in `Robust.HG.ini` |