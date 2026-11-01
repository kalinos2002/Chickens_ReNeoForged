# Chickens & Roost – Documentation for Minecraft 1.19.2 (Forge)

Versions: [26.1.2](README.md) [1.21.1](documentation_1_21_1_md.md) | 1.19.2

Contents: [1. Quick Start](#1-quick-start) · [2. chicken_type File](#2-chicken_type-file--all-fields) ·
[3. Items](#3-items) · [4. Breeding](#4-breeding-parents-tier-and-chance) ·
[5. Laying Time](#5-laying-time) · [6. Spawning](#6-natural-spawning) · [7. Properties](#7-properties) ·
[8. Textures and Names](#8-textures-icons-and-names) · [9. Disabling and Overriding](#9-disabling-and-overriding-chickens) ·
[10. Configs](#10-config-files) · [11. Built-in List](#11-built-in-chickens) · [12. Troubleshooting](#12-troubleshooting) ·
[13. Version Details](#13-version-1192-details)

---

## About the Mod

**Chickens & Roost** combines the *Chickens Mod* (1.12.2) and *Roost* into a single mod. Chickens are defined via **datapacks**.

- **Chickens** (47 built-in + custom ones from datapacks): lay resources, breed, and have Growth / Gain / Strength stats.
- **Henhouse** (6 wood types), **Roost**, **Breeder**, **Collector** (collects items from Roost).
- **Egg Nest** (from the Hatchery mod): right-clicking with an egg (vanilla or colored) puts it into the nest; after a few minutes, a chick hatches (colored egg → chicken of that type, vanilla egg → vanilla chicken). Speeds up when a chicken sits in the nest and when a redstone lamp shines up to 3 blocks above it. Right-clicking a nest with an egg removes it.
- **Catcher**, **Analyzer**, colored eggs (throwable), liquid eggs, spawn eggs.
- **Chicken Item** = acts as both a spawn egg and a chicken to place inside a Roost / Breeder.
- **JEI** (optional): Breeding categories (parents → child + chance %), Laying, and Drops. Datapack chickens appear automatically.

---

## Differences Between Versions

| | 1.19.2 | 1.21.1 | 26.1.2 |
|---|---|---|---|
| Loaders | Forge (Fabric planned) | NeoForge + Fabric | NeoForge (Fabric planned) |
| Java | 17 | 21 | 25 |
| Item Data | NBT | Components | Components |
| `lay_item` / `drop_item` | `id`, `count`, `nbt` (legacy `components` syntax is converted) | `id`, `count`, `components` | `id`, `count`, `components` |
| Chicken Registry | Custom file loader (`/reload` works) | Datapack registry (rejoin world) | Datapack registry (rejoin world) |
| Name without lang entry | Formatted name from ID (Salt Chicken) | Formatted name from ID | Formatted name from ID |
| Baby Chicken Model | Vanilla (legacy appearance) | Legacy appearance | Legacy appearance (custom model) |
| Biomes in `spawn.biomes` | Only biomes available in 1.19.2 | 1.21.1 biomes | 26.1.2 biomes |

---

## 1. Quick Start

Minimal chicken (`data/mypack/chickens/chicken_type/salt.json`):

```json
{ "lay_item": { "id": "minecraft:sugar" } }
```

That's all: the chicken `mypack:salt` now exists, lays sugar, and can be obtained via command:
`/give @s chickens:chicken{ChickenData:{type:"mypack:salt"}}`.
All other fields are optional.

**Chicken ID** = `<namespace>:<file_name>`. Subfolders in the name work as well (`mypack:ores/tin` → file `ores/tin.json`); in translation keys, `/` is replaced by `.`.

**Package Structure** (datapack + resource pack combined):

```
mypack/
├─ pack.mcmeta
├─ data/mypack/chickens/chicken_type/salt.json         ← chicken definition
└─ assets/mypack/
   ├─ lang/en_us.json                                  ← name: "chicken.mypack.salt"
   └─ textures/
      ├─ entity/chicken/salt.png                       ← 64x32, in-world chicken texture
      └─ item/chicken/salt.png                         ← 16x16, item icon
```

`pack.mcmeta` for a package functioning as both types:

```json
{ "pack": { "description": "My Chickens", "pack_format": 10 } }
```

In 1.19.2, the datapack `pack_format` is **10**, and the resource pack format is **9**, with no `supported_formats` field. A package serving both roles should use **10** (datapack) – when enabled as a resource pack, the game will show a yellow version warning, but it remains fully functional. If you prefer to avoid warnings, create two separate zip files (`data/` with format 10 and `assets/` with format 9).

**Installation:** Copy the folder/zip to `saves/<world>/datapacks/` **and** to `resourcepacks/` (assets are not loaded directly from datapacks). In 1.19.2, chickens are loaded by a custom file loader, so after changing a JSON file, simply execute `/reload` (data is automatically synced to players).

---

## 2. chicken_type File – All Fields

| Field | Type | Default | Description |
|---|---|---|---|
| `lay_item` | item | **required** | Item laid by chicken → [ch. 3](#3-items) |
| `drop_item` | item | = `lay_item` | Item dropped when killed |
| `background_color` | color | `#FFFFFF` | Background color (e.g. icon/JEI) |
| `foreground_color` | color | `#FFFFFF` | Main color: tints colored eggs, backup icon color |
| `texture` | path | `<ns>:textures/entity/chicken/<name>.png` | In-world chicken texture (64×32) |
| `parents` | 2 IDs | none | `["a:x","b:y"]` – breeding these two yields this chicken |
| `breeding_chance` | 0–100 | by tier | % breeding chance → [ch. 4](#4-breeding-parents-tier-and-chance) |
| `lay_time_coefficient` | number | `1.0` | Laying time multiplier |
| `lay_time` | `{min,max}` | none | Explicit laying time in ticks (overrides formula) |
| `properties` | list | `[]` | `fire_immune`, `explosion_immune`, `drowning_immune`, `freeze_immune`, `hover` |
| `spawn` | object | none | Natural spawning configuration → [ch. 6](#6-natural-spawning) |
| `enabled` | bool | `true` | `false` disables the chicken (and its offspring) |

**Colors** can be specified as `"#RRGGBB"`, `"RRGGBB"`, or as a decimal number (`15132390`).

**Complete example** (same as `copper.json` in this package):

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

Format:

```json
{ "id": "minecraft:iron_ingot", "count": 3 }
```

`count` (1–64) = amount produced per laying cycle (for `lay_item`, this is later multiplied by the chicken's *Gain* stat).
Instead of `id`, `item` can be used. In 1.19.2, item data is stored as **NBT**, so `nbt` is used instead of `components`:

```json
{ "id": "minecraft:diamond_pickaxe", "count": 1,
  "nbt": { "display": { "Name": "{\"text\":\"Chicken Pickaxe\"}" } } }
```

**Datapacks from 1.21.1 work seamlessly:** the `components` field is accepted, and known components are converted (currently `chickens:liquid`). Unknown components are skipped with a log warning.

**Liquids** – produce a "liquid egg" containing the selected fluid (works like a single-use bucket):

```json
{ "id": "chickens:liquid_egg", "count": 1, "nbt": { "Liquid": "minecraft:lava" } }
```

Legacy syntax also works: `"components": { "chickens:liquid": "minecraft:lava" }`.

Chickens that lay **dye** (`minecraft:*_dye`) automatically receive a throwable colored egg (a single shared recipe: egg + dye for every chicken, including datapack ones).

**`/give` Commands:**

| Item | Command |
|---|---|
| Chicken | `/give @s chickens:chicken{ChickenData:{type:"mypack:salt"}}` |
| Colored Egg | `/give @s chickens:colored_egg{ChickenType:"mypack:salt"}` |
| Spawn Egg | `/give @s chickens:spawn_egg{ChickenType:"mypack:salt"}` |
| Liquid Egg | `/give @s chickens:liquid_egg{Liquid:"minecraft:water"}` |

Stats can also be provided in `ChickenData`: `{type:"mypack:salt",growth:5,gain:3,strength:2,analyzed:1b}`.

---

## 4. Breeding: parents, tier and chance

### Parents
`"parents": ["chickens:iron", "chickens:orange"]` – order does not matter. Parents can be chickens from other packs or the base mod. A chicken **without** `parents` is a base chicken (Tier 1) – obtained via spawn/commands, not breeding.

### Tier
Tier = `1` for base chickens, otherwise `max(Parent A tier, Parent B tier) + 1`. Affects:
- Default breeding chance (higher tier = rarer),
- Default laying time (`6000 × tier` ticks).

### Breeding Outcomes
Breeding Parent A and Parent B can yield: **A, B**, or **any chicken listing this exact pair in its `parents`** (there can be multiple!). Default weights: `weight = (highest tier in pool + 1) − chicken tier`, meaning lower tiers appear more frequently.

### Manual Chance Configuration
`"breeding_chance": 30` in JSON or in the `chickens-common.toml` config file:

```toml
breedingChances = ["mypack:copper = 30", "mypack:netherite_scrap = 10"]
```

Meaning: **this chicken drops with this exact percentage chance** when breeding its parents. Rules:
- Config overrides JSON settings.
- Remaining percentages are divided among other potential offspring from that pair (based on default weights).
- If multiple entries exceed 100% combined, they scale down proportionally to 100%.
- If *all* potential outcomes have defined chances, they normalize to 100%.
- Chance displays in JEI (Breeding category). Config is read independently on client and server – keep files synchronized, otherwise JEI may show inaccurate values.

Example: `iron + orange` yields A (iron), B (orange), and `mypack:copper` with `breeding_chance: 30` → copper 30%, while iron and orange evenly split the remaining 70% (same tier).

---

## 5. Laying Time

Default formula (in ticks, 20 ticks = 1s):

```
min = max(1, 6000 × tier × lay_time_coefficient)
max = 2 × min
```

Laying occurs at a random time between min and max. You can override this explicitly:
`"lay_time": { "min": 9000, "max": 15000 }` (min ≤ max; max will be raised to min if lower).
`lay_time_coefficient` < 1 = faster, > 1 = slower. In Roost/Breeder, time is additionally divided by `roostSpeed` / `breederSpeed` from `chickens-common.toml` and the chicken's *Growth* stat.

---

## 6. Natural Spawning

### In JSON
```json
"spawn": { "biomes": "#minecraft:is_badlands", "weight": 5 }
```
- `biomes`: tag (`"#minecraft:is_forest"`), single biome (`"minecraft:desert"`), or list:
  `["minecraft:plains", "#minecraft:is_taiga"]`.
- `weight` (1–1000, default 10): weighted selection within biomes that support multiple chicken spawns.

A chicken **without** a `spawn` block will not spawn naturally (it can still be obtained via breeding, commands, or catcher).

In 1.19.2, `biomes` is a string or list of strings: biome ID or tag with `#`. Biomes absent in 1.19.2 (e.g. `minecraft:cherry_grove`) are skipped.

Useful vanilla biome tags: `#minecraft:is_overworld`, `is_nether`, `is_end`, `is_forest`, `is_taiga`, `is_jungle`,
`is_savanna`, `is_badlands`, `is_mountain`, `is_beach`, `is_ocean`, `is_river`, `is_hill`. Mod tags:
`#chickens:spawn/normal` (forests, savannas, jungle, taiga, plains, meadows), `#chickens:spawn/snow`,
`#chickens:spawn/nether`, `#chickens:spawn/end`.

The mod adds chicken entities to Overworld, Nether, and End biomes (Forge biome modifier: `forge:add_spawns`) – *which* chicken appears is determined by JSON or config spawn definitions. Outside Overworld, chickens can spawn on solid blocks (netherrack, end stone…); in Overworld, like standard animals, on grass.

### In Config – `config/chickens-spawns.toml`
The config **overrides** JSON definitions and is the most convenient method for tuning without editing datapacks.

```toml
[spawns]
    # Each built-in chicken has an entry listing biome selectors
    ender = ["end"]                       # End only
    log   = ["#minecraft:is_forest"]      # Forests only
    white = ["nether"]
    smart = []                            # [] = never spawns naturally
    flint = ["datapack"]                  # "datapack" = use spawn block from JSON

# Datapack chickens (or overrides for any chicken): "id = selector, selector"
extraSpawns = ["mypack:copper = #minecraft:is_badlands, minecraft:desert"]

# Weights (default 10 or from JSON)
weights = ["mypack:copper = 5", "chickens:ender = 3"]
```

Selectors: `overworld`, `nether`, `end`, `any`, biome (`minecraft:plains`), biome tag (`#minecraft:is_forest`).
Invalid entries are skipped with a log warning. Changes apply after restarting the world/server.

---

## 7. Properties

| Value | Effect |
|---|---|
| `fire_immune` | Immune to fire and lava damage |
| `explosion_immune` | Immune to explosions |
| `drowning_immune` | Immune to drowning |
| `freeze_immune` | Immune to freezing (powder snow) – no freeze damage or slowdown |
| `hover` | Falls twice as slowly while airborne (glides) |

---

## 8. Textures, Icons, and Names

### In-World Chicken Texture
**64×32** PNG, matching vanilla chicken texture layout (`minecraft:textures/entity/chicken/chicken.png`).
Default path: `assets/<ns>/textures/entity/chicken/<name>.png`; custom path set via `texture` field
(e.g., `"texture": "chickens:textures/entity/iron_chicken.png"`).
Higher resolution textures (e.g., 128×64) are supported.

### Item Icon (16×16)
Custom item sprite: `assets/<ns>/textures/item/chicken/<name>.png` (vanilla chicken: `chickens:vanilla`).
Rendered as standard 3D item icon. **Without this file**, the item falls back to a template tinted with colors extracted from its entity texture. Built-in icons are located in `assets/chickens/textures/item/chicken/` for reference.

### Name
Key `chicken.<namespace>.<name>` in lang file:

```json
{ "chicken.mypack.salt": "Salt Chicken" }
```

For `mypack:ores/tin`, key is `chicken.mypack.ores.tin`. Without entry, name generates automatically from ID: `Salt Chicken`.

---

## 9. Disabling and Overriding Chickens

- **Disabling**: create a file with matching ID, e.g. `data/chickens/chickens/chicken_type/smart.json`:
  ```json
  { "lay_item": { "id": "minecraft:egg" }, "enabled": false }
  ```
  Disabled chickens are removed from breeding, spawning, and JEI; **their offspring are also disabled** recursively.
- **Overriding**: supply a full updated JSON definition; the datapack loaded later takes priority.
- `parents` can reference any chicken ID, including those from other packs.

---

## 10. Config Files

**`config/chickens-common.toml`**

| Key | Default | Description |
|---|---|---|
| `roostSpeed` | `1.0` | Roost processing speed multiplier |
| `breederSpeed` | `1.0` | Breeder processing speed multiplier |
| `vanillaChickensLayEggsInRoost` | `true` | `false` = vanilla chicken inside Roost produces only feathers |
| `alwaysShowStats` | `false` | Display Growth/Gain/Strength on unanalyzed chickens |
| `henhouseRadius` | `4` | Search radius (blocks) for Henhouse collection |
| `breedingChances` | `[]` | Breeding chance overrides, `"id = percentage"` |

**`config/chickens-spawns.toml`** – see [ch. 6](#in-config---configchickens-spawnstoml).

---

## 11. Built-in Chickens

`vanilla` (`minecraft:chicken`) plus 47 under `chickens` namespace:
black, blaze, blue, brown, clay, coal, cyan, diamond, emerald, ender, flint, ghast, glass, glowstone, gold, gray, green,
gunpowder, iron, lava, leather, light_blue, light_gray, lime, log, magenta, magma, netherwart, obsidian, orange, pink,
prismarine_crystal, prismarine_shard, purple, quartz, red, redstone, sand, slime, smart, snowball, soul_sand, string,
water, white, yellow.

Default spawns: `white`, `flint`, `quartz`, `soul_sand` – Nether; `ender` – End; `log`, `sand` – biomes in `#chickens:spawn/normal`; others via breeding only.

---

## 12. Troubleshooting

| Issue | Cause / Solution |
|---|---|
| Chicken does not appear | Check JSON for syntax errors (`Failed to load chicken type`). Run `/reload` after changes. |
| Chicken won't spawn naturally | Missing `spawn` block or non-matching biomes; check `chickens-spawns.toml` (`[]` prevents spawns). Spawning requires valid light and surface conditions (grass in Overworld). |
| Name shows as "Salt Chicken" despite lang entry | Key must match `chicken.<ns>.<name>` exactly, and package must be enabled as a **resource pack**. |
| White/magenta missing texture | Incorrect path/dimensions; verify `texture` path under `assets/<ns>/textures/entity/chicken/`. |
| Item icon uses generic template | Missing `assets/<ns>/textures/item/chicken/<name>.png` in resource pack. |
| Breeding doesn't yield target chicken | Verify `parents` configuration. Ensure neither chicken nor ancestors have `enabled: false`. Chance < 100% means outcomes are probabilistic. |
| JEI shows different chance than game | Mismatched `chickens-common.toml` between client and server. |

---

## 13. Version 1.19.2 Details

**Status:** Forge implementation active. **Fabric** version planned post-Forge release.

- Requirements: **Java 17**, Minecraft **1.19.2**, **Forge 43.x**, optional **JEI 11.x**.
- Chickens are loaded via custom file reader `ChickenTypeReloadListener` from `data/<ns>/chickens/chicken_type/*.json`; data syncs to clients upon connection and `/reload`.
- NBT Item Data: chicken `ChickenData{type,growth,gain,strength,analyzed}`, colored eggs `ChickenType`, liquid egg `Liquid` (optional `LiquidColor`).
- Creative Tabs: Chickens, Colored Eggs, Spawn Eggs (`itemGroup.chickens.main`, `.colored_eggs`, `.spawn_eggs`).
- Seed Tag: `chickens:seeds` includes `forge:seeds`, optional `c:seeds`, and vanilla seeds.
- Recipes: single dynamic recipe (egg + any dye); separate recipes per color are omitted.
- Automation: Forge item handler (`ForgeCapabilities.ITEM_HANDLER`) attached to Henhouse, Roost, Breeder, and Collector.
- Baby chicken utilizes vanilla 1.19.2 model.
- JEI 11.x integration via optional API plugin.

### Building (1.19.2)
Multi-loader layout: `Common` (VanillaGradle), `Forge` (ForgeGradle), `Fabric` (Loom).

- `./gradlew :Common:compileJava`
- `./gradlew :Forge:compileJava`, `./gradlew :Forge:runClient`, `./gradlew :Forge:build`
- JEI compile dependency: `compileOnly 'mezz.jei:jei-1.19.2-common-api:<version>'` (`https://maven.blamejared.com/`).
- Log output: `Forge/run/logs/latest.log`.
