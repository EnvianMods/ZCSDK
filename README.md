# Zero Company Mod SDK

The **developer** half of the Zero Company modding toolset by Envian Mods, for **STAR WARS: Zero Company** (Bit Reactor, Unreal Engine 5.6).
It builds content mods — new items, abilities, customization parts with real art, character classes and recruits, table rows, retextures — and
emits a `.zip` that **[Zero Company Mod Command](https://github.com/EnvianMods)** (the player-facing mod manager) installs like any other pak mod.
Mods that add *enumerable* content (things the game's menus, shops and pickers must list) need the small **[ZCSDK Runtime](https://github.com/EnvianMods/ZCSDK-Runtime-Release)**
on the player's side; Mod Command installs it on demand.

`latest.json` at the root of this repository describes the newest release (`version`, download `url`). The same package is on
**[Nexus Mods](https://www.nexusmods.com/games/starwarszerocompany/mods/163)**.

## What you need (developer machine)

| Prerequisite | Notes |
|---|---|
| Unreal Engine **5.6.x** | must match the game's major.minor (5.7 is rejected at load) |
| MSVC (VS Build Tools) + Windows 10 SDK 10.0.26100 + **.NET Framework 4.8.1 Developer Pack** | the one prerequisite that is easy to miss |
| `retoc` 0.1.5 | bundled with Mod Command (`ZeroCompanyModManager/tools/retoc.exe`) |
| node.js, python 3.8+ (Pillow optional — the demo art generators) | |
| the game installed | the tools read its paks; the reflection dump (`.jmap`) is made in-game with UE4SS |

`python tools/doctor.py` prints the readiness report.

## Quick start

```
copy tools\config.example.json tools\config.json        # then fill the paths
python tools/doctor.py                                  # prerequisites
node tools/zcmod-build.js --new --list                  # the recipes
node tools/zcmod-build.js --new tattoo MyFirstTattoo    # scaffold mods/MyFirstTattoo/MyFirstTattoo.json (+ demo art)
node tools/zcmod-build.js mods/MyFirstTattoo/MyFirstTattoo.json --check     # preflight
node tools/zcmod-build.js mods/MyFirstTattoo/MyFirstTattoo.json --deploy    # build + copy into the game (game closed)
```

The deliverable is `build/<Mod>_v<version>.zip`. Every recipe is a working sample under `tools/samples/` with its `_note` keys as documentation;
`tools/README.md` is the mod-def reference, `docs/SDK_GUIDE.md` the how-to, `docs/CHANGELOG.md` the history.

## Recipes (proven in the retail game)

| Recipe | What it adds |
|---|---|
| `cost`, `ability-seq` | rows in the game's composite data tables |
| `item`, `shop-item`, `reward-item`, `granted-item` | net-new inventory items (weapon mods) — sold, rewarded or granted |
| `appends`, `reward-shapes` | entries appended to the game's own list assets (reward / shop tables) |
| `ability`, `passive` | net-new abilities, effects, tags; a net-new tactical specialization |
| `color-part`, `tattoo`, `scar` | net-new customization parts — colors, and parts with real art (textures) |
| `robe`, `skirt`, `hood`, `boots` | outfit parts with NET-NEW SKINNED MESHES modelled in Blender on the shipped rig template — Tops, Legs, Headwear (hides the hair), Boots; hair via `samples/hair_mod.json` |
| `class` | a net-new character class on a net-new pre-authored recruit, with runtime recruit pins |
| `retexture` (`textures[]` with a game path) | replace any game texture (e.g. weapon paint patterns) |

## The artist path

`tools/mesh/`: rig templates (`samples/mesh/rig_HAA{M,F}_03A.glb`, the game's 632-bone rig + a stick-figure proxy), `blender_export.py`
(in-Blender exporter), `check_gltf.py --fix` (the gate: joints snapped back to the rig, inverse bind matrices rebuilt, weights, region rule,
winding), `scalp_from_mesh.py` (the game's head geometry out of a cooked mesh — hair and hoods are built on it). Materials are instances of
the game's own masters with your textures (`meshes[].material.instanceOf`). The [Zero Company Mod Studio](https://www.nexusmods.com/starwarszerocompany)
exports game meshes with skeleton and weights into Blender; `tools/studio.py` wraps its exporter — see `docs/MOD_STUDIO_BRIDGE.md`.

## Design

Two tools, one bridge: Mod Command stays a lean click-to-play app; the SDK is the separate dev tool with the Unreal + compiler environment.
They meet only at the package output. Runtime source: `tools/ue4ss-bridge` (C++ UE4SS mod) + `tools/ue4ss-loader` (Lua).
