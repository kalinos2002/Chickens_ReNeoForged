# Chickens & Roost – dokumentacja dla Minecrafta 26.1.2 (NeoForge)

Spis: [1. Szybki start](#1-szybki-start) · [2. Plik chicken_type](#2-plik-chicken_type--wszystkie-pola) ·
[3. Przedmioty](#3-przedmioty) · [4. Krzyżowanie](#4-krzyżowanie-parents-tier-i-szansa) ·
[5. Czas znoszenia](#5-czas-znoszenia) · [6. Spawn](#6-naturalny-spawn) · [7. Własności](#7-własności-properties) ·
[8. Tekstury i nazwy](#8-tekstury-ikony-i-nazwy) · [9. Wyłączanie i nadpisywanie](#9-wyłączanie-i-nadpisywanie-kurczaków) ·
[10. Configi](#10-pliki-config) · [11. Lista wbudowanych](#11-wbudowane-kurczaki) · [12. Problemy](#12-rozwiązywanie-problemów) ·
[13. Specyfika 26.1.2](#13-specyfika-wersji-2612)

---

## O modzie

**Chickens & Roost** łączy *Chickens Mod* (1.12.2) i *Roost* w jeden mod. Kurczaki są definiowane w **datapackach**.

- **Kurczaki** (47 wbudowanych + dowolne z datapacków): znoszą surowce, krzyżują się, mają statystyki Growth / Gain / Strength.
- **Henhouse** (kurnik, 6 rodzajów drewna), **Roost**, **Breeder**, **Collector** (zbiera z Roost).
- **Gniazdo** (Egg Nest, z moda Hatchery): prawy klik jajkiem (zwykłym lub kolorowym) wkłada je do gniazda, po kilkunastu minutach wykluwa się pisklę (kolorowe jajko → kurczak danego typu, zwykłe → vanilla). Szybciej, gdy w gnieździe siedzi kurczak i gdy do 3 bloków nad nim świeci lampa redstone. Prawy klik na gnieździe z jajkiem wyjmuje je.
- **Łapacz** (Catcher), **Analizator**, kolorowe jajka (rzucane), płynne jajka, jajka spawnujące.
- Przedmiot **Kurczak** = jajko spawnujące i kurczak do wkładania w Roost / Breeder.
- **JEI** (opcjonalnie): kategorie Krzyżowanie (rodzice → dziecko + szansa %), Znoszenie i Dropy. Kurczaki z datapacków pojawiają się automatycznie.

---

## Różnice między wersjami

| | 1.19.2 | 1.21.1 | 26.1.2 |
|---|---|---|---|
| Loaders | Forge (Fabric w planie) | NeoForge + Fabric | NeoForge (Fabric w planie) |
| Java | 17 | 21 | 25 |
| Dane na przedmiocie | NBT | komponenty | komponenty |
| `lay_item` / `drop_item` | `id`, `count`, `nbt` (stary zapis `components` jest tłumaczony) | `id`, `count`, `components` | `id`, `count`, `components` |
| Rejestr kurczaków | własny wczytywacz plików (działa `/reload`) | rejestr datapackowy (wejdź do świata ponownie) | rejestr datapackowy (wejdź do świata ponownie) |
| Nazwa bez wpisu w lang | ładna nazwa z id (Salt Chicken) | ładna nazwa z id | ładna nazwa z id |
| Model małego kurczaka | vanilla (stary wygląd) | stary wygląd | stary wygląd (własny model) |
| Biomy w `spawn.biomes` | tylko te, które istnieją w 1.19.2 | biomy 1.21.1 | biomy 26.1.2 |

---

## 1. Szybki start

Minimalny kurczak (`data/mypack/chickens/chicken_type/salt.json`):

```json
{ "lay_item": { "id": "minecraft:sugar" } }
```

To wystarczy: kurczak `mypack:salt` istnieje, znosi cukier i da się go zdobyć komendą
`/give @s chickens:chicken[chickens:chicken_data={type:"mypack:salt"}]`.
Reszta pól jest opcjonalna.

**Id kurczaka** = `<namespace>:<nazwa pliku>`. Podfolder w nazwie też działa (`mypack:ores/tin` → plik `ores/tin.json`);
w kluczach tłumaczeń `/` zamienia się na `.`.

**Struktura pakietu** (datapack + resource pack w jednym):

```
mypack/
├─ pack.mcmeta
├─ data/mypack/chickens/chicken_type/salt.json         ← definicja kurczaka
└─ assets/mypack/
   ├─ lang/en_us.json                                  ← nazwa: "chicken.mypack.salt"
   └─ textures/
      ├─ entity/chicken/salt.png                       ← 64x32, kurczak w świecie
      └─ item/chicken/salt.png                         ← 16x16, ikona przedmiotu
```

`pack.mcmeta` dla pakietu działającego jako oba typy:

```json
{ "pack": { "description": "Moje kurczaki", "pack_format": 61 } }
```

Numer `pack_format` musi odpowiadać wersji 26.1.2 – sprawdź go w dokumentacji Minecrafta (Pack format) lub w pliku `pack.mcmeta`
wbudowanego pakietu gry. Nie podaję tu liczby, bo nie została sprawdzona dla tej wersji. Starą wartość z 1.21.1 (48) trzeba zmienić.

**Instalacja:** folder/zip do `saves/<świat>/datapacks/` **oraz** do `resourcepacks/` (assety nie są ładowane z datapacków).
Rejestr kurczaków wczytuje się przy ładowaniu świata – po zmianie JSON-a **wejdź do świata ponownie** (`/reload` nie wystarczy).

---

## 2. Plik chicken_type – wszystkie pola

| pole | typ | domyślnie | opis |
|---|---|---|---|
| `lay_item` | przedmiot | **wymagane** | co kurczak znosi → [rozdz. 3](#3-przedmioty) |
| `drop_item` | przedmiot | = `lay_item` | co wypada po zabiciu kurczaka |
| `background_color` | kolor | `#FFFFFF` | kolor tła (np. w ikonie/JEI) |
| `foreground_color` | kolor | `#FFFFFF` | kolor główny: zabarwienie kolorowego jajka, zapasowy kolor ikony |
| `texture` | ścieżka | `<ns>:textures/entity/chicken/<nazwa>.png` | tekstura kurczaka w świecie (64×32) |
| `parents` | 2 id | brak | `["a:x","b:y"]` – krzyżowanie tych dwóch daje tego kurczaka |
| `breeding_chance` | 0–100 | wg tieru | szansa % wykrzyżowania → [rozdz. 4](#4-krzyżowanie-parents-tier-i-szansa) |
| `lay_time_coefficient` | liczba | `1.0` | mnożnik czasu znoszenia |
| `lay_time` | `{min,max}` | brak | jawny czas znoszenia w tickach (zastępuje wzór) |
| `properties` | lista | `[]` | `fire_immune`, `explosion_immune`, `drowning_immune`, `freeze_immune`, `hover` |
| `spawn` | obiekt | brak | naturalny spawn → [rozdz. 6](#6-naturalny-spawn) |
| `enabled` | bool | `true` | `false` wyłącza kurczaka (i jego potomków) |

**Kolory** można podać jako `"#RRGGBB"`, `"RRGGBB"` lub liczbę dziesiętną (`15132390`).

**Kompletny przykład** (to samo, co `copper.json` w tym pakiecie):

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

## 3. Przedmioty

Format jak ItemStack w datapackach 1.21:

```json
{ "id": "minecraft:iron_ingot", "count": 3 }
```

`count` (1–99) = ile sztuk za jedno zniesienie (dla `lay_item` mnożone później przez statystykę *Gain* kurczaka).
Można dodać `components`:

```json
{ "id": "minecraft:diamond_pickaxe", "count": 1,
  "components": { "minecraft:custom_name": "\"Kilof z kurczaka\"" } }
```

**Płyny** – znoś „płynne jajko” z wybranym płynem (działa jak jednorazowe wiadro):

```json
{ "id": "chickens:liquid_egg", "count": 1, "components": { "chickens:liquid": "minecraft:lava" } }
```

Kurczaki, które znoszą **barwnik** (`minecraft:*_dye`), dostają automatycznie kolorowe jajko do rzucania.

---

## 4. Krzyżowanie: parents, tier i szansa

### Parents
`"parents": ["chickens:iron", "chickens:orange"]` – kolejność nie ma znaczenia. Rodzicami mogą być kurczaki z innych
pakietów i z moda. Kurczak **bez** `parents` jest bazowy (tier 1) – zdobywa się go ze spawnu/komendy, nie z krzyżowania.

### Tier
Tier = `1` dla bazowych, w pozostałych przypadkach `max(tier rodzica A, tier rodzica B) + 1`. Wpływa na:
- domyślną szansę krzyżowania (wyższy tier = rzadszy),
- domyślny czas znoszenia (`6000 × tier` ticków).

### Kto może powstać z pary
Z krzyżowania rodziców A i B mogą wypaść: **A, B** oraz **każdy kurczak, który wymienia dokładnie tę parę w `parents`**
(może ich być kilku!). Domyślne wagi: `waga = (najwyższy tier w puli + 1) − tier kurczaka`, więc niższy tier = częściej.

### Ręczne ustawienie szansy
`"breeding_chance": 30` w JSON-ie albo w configu `chickens-common.toml`:

```toml
breedingChances = ["mypack:copper = 30", "mypack:netherite_scrap = 10"]
```

Znaczenie: **ten kurczak wypada z dokładnie taką szansą (w %)**, gdy krzyżujesz jego rodziców. Zasady:
- Config ma pierwszeństwo przed JSON-em.
- Pozostałe procenty dzielą się między resztę możliwego potomstwa tej pary (wg domyślnych wag).
- Jeśli kilka wpisów razem przekracza 100 %, są skalowane w dół do 100 %.
- Jeśli *wszystkie* możliwe wyniki mają wpis, są normalizowane do 100 %.
- Szansa wyświetla się w JEI (kategoria Krzyżowanie). Config jest czytany osobno po stronie klienta i serwera – trzymaj
  ten sam plik w obu miejscach, inaczej JEI może pokazywać inną wartość niż ta, która działa.

Przykład: para `iron + orange` daje A (iron), B (orange) i `mypack:copper` z `breeding_chance: 30` → copper 30 %,
a iron i orange dzielą 70 % po równo (mają ten sam tier).

---

## 5. Czas znoszenia

Domyślnie (ticki, 20 ticków = 1 s):

```
min = max(1, 6000 × tier × lay_time_coefficient)
max = 2 × min
```

Zniesienie następuje w losowym czasie z przedziału min–max. Możesz to zastąpić:
`"lay_time": { "min": 9000, "max": 15000 }` (min ≤ max; max zostanie podniesione do min, jeśli jest mniejsze).
`lay_time_coefficient` < 1 = szybciej, > 1 = wolniej. W Roost/Breeder czas dzieli dodatkowo
`roostSpeed` / `breederSpeed` z `chickens-common.toml` oraz statystyka *Growth* kurczaka.

---

## 6. Naturalny spawn

### W JSON-ie
```json
"spawn": { "biomes": "#minecraft:is_badlands", "weight": 5 }
```
- `biomes`: tag (`"#minecraft:is_forest"`), pojedynczy biom (`"minecraft:desert"`) albo lista:
  `["minecraft:plains", "#minecraft:is_taiga"]`.
- `weight` (1–1000, domyślnie 10): w biomie, w którym może spawnować się kilka kurczaków, losowanie jest ważone.

Kurczak **bez** bloku `spawn` nie spawnuje się naturalnie (nadal można go dostać z krzyżowania, komendą lub łapaczem).

Przydatne tagi biomów z Minecrafta: `#minecraft:is_overworld`, `is_nether`, `is_end`, `is_forest`, `is_taiga`, `is_jungle`,
`is_savanna`, `is_badlands`, `is_mountain`, `is_beach`, `is_ocean`, `is_river`, `is_hill`. Tagi mini-moda:
`#chickens:spawn/normal` (lasy, sawanny, dżungla, tajga, równiny, łąki, cherry grove), `#chickens:spawn/snow`,
`#chickens:spawn/nether`, `#chickens:spawn/end`.

Mod dodaje encję kurczaka do biomów Overworldu, Netheru i Endu – o tym, *który* kurczak się pojawi, decyduje spawn z JSON-a
lub configu. Poza Overworldem kurczaki mogą stawać na każdym stałym bloku (netherrack, end stone…), w Overworldzie – jak
zwierzęta, na trawie.

### W configu – `config/chickens-spawns.toml`
Config **wygrywa** z JSON-em i jest najwygodniejszy do dostrojenia bez edycji paczek.

```toml
[spawns]
    # każdy wbudowany kurczak ma wpis: lista selektorów biomów
    ender = ["end"]                       # tylko End
    log   = ["#minecraft:is_forest"]      # tylko lasy
    white = ["nether"]
    smart = []                            # [] = nigdy naturalnie
    flint = ["datapack"]                  # "datapack" = bierz blok spawn z JSON-a

# kurczaki z datapacków (i nadpisanie dowolnego innego): "id = selektor, selektor"
extraSpawns = ["mypack:copper = #minecraft:is_badlands, minecraft:desert"]

# wagi (domyślnie 10 albo z JSON-a)
weights = ["mypack:copper = 5", "chickens:ender = 3"]
```

Selektory: `overworld`, `nether`, `end`, `any`, biom (`minecraft:plains`), tag biomów (`#minecraft:is_forest`).
Błędne wpisy są pomijane z ostrzeżeniem w logu. Zmiany działają po restarcie świata/serwera.

---

## 7. Własności (`properties`)

| wartość | efekt |
|---|---|
| `fire_immune` | odporny na obrażenia od ognia i lawy |
| `explosion_immune` | odporny na wybuchy |
| `drowning_immune` | odporny na utonięcie |
| `freeze_immune` | odporny na zamarzanie (powder snow) – nie zamarza i nie dostaje obrażeń od zimna |
| `hover` | w powietrzu spada dwa razy wolniej (szybuje) |

---

## 8. Tekstury, ikony i nazwy

### Tekstura kurczaka w świecie
PNG **64×32**, układ jak tekstura vanilla kurczaka (`minecraft:textures/entity/chicken/chicken.png`).
Domyślna ścieżka: `assets/<ns>/textures/entity/chicken/<nazwa>.png`; inną wskażesz polem `texture`
(np. `"texture": "chickens:textures/entity/iron_chicken.png"` – możesz użyć gotowych tekstur z moda).
Tekstura z większą rozdzielczością (np. 128×64) też działa.

### Ikona przedmiotu (16×16)
Ręcznie rysowany sprite: `assets/<ns>/textures/item/chicken/<nazwa>.png` (zwykły kurczak: `chickens:vanilla`).
Mod rysuje go jak zwykły przedmiot (z grubością). **Bez tego pliku** kurczak dostaje wspólny szablon zabarwiony kolorami
wyliczonymi z jego tekstury – wygląda poprawnie, ale nie unikalnie. Gotowe ikony wbudowanych kurczaków leżą w
`assets/chickens/textures/item/chicken/` – możesz je podejrzeć jako wzór rozmiaru i stylu.

### Nazwa
Klucz `chicken.<namespace>.<nazwa>` w pliku lang:

```json
{ "chicken.mypack.salt": "Salt Chicken" }
```

Dla `mypack:ores/tin` klucz to `chicken.mypack.ores.tin`. Bez wpisu nazwa powstaje automatycznie: `Salt Chicken`.

---

## 9. Wyłączanie i nadpisywanie kurczaków

- **Wyłączenie**: plik o tym samym id co wbudowany kurczak, np. `data/chickens/chickens/chicken_type/smart.json`:
  ```json
  { "lay_item": { "id": "minecraft:egg" }, "enabled": false }
  ```
  Wyłączony kurczak znika z krzyżowania, spawnu i JEI; **jego potomkowie również** (są wyłączeni, jeśli którykolwiek
  przodek jest wyłączony).
- **Nadpisanie**: ten sam plik z pełną, zmienioną definicją (inny `lay_item`, czas, własności…) – wygrywa datapack
  wczytany później (kolejność w menu datapacków).
- Parents mogą wskazywać dowolnego kurczaka, także z innego pakietu.

---

## 10. Pliki config

**`config/chickens-common.toml`**

| klucz | domyślnie | opis |
|---|---|---|
| `roostSpeed` | `1.0` | mnożnik szybkości Roost |
| `breederSpeed` | `1.0` | mnożnik szybkości Breeder |
| `vanillaChickensLayEggsInRoost` | `true` | `false` = vanilla kurczak w Roost daje tylko pióra |
| `alwaysShowStats` | `false` | pokazuj Growth/Gain/Strength także dla nieprzeanalizowanych |
| `henhouseRadius` | `4` | promień (bloki), w którym kurczaki dostarczają jajka do kurnika |
| `breedingChances` | `[]` | szanse krzyżowania, `"id = procent"` |

**`config/chickens-spawns.toml`** – patrz [rozdz. 6](#w-configu---configchickens-spawnstoml).

---

## 11. Wbudowane kurczaki

`vanilla` (`minecraft:chicken`) oraz 47 z namespace `chickens`:
black, blaze, blue, brown, clay, coal, cyan, diamond, emerald, ender, flint, ghast, glass, glowstone, gold, gray, green,
gunpowder, iron, lava, leather, light_blue, light_gray, lime, log, magenta, magma, netherwart, obsidian, orange, pink,
prismarine_crystal, prismarine_shard, purple, quartz, red, redstone, sand, slime, smart, snowball, soul_sand, string,
water, white, yellow.

Domyślny spawn: `white`, `flint`, `quartz`, `soul_sand` – Nether; `ender` – End; `log`, `sand` – biomy z
`#chickens:spawn/normal`; reszta tylko z krzyżowania.

---

## 12. Rozwiązywanie problemów

| problem | przyczyna / rozwiązanie |
|---|---|
| Kurczak się nie pojawia | Sprawdź literówki w JSON-ie (log: `Failed to load ... chicken_type`). Zmiany wczytują się przy wejściu do świata, nie przez `/reload`. |
| Kurczak nie spawnuje się naturalnie | Brak bloku `spawn` albo biom nie pasuje; sprawdź też `chickens-spawns.toml` (wpis `[]` blokuje spawn). Spawn zależy też od światła/podłoża jak u zwierząt (w Overworldzie trawa). |
| Nazwa wygląda jak „Salt Chicken”, a mam inną w lang | Klucz musi być dokładnie `chicken.<ns>.<nazwa>`, a paczka włączona jako **resource pack**. |
| Biała/fioletowa-czarna tekstura | Zła ścieżka lub rozmiar; sprawdź `texture` i czy plik leży w `assets/<ns>/textures/entity/chicken/`. |
| Ikona to wspólny szablon | Brak pliku `assets/<ns>/textures/item/chicken/<nazwa>.png` w resource packu. |
| Krzyżowanie nie daje mojego kurczaka | Dokładnie ta para rodziców w `parents`? Kurczak i jego przodkowie nie mogą mieć `enabled: false`. Szansa < 100 % – wynik jest losowy. |
| JEI pokazuje inną szansę niż w grze | Różne pliki `chickens-common.toml` u klienta i serwera. |

---

## 13. Specyfika wersji 26.1.2

**Stan:** moduł NeoForge działa w grze (kurczaki, Roost, Breeder, Collector, Henhouse, Gniazdo, jajka, spawn eggi, datapacki). **Fabric** dla 26.1.2 nie jest jeszcze uruchomiony.

- Wymagania: **Java 25**, Gradle 9.x, **NeoForge 26.1.2.x** (wersja z `gradle.properties`, zakres `[26.1.2,)`).
- Zakres wersji w `neoforge.mods.toml` musi być otwarty w górę (`[26.1.2,)`); zapis `[26.1.2]` odrzuca NeoForge `26.1.2.124`.
- Format `lay_item` / `drop_item` i komendy `/give` są takie same jak w 1.21.1 (komponenty).
- Przedmioty mają definicje w `assets/chickens/items/*.json` (wymóg nowych wersji). Kolory kolorowych jajek, płynnych jajek i spawn eggów dają własne źródła koloru (`ModTintSources`).
- Przedmiot „Kurczak” rysuje specjalny model przedmiotu (`ChickenItemRenderer`), według tej samej konwencji plików ikon: `assets/<ns>/textures/item/chicken/<nazwa>.png`.
- Roost i Gniazdo mają własne renderery (render state / `submit`); mały kurczak ma stary model z 1.21.1 (własna geometria), z animacjami, mignięciem po obrażeniach i obrotem przy śmierci.
- Pliki lang kurczaków: `en_us.json` i `pl_pl.json` (po 94 klucze) – wszystkie 46 wbudowanych typów ma polskie nazwy.
- **Znane braki w chwili pisania (sprawdź w swojej kompilacji):** automatyzacja lejami / rurami (nowe API transferu NeoForge) i tint płynów z innych modów były wyłączone na czas portu; Fabric nie jest jeszcze uruchomiony.

### Budowanie (26.1.2)
- `./gradlew :common:compileJava` – kompilacja wspólnego kodu.
- `./gradlew :neoforge:runClient` – test w grze.
- W razie błędów kompilacji narzędzie `tools/errlog.py build.log --md errors.md` robi zwięzłe podsumowanie.