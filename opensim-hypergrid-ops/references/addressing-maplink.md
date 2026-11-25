# Hypergrid Addressing & Map Linking

## URI Formats

A Hypergrid link is shaped like a web link. Canonical form:

```
hop://<grid-host>:<port>/<Region Name>/<x>/<y>/<z>
```

Examples (from OpenSim docs):
```
hop://example.grid.org:8002/keynote 1
hop://example.grid.org:9000
```

- `hop://example.grid.org:8002/keynote 1` — region **"keynote 1"** on grid
  `example.grid.org`, gatekeeper port **8002**.
- `hop://example.grid.org:9000` — **port-only** form: destination is the grid's **default
  region** as configured in `[GridService]` of `Robust.HG.ini` (grid) or
  `StandaloneCommon.ini` (standalone).

### Legacy format
Older bookmarks/viewers use the colon-delimited form:
```
grid.domain:8002:Region Name
```
Parse it as `host:port:Region Name` (note the **trailing** colon before the name, unlike the
`hop://` form). Convert to `hop://host:port/Region Name` for modern viewers.

## Parsing / Generating a `hop://` URI (for scripts & tools)

Split on `/`:
- `hop://` → scheme
- after `//` up to first `/` → `host:port` (split on `:` for host and port)
- remaining segments → `Region Name` then `x`, `y`, `z`

Guard: a valid HG URI must start with `hop://` and contain a numeric port. If the port is
missing, the viewer assumes the grid default region.

## Console Commands (linking / unlinking remote regions)

Run in the **simulator console** (`bin/OpenSim.exe` / `./OpenSim`), not the ROBUST console:

```
link-region <x> <y> <host> <port> <region>
unlink-region <x> <y>
```

- `link-region 128 128 example.grid.org 8002 "keynote 1"` — places a map hyperlink at grid
  coordinate (128,128) pointing at the foreign region.
- `unlink-region 128 128` — removes the hyperlink at that coordinate.

Map links are what let users "choose to teleport there" and then interact with the foreign
world without logging out, keeping their inventory (per the Hypergrid concept).

## The 4096-Region Distance Rule (critical)

Most Second-Life-compatible **viewers cannot teleport more than 4096 regions** from the current
region, due to implementation/protocol limits. OpenSim enforces this at **P4** in the teleport
handshake ("Source checks destination is within max teleport distance").

Implications:
- A map link placed **more than 4096 grid units** (in `Location X`/`Y` space) from the user's
  current region will fail the viewer's teleport-distance check — the link looks fine but the
  teleport is rejected. **This is a viewer limit, not a server/config bug.**
- To reach distant regions, chain intermediate links / hubs every ≤4096 units, or have the user
  teleport to a nearby hub first.
- Keep your grid's hub coordinates clustered so any two reachable nodes stay within 4096.

## Debugging a `hop://` Address

1. Confirm scheme is `hop://` (not `http://`).
2. Confirm the port is the **gatekeeper/login** port (grid default ~8002), not the region UDP.
3. Confirm the region name matches exactly (case-sensitive, spaces included).
4. Confirm the grid host resolves via DNS to the ROBUST.
5. Confirm `GatekeeperService` on that grid is reachable externally (see `troubleshooting.md`).