# Hermes Virtual-World Skills

Modular skills for **Hermes Agent** that cover scripting and administration for virtual-world
platforms: **Second Life** pure-LSL, **OpenSimulator** OSSL, cross-platform **LSL↔OSSL porting &
HUDs**, and **Hypergrid** operations.

These skills instruct the agent how to write, port, and troubleshoot LSL/OSSL scripts and
OpenSim/ROBUST configuration. They are written to be **grid-agnostic**: all examples use neutral
placeholders (`example.grid.org:8002`, `ExampleGrid`) and never ship real hostnames, IPs, paths,
or credentials.

## Skills

| Skill | Covers |
|---|---|
| [`lsl-core-scripting`](lsl-core-scripting/) | **Pure Second-Life LSL.** State-driven architecture, event handlers, strict typing, the 64 KB Mono limit, the permission model, and LPS-optimization patterns. Never uses any `os*` function. |
| [`ossl-opensim-scripting`](ossl-opensim-scripting/) | **OpenSimulator `os*` scripting.** NPCs, dynamic/in-world textures, notecards, instant teleport, avatar animation, link management, and parcel/grid admin — with the required `ThreatLevel` and `osslEnable.ini` gate documented for every `os*` call. |
| [`lsl-ossl-porting-and-hud-systems`](lsl-ossl-porting-and-hud-systems/) | **Cross-platform work.** Translating scripts between Second Life and OpenSim (including the hypergrid), choosing `ll*` vs `os*`, building HUDs/dialog menus, and external HTTP/JSON bridges. Includes the canonical OSSL-vs-LSL behavioral-differences table and XEngine/YEngine quirks. |
| [`opensim-hypergrid-ops`](opensim-hypergrid-ops/) | **Hypergrid operations.** Exact `.ini` file/section mapping (ROBUST grid vs Standalone), `hop://` addressing and map linking, HG-aware teleporter/portal/UD scripting, and a decision-tree troubleshooter for teleport, asset, and foreign-visitor failures. |

## Installation

Skills live in your Hermes `skills/` directory. Each skill is a folder containing a `SKILL.md`
plus any `references/`, `templates/`, or `assets/` files.

### Option A — copy this repo's skills into your Hermes profile

```bash
# 1. Clone this repo somewhere on your machine
git clone https://github.com/LilysBotHub/hermes-virtual-world-skills.git
cd hermes-virtual-world-skills

# 2. Copy the skill folders into your active Hermes skills directory
#    (resolve the real home from $HERMES_HOME; default ~/.hermes)
cp -r lsl-core-scripting ossl-opensim-scripting \
      lsl-ossl-porting-and-hud-systems opensim-hypergrid-ops \
    "$HERMES_HOME/skills/"   # e.g. ~/.hermes/skills/
```

### Option B — clone the whole repo as a skills source

```bash
git clone https://github.com/LilysBotHub/hermes-virtual-world-skills.git \
  "$HERMES_HOME/skills/virtual-world"
```

### Enabling them

- Hermes loads skills from the `skills/` directory automatically. New skills appear the next time
  the agent starts a session (they are surfaced in the skills list and loaded on demand when a
  task matches their description).
- To confirm a skill is available and correctly formed, inspect it with `skill_view(name)` — for
  example `skill_view(name='ossl-opensim-scripting')`. Its `readiness_status` must be
  `available` (the `platforms:` frontmatter only accepts OS values — `linux`, `macos`, `windows`;
  omit `platforms:` for virtual-world skills so it is not marked `unsupported`).
- No separate "enable" toggle is needed: an available skill is ready to use.

> **Profiles:** each Hermes profile has its own `skills/` directory
> (`$HERMES_HOME/profiles/<name>/skills/`). Copy into the profile you intend to use.

## Example prompts

### LSL scripting (Second Life)
> "Write a complete, Second-Life-native LSL script for a touch-to-sit chair that requests
> `PERMISSION_TRIGGER_ANIMATION`, handles permission denial gracefully, and uses a unique
> negative `llListen` channel. No `os*` functions."

### OSSL `os*` functions with `ThreatLevel` checks (OpenSim)
> "OpenSim script that creates an NPC and animates it without a permission prompt. For every
> `os*` call, document the `ThreatLevel` and the `osslEnable.ini` requirement, and confirm
> `AllowOSFunctions = true` in `OpenSim.ini [Scripting]`."

### LSL↔OSSL porting & HUDs
> "Port this Second Life controller to run on both OpenSim and Second Life: guard each `os*` call
> with an LSL fallback, note the XEngine vs YEngine requirement, and build a paginated
> `llDialog` HUD on a unique negative channel."

### Hypergrid `hop://` / teleport troubleshooting
> "A foreign visitor gets 'Access Denied' and map links are greyed out on a ROBUST Hypergrid.
> Walk the decision tree: check `ExternalHostName` DNS/NAT, gatekeeper port 8002 reachability,
> the 4096-region distance rule, and the `[GatekeeperService] ForeignAgentsAllowed` / `*Except`
> allowlists in `Robust.HG.ini`."

## Sanitization & licensing

- All examples use neutral placeholders (`example.grid.org:8002`, `ExampleGrid`). No real
  grid/hostnames, IPs, filesystem paths, or credentials are included.
- Distributed under the terms of the [MIT License](LICENSE).
