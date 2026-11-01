# Kartice (upgrade-i iz chestova)

Šta je trenutno napravljeno. Spisak ideja za buduće kartice je u `progresija.md` §6.

## Kako radi

1. Igrač priđe chestu (200 jedinica). Chest odigra `Chest_Open` i **ostaje otvoren zauvek** (jedan chest nudi karte samo jednom).
2. Miš se otključa, vreme ide 0.1×, preko ekrana (UI sloj) dolazi crni overlay sa 50% providnosti i 3 kartice.
3. Kartica se bira **mišem** (klik na karticu, ona preko koje je miš se uvećava) ili tasterima **1/2/3**.
4. Kartice i overlay nestaju, vreme ide normalno, miš se ponovo zaključava (isto kao pri prvom kliku).
5. Upgrade se stackuje i važi do kraja runa, **i između levela** (health i shield takođe).

Dok su kartice otvorene, ne radi ni melee ni pucanje.

## Retkost

Svaka od 3 kartice u ponudi dobija retkost nezavisno, a 3 kartice su uvek 3 različita upgrade-a. Retkost ne množi kartice, nego svaka kartica ima svoju vrednost za svaku retkost (tabele ispod).

| Retkost | Boja okvira |
|---|---|
| Common | siva |
| Uncommon | zelena |
| Rare | plava |
| Epic | ljubičasta |
| Mythic | zlatna |

**Težine** (l = Level − 1): Common `max(25, 55 − 3·l)`, Uncommon `26 + l`, Rare `13 + l`, Epic `5 + 0.7·l`, Mythic `1 + 0.3·l`.

| Level | Common | Uncommon | Rare | Epic | Mythic |
|---|---|---|---|---|---|
| 1 | 55% | 26% | 13% | 5% | 1% |
| 5 | 43% | 30% | 17% | 7.8% | 2.2% |
| 10 | 28% | 35% | 22% | 11.3% | 3.7% |
| 15 | 22.3% | 35.7% | 24.1% | 13.2% | 4.6% |

Common pada za 3 po levelu do praga 25, ostale retkosti rastu. Dublji chestovi su bolji, ali ništa nije zagarantovano.

**Seed:** ponuda se računa iz `hash(RunSeed, Level, x i y chesta)`, pa isti RunSeed daje iste kartice na istom chestu. Ne koristi se `random()`.

## Kartice po bojama

Vrednosti su po retkosti: Common / Uncommon / Rare / Epic / Mythic.

### Plava, shield

| Kartica (slika) | Naziv | Šta radi | Vrednosti |
|---|---|---|---|
| `shield` | Shield | +shield i odmah puni; ako je preko maksimuma, diže i maksimum. **Pokreće tajmer** (30 s, vidi „Tajmeri“) | 10 / 15 / 22 / 35 / 50 |
| `healing-shield` | Shield+ | +max shield **i isto toliko shield poena odmah** (10/100 + 25 → 35/125). Ako je shield bio istekao, ponovo ga pali i pokreće tajmer | 5 / 8 / 11 / 18 / 25 |
| `shield-time` | Shield Time | trajno produžava trajanje shield-a za 5 s, **puni shield do maksimuma i restartuje tajmer** (kao novi shield). Nudi se tek kad imaš max shield > 0, **cap 120 s** | **+5 s** za svaku retkost |
| `shield-reflect` | Reflect | šansa da se neprijateljski projektil vrati nazad (samo dok je shield > 0); vraćen projektil nanosi 8 neprijatelju u koga uleti | 5 / 8 / 11 / 18 / 25 %, **cap 75%** |

Shield ide preko Health ekstenzije (`MaxShield`, `ShieldPoints`). Native trajanje Health-a je 0 (ne ističe), a isticanje radi naš tajmer (vidi „Tajmeri“). Prikaz: plava providna sfera oko igrača, providnost = `0.5 · shield / maxShield`, i plava traka (`BlueFlatBar`, UI sloj) iznad crvene, skrivena dok nema shield-a.

### Narandžasta, kretanje

| Kartica | Naziv | Šta radi | Vrednosti |
|---|---|---|---|
| `running-shoe` | Running Shoes | + max brzina | +12 / 18 / 27 / 42 / 60, početno **300**, **cap 700** |
| `jump-across` | Jump | **prva otključava skok** (bez nje Space ne radi), kasnije + visina skoka | 8 / 12 / 18 / 28 / 40 % |

Kartica brzine se ne nudi kad je brzina na 700.

### Zelena, healing

| Kartica | Naziv | Šta radi | Vrednosti |
|---|---|---|---|
| `arm-bandage` | Bandage | odmah leči | 20 / 30 / 45 / 70 / 100 |
| `healing` | Regeneration | + health regen u sekundi | 0.5 / 0.75 / 1.1 / 1.75 / 2.5 |
| `health-increase` | Vitality | + max health, i leči za isto toliko | 10 / 15 / 22 / 35 / 50 |

### Crvena, napad

| Kartica | Naziv | Šta radi | Vrednosti |
|---|---|---|---|
| `falling-blob` | Heavy Shot | **prva otključava pucanje** (desni klik), kasnije + šteta i + domet projektila | 8 / 12 / 18 / 28 / 40 % |
| `ricochet` | Ricochet | + odbijanja projektila od zida (početno 0) | 1 / 1 / 1 / 2 / 3 |
| `fire-tail` | Chain Shot | projektil posle pogotka skače na najbližeg neprijatelja koji nije pogođen (do 800 daleko), ista šteta | 1 / 1 / 1 / 2 / 3 skokova |
| `punch-blast` | Punch Blast | + šteta melee udarca i + domet melee-ja (domet raste 0.6 × vrednost, 60 → **max 120**) | 10 / 15 / 22 / 35 / 50 % |

`ricochet` i `fire-tail` se **ne nude dok se ne otključa pucanje** (nemaju efekta bez njega).

### Žuta, pasivno

| Kartica | Naziv | Šta radi | Vrednosti |
|---|---|---|---|
| `orbital` | Orbital | + kugla koja kruži oko igrača (200°/s, radijus 130) i povređuje neprijatelje, isti neprijatelj najviše na 0.45 s. **Pokreće tajmer** (30 s) | 1 / 1 / 1 / 2 / 3, **cap 4** |
| `orbital-time` | Orbital Time | trajno produžava trajanje orbova za 5 s, **vraća sve orbove koje si do sada skupio** (broj se pamti i kad istekne tajmer) i restartuje tajmer. Ne dodaje novi orb. Nudi se tek kad imaš bar jedan orb, **cap 120 s** | **+5 s** za svaku retkost |

Šteta orbitala je `6 · (1 + šteta%) · TotalDamageMultiplier`. Objekat je `PlayerOrbital` (žuta kopija `EnemyProjectile`).

## Tajmeri (shield i orbovi)

Shield i orbovi nisu trajni, jer bi bili prejaki.

- **Početno trajanje je 30 s** za svaki (`shieldDur`, `orbDur`).
- **Shield:** kartica `shield` ga puni i pokreće tajmer. Kad tajmer istekne, shield se spušta na 0 (health ostaje). Ako shield potrošiš pre toga, tajmer staje. Poslednje 2 sekunde sfera treperi.
- **Orbovi:** kartica `orbital` dodaje orbove i vraća tajmer na pun. Kad istekne, orbovi nestaju, a broj orbova ostaje zapamćen, pa nova `orbital` kartica vraća sve.
- **Kartice za vreme** (`shield-time`, `orbital-time`) trajno povećavaju trajanje za 5 s (30 → 35 → 40…, cap 120 s) i **pale efekat ponovo**: shield se puni do maksimuma, a orbovi se vraćaju u broju koji si do sada skupio (ne dobijaju novi), i oba tajmera kreću od pune (novog) trajanja.
- Tajmeri se računaju sa vremenom igre, pa stoje dok su kartice otvorene (0.1× brzina) i **prelaze u sledeći level**.
- **Odbrojavanje iznad igrača:** sekunde do isteka, **plavo** za shield i **žuto** za orbove. Ako teku oba, stoje jedno iznad drugog (orbovi niže).
- Vrednosti su `shieldDur`, `shieldT`, `orbDur`, `orbT` u `gdjs.amazeUpg.U`.

## Početne vrednosti i borba

| Stat | Početno | Napomena |
|---|---|---|
| Max brzina | 300 | cap 700, polovina u BlueMist |
| Skok | zaključan | visina 100 + % |
| Pucanje | zaključano | desni klik, šteta 10, domet 4 s, odbijanja 0, lanac 0 |
| Melee | otključan | levi klik, animacija `Weapon`, šteta 15, domet 60 (max 120), konus 65° ispred igrača, meri se do ivice neprijatelja |
| Level | 1 | globalna promenljiva |

Sve ove šteta se množe sa `Player::TotalDamageMultiplier()`. Melee udara na `MeleeMoment = 0.35 s` u animaciji (procena, treba podesiti).

## Gde je šta u projektu

| Šta | Gde |
|---|---|
| Podaci o karticama (`CARDS`), retkosti, `apply`, `available`, `defaults`, rolovanje, crtanje kartice, melee, lanac, reflect, orbitali, shield sfera | `eventsFunctionsExtensions/amaze.json` → funkcija `onFirstSceneLoaded`, poslednji JS blok (`gdjs.amazeUpg`) |
| Stanje runa (preživljava promenu scene) | `gdjs.amazeUpg.U` |
| Svaki frejm | `Amaze::UpgradesStep` (događaj u sceni posle postavljanja seed-ova) |
| Chest, miš, desni/levi klik, brzina, skok | događaji u `layouts/untitled-scene.json` (traže `OpenChest`, `ExitPointerLock`, `CardJustPicked`, `ShotUnlocked`, `UpgradeMaxSpeed`...) |
| Melee stanje i animacija | `Amaze::PlayerFSM` (`Melee`, `MeleeMomentReached`, `MeleeMoment`, `MeleeDuration`) |
| Slike kartica | `assets/cards/<id>.png`, resurs `card_<id>.png`; objekat `CardIcons` (Sprite) je tu samo da ih izvoz ne izbaci |

Novi run počinje kad je `Level` manji nego prošli put ili kad se promeni `RunSeed`.

## Kako dodati novu karticu

1. Stavi sliku u `assets/cards/<id>.png` i dodaj resurs `card_<id>.png` (u `aMAZE.json`, `file` `assets/cards/<id>.png`).
2. U objekat `CardIcons` dodaj animaciju sa tom slikom. Bez toga izvoz ne uključuje sliku, jer se u kodu poziva samo po imenu.
3. U `CARDS` dodaj `{ id, title, cat, v: [C, U, R, E, M], text: (v, U) => '...' }`. `cat` je jedno od `shield`, `movement`, `healing`, `attack`, `passive` (to određuje boju kartice). (Ako bi kartica ipak delila sliku sa drugom, dodaj `icon: '<id druge kartice>'` i koraci 1 i 2 otpadaju.)
4. U `G.apply` dodaj `case` koji menja `G.U`.
5. Ako kartica ima cap ili ne smije uvek da se nudi, dodaj uslov u `G.available`.
6. Ako treba novi stat, dodaj ga u `G.defaults`, pa ga primeni tamo gde se koristi (`G.push` za Health i fiziku, `UpgradesStep` za ostalo, ili izraz kao `UpgradeMaxSpeed`).
7. Da se stat vidi u panelu: dodaj `case` u `G.stat` (vrati formatiran tekst) i jednu `BBText::SetBBText` akciju u događaju panela, sa bojom kategorije.

## Panel statistika (`NewStatsText`)

BBText objekat na `UI` sloju (gore desno). Puni ga događaj u sceni odmah posle onog za `LevelText`: jedna akcija `BBText::SetBBText` po liniji (prva `=`, ostale `+` sa `NewLine()`). Labela, boja i redosled su u event sheet-u, a vrednost daje izraz `Amaze::Stat("ključ")` (formatiran tekst, npr. `85 / 125`, `locked`, `24s / 35s`).

| Boja | Linije (ključ) |
|---|---|
| zelena | `health`, `regen` |
| plava | `shield`, `shieldTime`, `reflect` |
| narandžasta | `speed`, `jump` |
| crvena | `shooting`, `shotDamage`, `shotRange`, `bounces`, `chain`, `melee`, `reach` |
| žuta | `orbs`, `orbTime`, `orbDamage` |
| po kategoriji kartice | `picked`: BBCode lista uzetih kartica (`Ricochet x2, ...`) |

Brojevi štete uključuju `TotalDamageMultiplier`. Stari objekat `StatsText` (običan Text) više nije povezan i može da se obriše.

## Zamke

- `getCenterX/Y/Z()` u JS-u su pomeraji od ugla objekta, ne pozicije u sceni. Koristi `getCenterXInScene()`, `getCenterYInScene()` i `getZ() + getDepth()/2` (u kodu `G.cx`, `G.cy`, `G.cz`).
- Vrednosti iz Health ponašanja kroz JS mogu biti stringovi: obavezno `Number(...)`, inače `+15` spaja tekst.
- Pointer lock se otključava u događaju chesta, a ponovo zaključava događajem `CardJustPicked`. Događaj prvog klika ne radi dok su kartice otvorene.

## Nije proveravano u igri

Sve gore je napisano bez preview-a. Treba pogledati: izgled i tekst kartica, `MeleeMoment`, veličinu shield sfere, vatru orbitala (ostaje li iza njih), i da se miš dobro zaključava posle izbora.
