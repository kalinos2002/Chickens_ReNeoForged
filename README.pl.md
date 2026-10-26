# Chickens – naturalny spawn i kurczaki z datapacków

*(English version: [SPAWNING.en.md](README.md))*

Ten dokument opisuje, jak kurczaki pojawiają się w świecie, jak zdefiniować nowego kurczaka w datapacku oraz jak plik `chickens-spawns.toml` współpracuje z definicjami z datapacków.

---

## 1. Krótkie podsumowanie

| Pytanie | Odpowiedź |
|---|---|
| Gdzie ustawić spawn **nowego kurczaka z datapacka**? | W JSON-ie kurczaka, w bloku `spawn`. |
| Gdzie **nadpisać** spawn (dowolnego kurczaka)? | W `config/chickens-spawns.toml`. |
| Co ma pierwszeństwo? | **Toml.** Jeśli kurczak ma wpis w tomlu, blok `spawn` z jego JSON-a jest ignorowany. |
| Zmieniłem `spawn` w JSON-ie **wbudowanego** kurczaka (np. `chickens:iron`) i nic się nie dzieje. | Wbudowani kurczaki mają już wpis w sekcji `[spawns]` w tomlu. Ustaw ten wpis na `["datapack"]`, żeby JSON zaczął działać. |
| Kurczak nie ma wpisu w tomlu ani bloku `spawn`. | Nigdy nie spawnuje się naturalnie (tylko hodowla, łapacz i przedmioty). |

---

## 2. Jak działa spawn

1. **Biome modifier** (`data/chickens/neoforge/biome_modifier/add_chicken_spawns.json`) dodaje encję `chickens:chicken` do każdego biomu z tagu `#chickens:spawns_chickens` (domyślnie wszystkie biomy Overworldu, Netheru i Endu). Waga spawnu to `10`, a wielkość grupy `2–4`. Tych dwóch liczb **nie ma** w tomlu.
2. **Sprawdzenie spawnu** – przed naturalnym spawnem mod sprawdza, czy w danym biomie może się pojawić co najmniej jeden włączony typ kurczaka. Jeśli nie, `chickens:chicken` nie spawnuje się tam wcale (zwykłe kurczaki z vanilli nie są ruszane).
3. **Losowanie typu** – po pojawieniu się encji typ jest losowany z wagami spośród włączonych kurczaków dozwolonych w tym biomie. Cała grupa dostaje **ten sam** typ.
4. **Zabezpieczenia** – kurczak bez typu (spawner, `/summon`, stare zapisy) dostaje typ pasujący do biomu, a jeśli żaden nie pasuje, losowego włączonego kurczaka bazowego (bez rodziców).

Typ kurczaka jest **włączony**, tylko jeśli `enabled` nie jest `false` **i** wszyscy jego przodkowie (rodzice, rekurencyjnie) są włączeni.

### Kolejność decyzji dla jednego kurczaka w jednym biomie

```
1. Wpis w chickens-spawns.toml  (sekcja [spawns] lub "extraSpawns")?
      tak → używany jest TYLKO ten wpis (chyba że to ["datapack"])
      nie ↓
2. Blok "spawn" w JSON-ie chicken_type?
      tak → używany jest on
      nie → ten kurczak nie spawnuje się naturalnie
```

Kolejność dla wagi: `weights` w tomlu → `spawn.weight` w JSON-ie → `10`.

---

## 3. Definiowanie kurczaka w datapacku

Lokalizacja pliku:

```
data/<namespace>/chickens/chicken_type/<nazwa>.json
```

Id kurczaka to `<namespace>:<nazwa>`. Rejestr jest synchronizowany do klientów.

### Przykład (z naturalnym spawnem)

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

### Przykład (tylko z hodowli, bez naturalnego spawnu)

```json
{
  "lay_item": { "id": "minecraft:amethyst_shard", "count": 1 },
  "texture": "mypack:textures/entity/amethyst_chicken.png",
  "parents": [ "chickens:purple", "chickens:diamond" ]
}
```

### Pola

| Pole | Wymagane | Opis |
|---|---|---|
| `lay_item` | **tak** | Przedmiot, który kurczak znosi. |
| `drop_item` | nie | Przedmiot wypadający po śmierci (domyślnie `lay_item`). |
| `background_color` / `foreground_color` | nie | `"#RRGGBB"` lub liczba. Domyślnie `#FFFFFF`. Używane do barwienia i w interfejsie. |
| `texture` | nie | Tekstura encji. Bez niej: `<namespace>:textures/entity/chicken/<nazwa>.png`. |
| `parents` | nie | Dokładnie 2 id kurczaków. Pozwala hodować tego kurczaka. |
| `lay_time_coefficient` | nie | Mnożnik czasu znoszenia (domyślnie `1.0`). Minimum to `6000 * tier * współczynnik` ticków, maksimum to dwukrotność. |
| `lay_time` | nie | `{ "min": ticki, "max": ticki }` – zastępuje powyższy wzór. |
| `properties` | nie | Lista: `fire_immune`, `explosion_immune`, `drowning_immune`, `freeze_immune`, `hover`. |
| `spawn` | nie | Naturalny spawn – patrz niżej. |
| `enabled` | nie | `false` wyłącza kurczaka (i wszystkich jego potomków). Domyślnie `true`. |
| `breeding_chance` | nie | 0–100 (procent). Można nadpisać przez `breedingChances` w głównym configu. |

### Blok `spawn`

```json
"spawn": {
  "biomes": "#chickens:spawn/normal",
  "weight": 10
}
```

* `biomes` – pojedynczy tag biomów (`"#namespace:tag"`), pojedynczy biom (`"minecraft:plains"`) albo lista biomów (`["minecraft:plains", "minecraft:desert"]`).
* `weight` – 1–1000, domyślnie `10`. Im więcej, tym częściej w biomach współdzielonych z innymi kurczakami.

Tagi dołączone do moda: `#chickens:spawn/normal` (las, sawanna, dżungla, tajga, równiny, słonecznikowe równiny, łąka, wiśniowy gaj), `#chickens:spawn/snow`, `#chickens:spawn/nether`, `#chickens:spawn/end`.

> Blok `spawn` jest wczytywany **tylko po stronie serwera**. Nie jest synchronizowany do klientów (nie wpływa to na rozgrywkę, bo o spawnie decyduje serwer).

---

## 4. `config/chickens-spawns.toml`

Plik generuje się przy pierwszym uruchomieniu. **Istniejący plik nie dostaje nowych wartości domyślnych** – po aktualizacji moda usuń go albo edytuj ręcznie, jeśli chcesz nowych ustawień.

### 4.1 `[spawns]` – wbudowani kurczaki

Każdy wbudowany kurczak ma jedną listę selektorów biomów:

```toml
[spawns]
    log = ["#chickens:spawn/normal"]
    ender = ["#minecraft:is_end"]
    iron = []
    gold = ["datapack"]
    diamond = ["overworld", "minecraft:desert"]
```

| Wartość | Znaczenie |
|---|---|
| `[]` | Nigdy nie spawnuje się naturalnie. |
| `["datapack"]` | Użyj bloku `spawn` z JSON-a kurczaka (**wymagane, żeby blok `spawn` w JSON-ie wbudowanego kurczaka zadziałał**). |
| `[ "selektor", ... ]` | Spawnuje się we wszystkich wymienionych biomach. |

Większość wbudowanych kurczaków ma domyślnie `[]`, więc w świeżym configu tylko kilku z nich spawnuje się naturalnie.

### 4.2 Selektory

| Selektor | Pasuje do |
|---|---|
| `overworld` | Biomów z `#minecraft:is_overworld` |
| `nether` | Biomów z `#minecraft:is_nether` |
| `end` | Biomów z `#minecraft:is_end` |
| `any` (także `all`, `*`) | Każdego biomu |
| `minecraft:plains` | Jednego konkretnego biomu |
| `#minecraft:is_forest` | Tagu biomów (prefiks `#`) |

### 4.3 `extraSpawns` – kurczaki z datapacków i nadpisania

Dla kurczaków, których **nie ma** w `[spawns]` (kurczaki z datapacków), albo do nadpisania dowolnego kurczaka:

```toml
extraSpawns = [
    "mypack:copper = #minecraft:is_badlands",
    "mypack:amethyst = overworld, minecraft:dripstone_caves",
    "chickens:iron = #minecraft:is_mountain",
    "mypack:tin = datapack"
]
```

* Format: `"namespace:kurczak = selektor, selektor, ..."`.
* Te wpisy **mają pierwszeństwo** przed `[spawns]`.
* `= datapack` usuwa nadpisanie i oddaje kontrolę JSON-owi.
* Pusta lista selektorów (`"mypack:tin = "`) oznacza „nigdy nie spawnuje się naturalnie”.

### 4.4 `weights`

```toml
weights = [
    "chickens:ender = 5",
    "mypack:copper = 20"
]
```

Domyślna waga to `spawn.weight` z JSON-a, a bez niego `10`. Wagi mają znaczenie tylko wtedy, gdy w jednym biomie może się pojawić kilku kurczaków.

---

## 5. Typowe scenariusze

**Zrobiłem nowego kurczaka w datapacku i chcę, żeby spawnował się w badlandsach.**
Dodaj blok `spawn` do jego JSON-a. Nie ruszaj tomla.

**Chcę zmienić, gdzie spawnuje się wbudowany kurczak (np. `chickens:iron`) przez datapack.**
Nadpisz jego JSON, dodaj blok `spawn` **oraz** ustaw `iron = ["datapack"]` w sekcji `[spawns]` w tomlu.

**Jestem adminem serwera i chcę zmienić spawn bez ruszania datapacków.**
Użyj `extraSpawns` i `weights` w tomlu.

**Chcę, żeby kurczak przestał się spawnować naturalnie.**
Toml: `iron = []` (wbudowany) albo `"mypack:copper = "` w `extraSpawns`. Datapack: usuń blok `spawn`.

**Chcę całkowicie wyłączyć kurczaka.**
Ustaw `"enabled": false` w jego JSON-ie (potomkowie też zostaną wyłączeni).

---

## 6. Rozwiązywanie problemów

| Objaw | Prawdopodobna przyczyna |
|---|---|
| Kurczak z datapacka nigdy się nie spawnuje. | Brak bloku `spawn`, nadpisuje go wpis w tomlu albo kurczak/jego przodek jest wyłączony. Sprawdź w logu ostrzeżenia `chickens-spawns.toml: ...`. |
| `spawn` w JSON-ie wbudowanego kurczaka jest ignorowany. | Jego wpis w tomlu nie jest `["datapack"]`. |
| W ogóle nie ma `chickens:chicken` w danym biomie. | Żaden włączony kurczak nie jest tam dozwolony (sekcja 2, punkt 2). |
| Po aktualizacji moda brakuje nowych ustawień domyślnych. | Nadal używany jest stary `chickens-spawns.toml` – usuń go, żeby się wygenerował na nowo. |
| W logu ostrzeżenie `bad selector` / `ignoring ...`. | Literówka w id lub selektorze w tomlu; wpis jest pomijany. |
| Kurczak pojawił się z teksturą vanilli / daje przedmiot z brakującą teksturą. | Naprawione: kurczak bez typu dostaje teraz typ przy spawnie lub w pierwszym ticku. Upewnij się, że ścieżka `texture` danego typu istnieje w resource packu. |

---

## 7. Powiązane zachowanie

* Kurczaki z tego moda **nigdy nie dostają obrażeń od upadku**, tak jak vanillowe (`causeFallDamage` zwraca `false`, a encja jest w tagu typów encji `minecraft:fall_damage_immune`).
* Łapacz odmawia złapania kurczaka o nieznanym lub wyłączonym typie.
