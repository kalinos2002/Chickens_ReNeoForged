# Chickens & Roost – Documentation for Minecraft 1.21.1 (NeoForge & Fabric)

Lang: EN | [PL](dokumentacja_1_21.1_md.md)

Versions: [26.1.2](README.md) 1.21.1 | [1.19.2](documentation_1_19_2_md.md)

Contents: [1. Quick Start](#1-quick-start) · [2. chicken_type File](#2-chicken_type-file--all-fields) ·
[3. Items](#3-items) · [4. Breeding](#4-breeding-parents-tier-and-chance) ·
[5. Laying Time](#5-laying-time) · [6. Spawning](#6-natural-spawning) · [7. Properties](#7-properties) ·
[8. Textures and Names](#8-textures-icons-and-names) · [9. Disabling and Overriding](#9-disabling-and-overriding-chickens) ·
[10. Configs](#10-config-files) · [11. Built-in List](#11-built-in-chickens) · [12. Troubleshooting](#12-troubleshooting) ·
[13. Build & Execution](#13-building-and-running-1211)

---

## About the Mod

**Chickens & Roost** combines the *Chickens Mod* (1.12.2) and *Roost* into a single mod. Chickens are defined via **datapacks**.

- **Chickens** (47 built-in + custom ones from datapacks): lay resources, breed, and have Growth / Gain / Strength stats.
- **Henhouse** (6 wood types), **Roost**, **Breeder**, **Collector** (collects items from Roost).
- **Egg Nest** (from Hatchery mod): right-clicking with an egg (vanilla or colored) places it inside; after a few minutes, a chick hatches (colored egg → chicken of that type, vanilla egg → vanilla chicken). Speeds up when a chicken sits in the nest and when illuminated by a redstone lamp up to 3 blocks above. Right-clicking retrieves the egg.
- **Catcher**, **Analyzer**, colored eggs (throwable), liquid eggs, spawn eggs.
- **Chicken Item** = spawn egg and entity container for Roost / Breeder placement.
- **JEI** (optional): Breeding, Laying, and Drop categories. Datapack chickens integrate automatically.

---

## Differences Between Versions

| | 1.19.2 | 1.21.1 | 26.1.2 |
|---|---|---|---|
| Loaders | Forge (Fabric planned) | NeoForge + Fabric | NeoForge (Fabric planned) |
| Java | 17 | 21 | 25 |
| Item Data | NBT | Components | Components |
| `lay_item` / `drop_item` | `id`, `count`, `nbt` (legacy components converted) | `id`, `count`, `components` | `id`, `count`, `components` |
| Chicken Registry | Custom loader (`/reload` works) | Datapack registry (rejoin world) | Datapack registry (rejoin world) |
| Name without lang entry | Formatted name from ID | Formatted name from ID | Formatted name from ID |
| Baby Chicken Model | Vanilla | Legacy appearance | Legacy appearance (custom model) |
| Biomes in `spawn.biomes` | 1.19.2 biomes | 1.21.1 biomes | 26.1.2 biomes |

---

## 1. Quick Start

Minimal chicken (`data/mypack/chickens/chicken_type/salt.json`):

```json
{ "lay_item": { "id": "minecraft:sugar" } }
```

That's all: chicken `mypack:salt` exists, lays sugar, obtainable via:
`/give @s chickens:chicken[chickens:chicken_data={type:"mypack:salt"}]`.
All other fields are optional.

**Chicken ID** = `<namespace>:<file_name>`. Subfolders work (`mypack:ores/tin` → `ores/tin.json`); translation keys replace `/` with `.`.

**Package Structure** (datapack + resource pack combined):

```
mypack/
├─ pack.mcmeta
├─ data/mypack/chickens/chicken_type/salt.json         ← chicken definition
└─ assets/mypack/
   ├─ lang/en_us.json                                  ← name: "chicken.mypack.salt"
   └─ textures/
      ├─ entity/chicken/salt.png                       ← 64x32, entity texture
      └─ item/chicken/salt.png                         ← 16x16, item icon
```

`pack.mcmeta` for combined pack usage:

```json
{ "pack": { "description": "My Chickens", "pack_format": 48,
            "supported_formats": { "min_inclusive": 34, "max_inclusive": 48 } } }
```

**Installation:** Copy folder/zip to `saves/<world>/datapacks/` **and** `resourcepacks/`.
Registry loads on world load – after JSON edits, **rejoin the world** (`/reload` is insufficient).

---

## 2. chicken_type File – All Fields

| Field | Type | Default | Description |
|---|---|---|---|
| `lay_item` | item | **required** | Item laid by chicken → [ch. 3](#3-items) |
| `drop_item` | item | = `lay_item` | Item dropped on death |
| `background_color` | color | `#FFFFFF` | Background tint (icon/JEI) |
| `foreground_color` | color | `#FFFFFF` | Main color tint (eggs, fallback icon) |
| `texture` | path | `<ns>:textures/entity/chicken/<name>.png` | In-world texture (64×32) |
| `parents` | 2 IDs | none | `["a:x","b:y"]` – breeding pair definition |
| `breeding_chance` | 0–100 | by tier | % breeding chance → [ch. 4](#4-breeding-parents-tier-and-chance) |
| `lay_time_coefficient` | number | `1.0` | Lay time multiplier |
| `lay_time` | `{min,max}` | none | Explicit tick lay time |
| `properties` | list | `[]` | `fire_immune`, `explosion_immune`, `drowning_immune`, `freeze_immune`, `hover` |
| `spawn` | object | none | Spawning rules → [ch. 6](#6-natural-spawning) |
| `enabled` | bool | `true` | `false` disables chicken and descendants |

**Colors** accept `"#RRGGBB"`, `"RRGGBB"`, or decimal integer (`15132390`).

**Complete Example** (`copper.json`):

```json
{
  "lay_item": { "id": "minecraft:copper_ingot", "count": 2 },
  "drop_item": { "id": "minecraft:raw_copper" },
  "background_color": "#C15A36",
  "foreground_color": "#E77C56",
  "parents": ["chickens:iron", "chickens:orange"],
  "breeding_chance": 30,
  "lay_time_coefficient": 0.8,
  "properties": ["fire_immune"],
  "spawn": { "biomes": "#minecraft:is_badlands", "weight": 5 }
}
```

---

## 3. Items

Datapack 1.21 ItemStack format:

```json
{ "id": "minecraft:iron_ingot", "count": 3 }
```

`count` (1–99) = base yield per cycle (multiplied by chicken *Gain* stat).
`components` supported:

```json
{ "id": "minecraft:diamond_pickaxe", "count": 1,
  "components": { "minecraft:custom_name": "\"Chicken Pickaxe\"" } }
```

**Liquids** – liquid egg layout:

```json
{ "id": "chickens:liquid_egg", "count": 1, "components": { "chickens:liquid": "minecraft:lava" } }
```

Dye-laying chickens (`minecraft:*_dye`) automatically get a matching throwable colored egg.

---

## 4. Breeding: parents, tier and chance

### Parents
`"parents": ["chickens:iron", "chickens:orange"]`. Chickens **without** `parents` are Tier 1 base chickens.

### Tier
Tier = `1` for base, otherwise `max(Parent A, Parent B) + 1`. Controls default breeding weights and base lay time (`6000 × tier` ticks).

### Outcomes
Breeding A + B yields **A, B**, and any chicken listing `["A", "B"]` as parents. Default weight = `(max_tier + 1) − chicken_tier`.

### Manual Chances
Set via `"breeding_chance": 30` in JSON or `chickens-common.toml`:

```toml
breedingChances = ["mypack:copper = 30", "mypack:netherite_scrap = 10"]
```

Config overrides JSON. Non-explicit outcomes split remaining percentage based on default weights.

---

## 5. Laying Time

Default ticks formula:
`min = max(1, 6000 × tier × lay_time_coefficient)`, `max = 2 × min`.
Override via `"lay_time": { "min": 9000, "max": 15000 }`.

---

## 6. Natural Spawning

### In JSON
```json
"spawn": { "biomes": "#minecraft:is_badlands", "weight": 5 }
```
`biomes` accepts single biome, biome tag (`#`), or list.

### In Config – `config/chickens-spawns.toml`
Config takes priority over JSON:

```toml
[spawns]
    ender = ["end"]
    log   = ["#minecraft:is_forest"]
    white = ["nether"]
    smart = []
    flint = ["datapack"]

extraSpawns = ["mypack:copper = #minecraft:is_badlands, minecraft:desert"]
weights = ["mypack:copper = 5", "chickens:ender = 3"]
```

---

## 7. Properties

| Value | Effect |
|---|---|
| `fire_immune` | Fire/lava damage immunity |
| `explosion_immune` | Explosion immunity |
| `drowning_immune` | Drowning immunity |
| `freeze_immune` | Powder snow / freeze immunity |
| `hover` | Reduced fall speed |

---

## 8. Textures, Icons, and Names

- Entity texture: 64x32 PNG at `assets/<ns>/textures/entity/chicken/<name>.png`.
- Item icon: 16x16 PNG at `assets/<ns>/textures/item/chicken/<name>.png`.
- Name: lang key `chicken.<namespace>.<name>`.

---

## 9. Disabling and Overriding Chickens

- Set `"enabled": false` to disable a chicken and its recursive descendants.
- Datapacks loaded later override existing definitions.

---

## 10. Config Files

`config/chickens-common.toml` handles speed multipliers (`roostSpeed`, `breederSpeed`), henhouse radius, and breeding chances. `config/chickens-spawns.toml` handles spawn rules.

---

## 11. Built-in Chickens

Includes `vanilla` plus 47 types (`chickens` namespace). Spawns follow Nether, End, and Overworld biome tag assignments.

---

## 12. Troubleshooting

- **Chicken missing**: check syntax in log; rejoin world (no `/reload`).
- **Texture issues**: verify file path under `assets/<ns>/textures/entity/chicken/`.
- **Icon uses template**: missing 16x16 icon in resource pack.

---

## 13. Building and Running (1.21.1)

Project structure:
- `common`: loader-agnostic logic.
- `neoforge`: NeoForge entry point, registries, capabilities.
- `fabric`: Fabric entry point, registries, Transfer API.

Commands:
- JDK 21 required.
- `./gradlew buildAll` produces artifacts under `neoforge/build/libs/` and `fabric/build/libs/`.
- `./gradlew :neoforge:runClient` / `./gradlew :fabric:runClient`.

Data components: chicken data = `chickens:chicken_data`, colored/spawn egg = `chickens:chicken_type`, liquid egg = `chickens:liquid`.
