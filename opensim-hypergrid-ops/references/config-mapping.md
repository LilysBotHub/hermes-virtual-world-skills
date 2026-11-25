# Hypergrid Configuration File & Section Mapping

Hypergrid settings live in **different files per architecture**. Getting the file right is
80% of the fix. Since OpenSim 0.8 most variables are pre-uncommented; since 0.7.6 the
`[Hypergrid]` section lets you declare `HomeURI` / `GatekeeperURI` once instead of per-service.
Variable substitution `${Const|BaseURL}:${Const|PublicPort}` means domain+port should be
declared **once** in the `[Const]` section — never repeat it.

## 1. Grid Mode (ROBUST)

Two files must agree. The ROBUST side owns the services; the Simulator side (running against
the ROBUST) owns the connectors.

### `bin/Robust.HG.ini` — ROBUST / server-side services
| Service | Role |
|---|---|
| `[GatekeeperService]` | Authorizes a viewer's login to a grid; resolves `hop://` grid host:port. Set `ForeignAgentsAllowed` (see `troubleshooting.md`). |
| `[UserAgentService]` | Local-account management; controls whether **local** users may travel to foreign grids (`ForeignTripsAllowed_Level_*`). |
| `[HGAssetService]` | Pulls/pushes assets across the HG (`HomeURI`, `DisallowExport`, `DisallowImport`). |
| `[HGInventoryService]` | Pulls inventory on demand across the HG (`My Suitcase`, `HomeURI`). |
| `[LoginService]` / `LoginURI` | Public login endpoint viewers hit; must resolve to the ROBUST FQDN + HTTPS port. |

### `bin/config-include/GridCommon.ini` — Simulator-side connectors
| Section | Role |
|---|---|
| `[HypergridConnectors]` | The simulator's Hypergrid module — tells the sim it is running in HG mode. |
| `[Hypergrid]` | `GatekeeperURI` = the ROBUST gatekeeper `hop://` URL the sim dials for cross-region/cross-grid teleports. |
| `[SimulationService]` | `ConnectorProtocolVersion = "SIMULATION/0.2"` (default) or `"SIMULATION/0.1"` for interop with older sims. |
| `[GridService]` | `AllowHypergridMapSearch` (default `false`) — toggles main-map + address-bar HG resolution. |

## 2. Standalone Mode

One machine runs both the ROBUST-equivalent and the simulator.

### `bin/OpenSim.ini` — architecture switch
In `[Architecture]`, the **only** uncommented line must select the HG standalone architecture:
```ini
[Architecture]
    ;Include-Architecture = "config-include/Standalone.ini"
    Include-Architecture = "config-include/StandaloneHypergrid.ini"   ; <-- the HGStandalone include
    ;Include-Architecture = "config-include/Grid.ini"
    ;Include-Architecture = "config-include/GridHypergrid.ini"
    ;Include-Architecture = "config-include/SimianGrid.ini"
```
> The HGStandalone architecture file is `config-include/StandaloneHypergrid.ini`. Ensure **no
> other `Include-Architecture` line is active** — a stray `Grid.ini` or plain `Standalone.ini`
> silently disables Hypergrid.

### `bin/config-include/StandaloneCommon.ini` — HG settings
- `[Hypergrid]` → set `HomeURI` and `GatekeeperURI` (the two values that used to be per-service
  before 0.7.6).
- `[LoginService]` → set the `SRV_*` addresses (`SRV_Login`, `SRV_Authentication`, etc.) to the
  public FQDN so the viewer can reach them.
- `[SimulationService]` → `ConnectorProtocolVersion` for outgoing/incoming interop.
- `[GridService]` → `AllowHypergridMapSearch`.

## 3. Region Level — `bin/Regions/Regions.ini`

Per-region block; the most common HG breakage lives here.

| Setting | Purpose |
|---|---|
| `ExternalHostName` | The **public FQDN** (not `127.0.0.1` / LAN IP) that other sims and viewers must resolve. Wrong here → NAT loopback / "Region Not Found" across grids. Ensure DNS resolves and, on a LAN, that loopback/NAT hairpinning works. |
| `UDP_PORT` | Region UDP port (viewer circuit). Must be forwarded/open. |
| `TCP_PORT` | Region HTTP port (capabilities, event queue, `UpdateAgent`). Must be forwarded/open. |
| `Location X` / `Location Y` | **Grid coordinates.** These are the map-link coordinates; the 4096-region distance rule applies between any two linked regions' X/Y. |

## Quick Placement Checklist

- [ ] **Grid:** `Robust.HG.ini` services (Gatekeeper/UserAgent/HGAsset/HGInventory/LoginURI) **and** `GridCommon.ini` connectors (`[Hypergrid]` `GatekeeperURI`, `[SimulationService]`) both configured.
- [ ] **Standalone:** `OpenSim.ini` `[Architecture]` = `StandaloneHypergrid.ini` **and** `StandaloneCommon.ini` `[Hypergrid]` + `[LoginService]` `SRV_*` set.
- [ ] **Every region:** `ExternalHostName` = public FQDN, UDP/TCP ports forwarded, `Location X/Y` set.
- [ ] `[Const]` `BaseURL`/`PublicPort` declared once; `${Const|...}` substitution used elsewhere.