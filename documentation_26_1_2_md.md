# Chickens & Roost – Documentation for Minecraft 26.1.2 (NeoForge)

Contents: [1. Quick Start](#1-quick-start) · [2. chicken_type File](#2-chicken_type-file--all-fields) ·
[3. Items](#3-items) · [4. Breeding](#4-breeding-parents-tier-and-chance) ·
[5. Laying Time](#5-laying-time) · [6. Spawning](#6-natural-spawning) · [7. Properties](#7-properties) ·
[8. Textures and Names](#8-textures-icons-and-names) · [9. Disabling and Overriding](#9-disabling-and-overriding-chickens) ·
[10. Configs](#10-config-files) · [11. Built-in List](#11-built-in-chickens) · [12. Troubleshooting](#12-troubleshooting) ·
[13. Version Details](#13-version-2612-details)

## About the Mod

**Chickens & Roost** combines the *Chickens Mod* (1.12.2) and *Roost* into a single mod. Chickens are defined via **datapacks**.

- **Chickens** (47 built-in + custom ones from datapacks): lay resources, breed, and have Growth / Gain / Strength stats.
- **Henhouse** (6 wood types), **Roost**, **Breeder**, **Collector** (collects items from Roost).
- **Egg Nest** (Hatchery mod): right-clicking with an egg places it in the nest; hatches after a short duration. Accelerated by redstone lighting and sitting chickens.
- **Catcher**, **Analyzer**, throwable colored eggs, liquid eggs, spawn eggs.
- **Chicken Item** = acts as spawn egg and container item for Roost/Breeder structures.
- **JEI** (optional): Breeding, Laying, and Drops categories.

## Differences Between Versions

| | 1.19.2 | 1.21.1 | 26.1.2 |
|---|---|---|---|
| Loaders | Forge (Fabric planned) | NeoForge + Fabric | NeoForge (Fabric planned) |
| Java | 17 | 21 | 25 |
| Item Data | NBT | Components | Components |
| `lay_item` / `drop_item` | `id`, `count`, `nbt` | `id`, `count`, `components` | `id`, `count`, `components` |
| Chicken Registry | Custom loader (`/reload`) | Datapack registry (rejoin world) | Datapack registry (rejoin world) |
| Name without lang entry | Formatted name from ID | Formatted name from ID | Formatted name from ID |
| Baby Chicken Model | Vanilla | Legacy appearance | Legacy appearance (custom model) |
| Biomes in `spawn.biomes` | 1.19.2 biomes | 1.21.1 biomes | 26.1.2 biomes |

## 1. Quick Start

Minimal chicken (`data/mypack/chickens/chicken_type/salt.json`):

```json
{ "lay_item": { "id": "minecraft:sugar" } }
```

Obtain in-game via:
`/give @s chickens:chicken[chickens:chicken_data={type:"mypack:salt"}]`.

**Package Structure**:

```
mypack/
├─ pack.mcmeta
├─ data/mypack/chickens/chicken_type/salt.json
└─ assets/mypack/
   ├─ lang/en_us.json
   └─ textures/
      ├─ entity/chicken/salt.png
      └─ item/chicken/salt.png
```

`pack.mcmeta`:

```json
{ "pack": { "description": "My Chickens", "pack_format": 61 } }
```

*Note: Ensure `pack_format` matches the required value for version 26.1.2.*

## 2. chicken_type File – All Fields

| Field | Type | Default | Description |
|---|---|---|---|
| `lay_item` | item | **required** | Lay item definition → [ch. 3](#3-items) |
| `drop_item` | item | = `lay_item` | Entity drop item |
| `background_color` | color | `#FFFFFF` | Background tint |
| `foreground_color` | color | `#FFFFFF` | Main tint color |
| `texture` | path | `<ns>:textures/entity/chicken/<name>.png` | In-world texture |
| `parents` | 2 IDs | none | Parents pair |
| `breeding_chance` | 0–100 | by tier | Breeding percentage → [ch. 4](#4-breeding-parents-tier-and-chance) |
| `lay_time_coefficient` | number | `1.0` | Lay interval modifier |
| `lay_time` | `{min,max}` | none | Explicit lay timing |
| `properties` | list | `[]` | `fire_immune`, `explosion_immune`, `drowning_immune`, `freeze_immune`, `hover` |
| `spawn` | object | none | Natural spawning configuration → [ch. 6](#6-natural-spawning) |
| `enabled` | bool | `true` | Enables/disables chicken |

## 3. Items

Uses component format:

```json
{ "id": "minecraft:iron_ingot", "count": 3 }
```

Liquids and dye-laying behaviors mirror 1.21.1 configurations.

## 4. Breeding: parents, tier and chance

Refer to standard breeding mechanics: parent matching, tier calculations, and manual weighting overrides via `breedingChances` in `chickens-common.toml`.

## 5. Laying Time

Calculated using `min = max(1, 6000 × tier × lay_time_coefficient)`. Override using `"lay_time": { "min": 9000, "max": 15000 }`.

## 6. Natural Spawning

Uses JSON `spawn` block or `chickens-spawns.toml` overrides. Compatible with 26.1.2 biome registrations.

## 7. Properties

Supports `fire_immune`, `explosion_immune`, `drowning_immune`, `freeze_immune`, and `hover`.

## 8. Textures, Icons, and Names

- Entity textures located in `assets/<ns>/textures/entity/chicken/`.
- Item models configured via item definitions in `assets/chickens/items/*.json` and `assets/<ns>/textures/item/chicken/`.

## 9. Disabling and Overriding Chickens

Disabling base chickens via `"enabled": false` propagates down inheritance chains. Datapack loading priority determines overrides.

## 10. Config Files

Includes `chickens-common.toml` for mechanics and `chickens-spawns.toml` for world generation.

## 11. Built-in Chickens

Contains standard set of 47 custom chickens plus vanilla chicken integration.

## 12. Troubleshooting

Consult debug logs for registry errors. Ensure world reload upon datapack changes.

## 13. Version 26.1.2 Details

**Status:** NeoForge module operational. **Fabric** module pending.

- Environment: **Java 25**, Gradle 9.x, **NeoForge 26.1.2.x** (`[26.1.2,)`).
- Item declarations located in `assets/chickens/items/*.json`. Custom tint source implementations (`ModTintSources`) handle egg rendering.
- `ChickenItemRenderer` manages chicken item model rendering.
- Custom render states and submission pipelines handle Roost and Nest rendering.
- Includes updated localization files (`en_us.json`, `pl_pl.json`).

### Building (26.1.2)

- `./gradlew :common:compileJava`
- `./gradlew :neoforge:runClient`