# Chickens – Natural Spawning & Datapack Chickens

*(Polish version: [SPAWNING.pl.md](README.pl.md))*

This document explains how chickens spawn in the world, how to define a new chicken in a datapack, and how the `chickens-spawns.toml` config interacts with datapack definitions.

---

## 1. Quick summary

| Question | Answer |
|---|---|
| Where do I define spawning for a **new datapack chicken**? | In the chicken's JSON, in the `spawn` block. |
| Where do I **override** spawning (any chicken)? | In `config/chickens-spawns.toml`. |
| Which one wins? | **The toml.** If a chicken has an entry in the toml, its JSON `spawn` block is ignored. |
| I changed `spawn` in the JSON of a **bundled** chicken (e.g. `chickens:iron`) and nothing happens. | Bundled chickens already have an entry in the `[spawns]` section of the toml. Set that entry to `["datapack"]` to make the JSON count. |
| A chicken has no toml entry and no `spawn` block. | It never spawns naturally (breeding, catcher and spawn items only). |

---

## 2. How spawning works

1. **Biome modifier** (`data/chickens/neoforge/biome_modifier/add_chicken_spawns.json`) adds the `chickens:chicken` entity to every biome in the tag `#chickens:spawns_chickens` (by default all Overworld, Nether and End biomes). Spawn weight `10`, group size `2–4`. These two numbers are **not** in the toml.
2. **Spawn check** – before a natural spawn happens, the mod checks that at least one enabled chicken type can spawn in that biome. If none can, no `chickens:chicken` spawns there (vanilla chickens are not affected).
3. **Type selection** – when the entity spawns, one type is picked by weighted random among all enabled chickens allowed in that biome. The whole group gets the **same** type.
4. **Fallbacks** – a chicken that has no type (spawner, `/summon`, old saves) gets a type matching its biome; if none matches, a random enabled root chicken (no parents).

A chicken type is **enabled** only if `enabled` is not `false` **and** all of its ancestors (parents, recursively) are enabled.

### Decision order for one chicken type in one biome

```
1. Entry in chickens-spawns.toml  (section [spawns] or "extraSpawns")?
      yes → use ONLY that entry (unless it is ["datapack"])
      no  ↓
2. "spawn" block in the chicken_type JSON?
      yes → use it
      no  → this chicken does not spawn naturally
```

Weight order: `weights` in the toml → `spawn.weight` in the JSON → `10`.

---

## 3. Defining a chicken in a datapack

File location:

```
data/<namespace>/chickens/chicken_type/<name>.json
```

The chicken id is `<namespace>:<name>`. The registry is synced to clients.

### Example (with natural spawning)

```json
{
  "lay_item": { "id": "minecraft:copper_ingot", "count": 1 },
  "drop_item": { "id": "minecraft:raw_copper", "count": 1 },
  "background_color": "#C77D4F",
  "foreground_color": "#E0A070",
  "texture": "mypack:textures/entity/copper_chicken.png",
  "lay_time_coefficient": 1.0,
  "properties": ["fire_immune"],
  "spawn": {
    "biomes": "#minecraft:is_badlands",
    "weight": 10
  }
}
```

### Example (bred only, no natural spawning)

```json
{
  "lay_item": { "id": "minecraft:amethyst_shard", "count": 1 },
  "texture": "mypack:textures/entity/amethyst_chicken.png",
  "parents": [ "chickens:purple", "chickens:diamond" ]
}
```

### Fields

| Field | Required | Description |
|---|---|---|
| `lay_item` | **yes** | Item stack the chicken lays. |
| `drop_item` | no | Item dropped on death (defaults to `lay_item`). |
| `background_color` / `foreground_color` | no | `"#RRGGBB"` or integer. Default `#FFFFFF`. Used for tinting and UI. |
| `texture` | no | Entity texture. If omitted: `<namespace>:textures/entity/chicken/<name>.png`. |
| `parents` | no | Exactly 2 chicken ids. Makes the chicken breedable. |
| `lay_time_coefficient` | no | Multiplier for laying time (default `1.0`). Base is `6000 * tier * coefficient` ticks (min), max is twice that. |
| `lay_time` | no | `{ "min": ticks, "max": ticks }` – overrides the formula above. |
| `properties` | no | List: `fire_immune`, `explosion_immune`, `drowning_immune`, `freeze_immune`, `hover`. |
| `spawn` | no | Natural spawning – see below. |
| `enabled` | no | `false` disables the chicken (and all its descendants). Default `true`. |
| `breeding_chance` | no | 0–100 (percent). Can be overridden by `breedingChances` in the main config. |

### The `spawn` block

```json
"spawn": {
  "biomes": "#chickens:spawn/normal",
  "weight": 10
}
```

* `biomes` – a single biome tag (`"#namespace:tag"`), a single biome (`"minecraft:plains"`) or a list of biomes (`["minecraft:plains", "minecraft:desert"]`).
* `weight` – 1–1000, default `10`. Higher = more common in biomes shared with other chickens.

Tags bundled with the mod: `#chickens:spawn/normal` (forest, savanna, jungle, taiga, plains, sunflower plains, meadow, cherry grove), `#chickens:spawn/snow`, `#chickens:spawn/nether`, `#chickens:spawn/end`.

> The `spawn` block is read **on the server only**. It is not synced to clients (this does not affect gameplay, because the server decides spawns).

---

## 4. `config/chickens-spawns.toml`

The file is generated on the first start. **Existing files are never overwritten by newer defaults** – if you update the mod and want new defaults, delete the file or edit it by hand.

### 4.1 `[spawns]` – bundled chickens

Every bundled chicken has one list of biome selectors:

```toml
[spawns]
    log = ["#chickens:spawn/normal"]
    ender = ["#minecraft:is_end"]
    iron = []
    gold = ["datapack"]
    diamond = ["overworld", "minecraft:desert"]
```

| Value | Meaning |
|---|---|
| `[]` | Never spawns naturally. |
| `["datapack"]` | Use the `spawn` block from the chicken's JSON (**required to make a JSON `spawn` block count for bundled chickens**). |
| `[ "selector", ... ]` | Spawns in all listed biomes. |

Most bundled chickens default to `[]`, so in a fresh config only a few of them spawn naturally.

### 4.2 Selectors

| Selector | Matches |
|---|---|
| `overworld` | Biomes in `#minecraft:is_overworld` |
| `nether` | Biomes in `#minecraft:is_nether` |
| `end` | Biomes in `#minecraft:is_end` |
| `any` (also `all`, `*`) | Every biome |
| `minecraft:plains` | One specific biome |
| `#minecraft:is_forest` | A biome tag (prefix `#`) |

### 4.3 `extraSpawns` – datapack chickens and overrides

For chickens that are **not** listed in `[spawns]` (datapack chickens), or to override any chicken:

```toml
extraSpawns = [
    "mypack:copper = #minecraft:is_badlands",
    "mypack:amethyst = overworld, minecraft:dripstone_caves",
    "chickens:iron = #minecraft:is_mountain",
    "mypack:tin = datapack"
]
```

* Format: `"namespace:chicken = selector, selector, ..."`.
* These entries **win over** `[spawns]`.
* `= datapack` removes the override and hands control back to the JSON.
* An empty selector list (`"mypack:tin = "`) means "never spawns naturally".

### 4.4 `weights`

```toml
weights = [
    "chickens:ender = 5",
    "mypack:copper = 20"
]
```

Default weight is the JSON `spawn.weight`, or `10` if there is none. Weights only matter when several chickens are allowed in the same biome.

---

## 5. Common scenarios

**I made a new datapack chicken and want it to spawn in badlands.**
Add the `spawn` block to its JSON. Do not touch the toml.

**I want to change where a bundled chicken (e.g. `chickens:iron`) spawns, by datapack.**
Override its JSON, add a `spawn` block **and** set `iron = ["datapack"]` in the `[spawns]` section of the toml.

**I'm a server admin and want to change spawns without touching datapacks.**
Use `extraSpawns` and `weights` in the toml.

**I want a chicken to stop spawning naturally.**
Toml: `iron = []` (bundled) or `"mypack:copper = "` in `extraSpawns`. Datapack: remove its `spawn` block.

**I want to disable a chicken completely.**
Set `"enabled": false` in its JSON (descendants are disabled as well).

---

## 6. Troubleshooting

| Symptom | Likely cause |
|---|---|
| Datapack chicken never spawns. | Missing `spawn` block, or an entry in the toml overrides it, or the chicken/an ancestor is disabled. Check the log for `chickens-spawns.toml: ...` warnings. |
| JSON `spawn` of a bundled chicken is ignored. | Its toml entry is not `["datapack"]`. |
| No `chickens:chicken` in a biome at all. | No enabled chicken is allowed in that biome (see section 2, step 2). |
| New defaults after a mod update are missing. | The old `chickens-spawns.toml` is still used – delete it to regenerate. |
| Warning `bad selector` / `ignoring ...` in the log. | Typo in an id or selector in the toml; the entry is skipped. |
| Chicken spawned with the vanilla texture / gives an item with a missing texture. | Fixed: an untyped chicken now receives a type on spawn or on its first tick. Make sure the type's `texture` path exists in a resource pack. |

---

## 7. Related behaviour

* Chickens of this mod **never take fall damage**, like vanilla chickens (`causeFallDamage` returns `false`, and the entity is in the `minecraft:fall_damage_immune` entity type tag).
* The Catcher refuses to catch a chicken whose type is unknown or disabled.
