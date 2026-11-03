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
| `shield` | Shield | +shield poena, ali nikad preko maksimuma (početni max je **50**); ne nudi se dok je shield pun | 10 / 15 / 22 / 35 / 50 |
| `healing-shield` | Shield+ | samo diže max shield, ne dodaje shield poene | 5 / 8 / 11 / 18 / 25 |
| `shield-reflect` | Reflect | šansa da se neprijateljski projektil vrati nazad (samo dok je shield > 0); vraćen projektil nanosi neprijatelju u koga uleti 60% svoje štete (šteta projektila = `Damage` onoga ko ga je ispalio, `BASE.reflectDmg`); nad neprijateljem se vidi plavo `-N` | 5 / 8 / 11 / 18 / 25 %, **cap 75%** |

Shield ide preko Health ekstenzije (`MaxShield`, `ShieldPoints`). Nema tajmer: Health-ovo trajanje je 0 (ne ističe), shield se troši samo udarcima. Prikaz: plava providna sfera oko igrača, providnost = `0.5 · shield / maxShield`, i plava traka (`BlueFlatBar`, UI sloj) iznad crvene, skrivena dok nema shield-a.

### Narandžasta, kretanje

| Kartica | Naziv | Šta radi | Vrednosti |
|---|---|---|---|
| `running-shoe` | Running Shoes | + max brzina | +12 / 18 / 27 / 42 / 60, početno **300**, **cap 700** |
| `jump-across` | Jump | **prva otključava skok** (bez nje Space ne radi), kasnije + visina skoka | 8 / 12 / 18 / 28 / 40 % |
| `phase-time` | Phase Time | duža neranjivost posle udarca (početno 0.5 s) | +0.1 / 0.15 / 0.2 / 0.3 / 0.4 s, cap +1.5 s |
| `dash` | Dash | **prva otključava dash** (Shift, cooldown 3 s): kratak nalet (0.15 s, brzina 1400) u pravcu u kom se krećeš, a ako stojiš, napred; nepovrediv dok traje. Kasnije skraćuje cooldown | −0.15 / 0.2 / 0.3 / 0.45 / 0.6 s, **min 0.8 s** |

Kartica brzine se ne nudi kad je brzina na 700.

### Zelena, healing

| Kartica | Naziv | Šta radi | Vrednosti |
|---|---|---|---|
| `arm-bandage` | Bandage | odmah leči | 20 / 30 / 45 / 70 / 100 |
| `healing` | Regeneration | + health regen u sekundi | 0.2 / 0.3 / 0.45 / 0.7 / 1 |
| `health-increase` | Vitality | samo diže max health, ne leči | 10 / 15 / 22 / 35 / 50 |
| `soul-catcher` | Soul Catcher | deo **stvarno nanete štete** se vraća kao HP (računa se pad zdravlja neprijatelja u `G.trackDamage`, pa overkill ne daje ništa; važi za sve izvore) | 2 / 3 / 5 / 8 / 12 %, **cap 25%** |
| `quick-recovery` | Quick Recovery | regen kreće ranije posle udarca (početno čekanje **10 s**, min **3 s**); nudi se tek kad imaš regen | −1 / 1 / 1.5 / 2 / 3 s |
| `dodge` | Dodge | šansa da izbegneš udarac (`ChanceToDodge` u Health) | 3 / 5 / 7 / 10 / 15 %, **cap 50%** |
| `second-chance` | Second Chance | **+1 život**: kad bi umro, vraćaš se sa delom max HP-a i 1.5 s si nepovrediv. Život se troši i ne vraća. HP pri oživljavanju je najbolja vrednost koju si birao | HP 25 / 35 / 50 / 70 / 100 %, **max 3 života** |

### Crvena, napad

| Kartica | Naziv | Šta radi | Vrednosti |
|---|---|---|---|
| `falling-blob` | Heavy Shot | **prva otključava pucanje** (desni klik), kasnije + šteta (%) | šteta 8 / 12 / 18 / 28 / 40 % |
| `projectile-timer` | Long Shot | + domet projektila u **sekundama** (osnovni domet je 0.5 s, oko 300 jedinica) | +0.5 / 0.5 / 0.75 / 1 / 2 s |
| `ricochet` | Ricochet | + odbijanja projektila od zida (početno 0) | 1 / 1 / 1 / 2 / 3 |
| `fire-tail` | Chain Shot | posle pogotka iz neprijatelja izlazi **pravi projektil** (isti kao prvi hitac, ista šteta) ka najbližem neprijatelju koji nije pogođen, **u vidokrugu** (isti red ili kolona lavirinta bez zida) i **najviše 800 daleko** (2 polja). Nema odbijanja, staje na prvom zidu; kratko ne može da pogodi neprijatelja iz kog je izašao | 1 / 1 / 1 / 2 / 3 skokova |
| `punch-blast` | Punch Blast | + šteta melee udarca | 10 / 15 / 22 / 35 / 50 % |
| `punch-range` | Long Fist | + domet melee-ja (60 → **max 120**); ne nudi se kad je cap dostignut | 6 / 9 / 13 / 21 / 30 |
| `crit-chance` | Crit Chance | šansa za kritični pogodak (osnovno **×2** štete) na hitac, melee i orbove; pogodak daje dodatnu eksploziju | 3 / 5 / 7 / 10 / 15 %, **cap 60%** |
| `crit-damage` | Crit Damage | + šteta kritika (×2 → max **×5**); nudi se tek kad imaš Crit Chance | 20 / 30 / 45 / 70 / 100 %, **cap 300%** |
| `executioner` | Executioner | + šteta protiv neprijatelja (i spawnera) ispod **30% HP** | 10 / 15 / 22 / 35 / 50 %, cap 150% |
| `knockback` | Knockback | hitac i melee guraju **živog** neprijatelja unazad (klizi sa otporom, staje na zidu lavirinta, ne radi na `SuperArmor = Yes` ni na ulozi `Stationary`, tj. mushnubi) | +40 / 60 / 90 / 140 / 200 (piksela), **cap 400** |
| `rapid-fire` | Rapid Fire | brže pucanje: kraći `FireMoment` i `ShootDuration` u `PlayerFSM`, animacija ubrzana isto | 8 / 12 / 18 / 28 / 40 %, cap 150% |
| `fast-bullets` | Fast Bullets | brži metak (600 →), a kako je domet u sekundama, i dalje stiže | 8 / 12 / 18 / 28 / 40 %, **cap 150% (1500)** |
| `double-shot` | Double Shot | + metaka u jednoj salvi, u luku (razmak 8°). Svaki metak je slabiji: šteta salve je `1 + 0.6·extra` šteta jednog hica, podeljena na sve metke | 1 / 1 / 1 / 2 / 3, **cap 4 dodatna** |
| `steady-aim` | Steady Aim | manje rasipanje hitaca (početno **±7°**); ne nudi se na 0 | −1 / 1.5 / 2 / 3 / 4 ° |

`ricochet`, `fire-tail`, `projectile-timer`, `rapid-fire`, `fast-bullets`, `double-shot` i `steady-aim` se **ne nude dok se ne otključa pucanje** (nemaju efekta bez njega).

### Žuta, pasivno

| Kartica | Naziv | Šta radi | Vrednosti |
|---|---|---|---|
| `orbital` | Orbital | + kugla koja kruži oko igrača (200°/s, radijus 130) i povređuje neprijatelje. **Svaka kugla nestaje kad pogodi neprijatelja; kad nestanu sve, kreće cooldown** (30 s), pa se sve vrate. Nova kartica vraća sve odmah | 1 / 1 / 1 / 2 / 3, **cap 4** |
| `crown-of-thorns` | Crown of Thorns | neprijatelj koji te udari u melee dobija deo štete nazad (samo kad udarac stvarno prođe: ne dodge, ne cooldown) | 25 / 35 / 50 / 75 / 100 %, cap 200% |
| `orbital-time` | Orbital Time | trajno smanjuje cooldown orbitala za 1 s (min **10 s**). Nudi se tek kad imaš bar jedan orb | **−1 s** za svaku retkost |

Šteta orbitala je `6 · (1 + šteta%) · TotalDamageMultiplier`. Objekat je `PlayerOrbital` (žuta kopija `EnemyProjectile`).

## Cooldown orbova

Orbovi su jaki, pa se posle pogotka troše.

- Svaki orb **nestaje kad pogodi neprijatelja** (šteta se računa normalno). Kad nestane poslednji, kreće **cooldown od 30 s** (`orbCd`), pa se vraćaju svi orbovi.
- Kartica `orbital` dodaje orb i odmah vraća sve orbove, bez cooldown-a. `orbital-time` skraca cooldown za 1 s (min 10 s).
- Cooldown se računa sa vremenom igre, pa stoji dok su kartice otvorene i **prelazi u sledeći level**.
- **Ikonice pasiva** su iznad XP bara na UI sloju (ikonica kartice), `G.passiveHud`: orbital i crown-of-thorns, lista `PASSIVES`. Tokom cooldown-a ikonica je zatamnjena i preko nje je **žuto** odbrojavanje.
- Vrednosti su `orbCd`, `orbCdT` i `orbLive` u `gdjs.amazeUpg.U`.

## Početne vrednosti i borba

| Stat | Početno | Napomena |
|---|---|---|
| Max brzina | 300 | cap 700, polovina u BlueMist |
| Skok | zaključan | visina 100 + % |
| Pucanje | zaključano | desni klik, šteta 10, domet 0.5 s (brzina 600, kao neprijateljski projektil), odbijanja 0, lanac 0, rasipanje **±7°**, 1 metak |
| Regen | 0 | kad se doda, čeka **10 s** posle udarca **ili hica** (`R.lastShot`) |
| Neranjivost posle udarca | 0.5 s | `DamageCooldown` u Health; Dodge početno 0 |
| Melee | otključan | levi klik, animacija `Weapon`, šteta 5, domet 60 (max 120). Sečivo (`G.meleeTick`) maše lukom od −65° do +65° ispred igrača (s leva na desno) u prozoru `MeleeMoment`–`MeleeHitEnd`, dužine jednake dometu; svaki neprijatelj se pogađa najviše jednom po zamahu. Kost šake se koristi samo ako daje pravu poziciju. Trag `FireEmbers` je na vrhu sečiva, flow (čestica/s) = jačina melee napada |
| Level | 1 | globalna promenljiva |

Sve ove šteta se množe sa `Player::TotalDamageMultiplier()`. Melee je aktivan u prozoru `MeleeMoment = 0.35 s` do `MeleeHitEnd = 0.55 s` u animaciji (procena, treba podesiti po tragu). Događaj u sceni je `Amaze::MeleeUpdate` (svaki frame).

## Gde je šta u projektu

| Šta | Gde |
|---|---|
| Podaci o karticama (`CARDS`), retkosti, `apply`, `available`, `defaults`, rolovanje, crtanje kartice, melee, lanac, reflect, orbitali, shield sfera | `eventsFunctionsExtensions/amaze.json` → funkcija `onFirstSceneLoaded`, poslednji JS blok (`gdjs.amazeUpg`) |
| Stanje runa (preživljava promenu scene) | `gdjs.amazeUpg.U` |
| Svaki frejm | `Amaze::UpgradesStep` (događaj u sceni posle postavljanja seed-ova) |
| Chest, miš, desni/levi klik, brzina, skok | događaji u `layouts/untitled-scene.json` (traže `OpenChest`, `ExitPointerLock`, `CardJustPicked`, `ShotUnlocked`, `UpgradeMaxSpeed`...) |
| Ispaljivanje (salva, rasipanje), pogodak hica, melee udarac neprijatelja, dash | funkcije `Amaze::FireShot`, `Amaze::ShotHit`, `Amaze::EnemyMeleeHit`, `Amaze::Dash` (pozivaju `G.fire`, `G.shotHit`, `G.enemyMeleeHit`, `G.dash`); u sceni zamenjuju `FireForwardObject3D` i `Health::Hit` |
| Šteta igrača (krit, executioner, guranje) | `G.strike(o, dmg, p, shove)`; koriste ga hitac, melee i orbovi. `G.hit` je goli udarac (reflect, thorns) |
| Second Chance | `G.rescue(o)`, poziva se na kraju `Health::Hit` u `eventsFunctionsExtensions/health.json` (JS događaj na kraju funkcije; ne radi ništa za neprijatelje) |
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
7. Svaka nova kartica treba i red u tabeli `UNLOCK` (od kog ranka se nudi). Bez reda je dostupna od ranka 1.
8. Da se stat vidi u panelu: dodaj `case` u `G.stat` (vrati formatiran tekst) i `add(...)` red u `G.panel`, u grupi boje kategorije i sa uslovom kad treba da se prikaže.

## XP i rank

**Rank** je nivo igrača od XP-a (da se ne meša sa **Level**, nivoom lavirinta). Počinje od **1**, preživljava smrt i novi level, i **otključava kartice i retkosti**, ne daje snagu.

- **XP po šteti**: 1 izgubljeni poen zdravlja neprijatelja = 1 XP (puta polje `XP` na tipu neprijatelja, podrazumevano 1), pa 10 štete = 10 XP. Računa se stvarni pad zdravlja svaki frame (`G.trackDamage`), a ne sirova šteta, pa jači udarac ne daje više XP-a od zdravlja koje je neprijatelju ostalo (kartice za štetu ne ubrzavaju rank). Važi za sve izvore: hitac, melee, orbovi, lanac. `Amaze::AwardKillXP` (događaj `JustDied`) samo dodaje zdravlje koje nije stiglo da se izbroji.
- **XP se upisuje tek kad se level pobedi**: dodir vortexa poziva `Amaze::BankXP` (prvi korak događaja sa vortexom). Ako igrač umre ili ponovo učita level, XP tog levela se gubi, a već upisani XP ostaje.
- **Čuvanje:** za sada **nigde**. XP živi samo u memoriji (`gdjs.amazeUpg.rank`) i posle ponovnog učitavanja projekta kreće od 0, tj. rank 1. Čuvanje po runu dolazi kasnije.
- **Kriva** (linearna): za prelazak sa ranka R na R+1 treba `rankXP · R` XP, `rankXP = 10` (`G.BASE.rankXP`, jedino mesto za podešavanje tempa). Rank ne daje snagu, samo otključava kartice.

| Rank | Ukupno XP za taj rank |
|---|---|
| 10 | 450 |
| 20 | 1900 |
| 30 | 4350 |
| 40 | 7800 |
| 50 | 12250 |
| 60 | 17700 |

- **Otključavanja** (tabela `UNLOCK` u biblioteci; kartica koja nije u tabeli je otključana od ranka 1):

| Rank | Otključava |
|---|---|
| 1 | arm-bandage, health-increase, running-shoe, shield, punch-blast |
| 3 | punch-range |
| 8 | falling-blob (prvi pucanj) |
| 20 | healing, projectile-timer |
| 30 | jump-across, healing-shield |
| 40 | ricochet, orbital, retkost **Epic** |
| 50 | shield-reflect, fire-tail |
| 60 | orbital-time, retkost **Mythic** |

Nove kartice: rank 5 `crit-chance`, `dodge`; 10 `knockback`, `rapid-fire`; 12 `steady-aim`; 15 `executioner`, `phase-time`; 25 `fast-bullets`, `soul-catcher`, `quick-recovery`; 35 `crit-damage`, `dash`; 45 `double-shot`, `crown-of-thorns`; 55 `second-chance`. (Tabela `UNLOCK` je jedino mesto za podešavanje.)

Zaključana retkost se ne bira (težine ostalih se preraspodeljuju). Pošto se skup kartica menja sa rankom, isti `RunSeed` daje drugačiju ponudu na različitom ranku.

**Izrazi** (za progress bar i tekst). `Rank`, `RankXP` i `RankGoal` su **živi**: broje i XP skupljen na levelu koji se igra, pa se traka puni odmah posle svakog ubistva. Ako igrač umre, XP tog levela nestaje i traka se vraća na upisanu vrednost. Kartice se otključavaju samo **upisanim** rankom.

| Izraz | Šta vraća |
|---|---|
| `Amaze::RankXP()` | XP unutar trenutnog ranka, sa XP-om ovog levela (vrednost trake) |
| `Amaze::RankGoal()` | cilj tog ranka (maksimum trake) |
| `Amaze::Rank()` | rank, sa XP-om ovog levela (može da poraste i pre vortexa) |
| `Amaze::PendingXP()` | XP skupljen na ovom levelu, još nije upisan |
| `Amaze::SavedRank()`, `Amaze::SavedRankXP()` | isto, ali samo upisano (bez XP-a ovog levela) |
| `Amaze::TotalXP()` | sav upisani XP |

Resetovanje napretka tokom testiranja: ponovo učitaj projekat (preview).

## Panel statistika (`NewStatsText`, `RankText`)

Dva BBText objekta na `UI` sloju: `RankText` (Level, Rank, XP) i `NewStatsText` (statovi). Oba se **grade u JS-u kao jedan tekst** (`G.panel()` i ključ `rankPanel` u `G.stat`) i postavljaju jednom akcijom `SetBBText(..., =, Amaze::Stat("panel"))`, a događaj ima uslov `Amaze::TextChanged("panel")` (odnosno `"rankPanel"`). Uslov je tačan samo kad je tekst drugačiji od poslednjeg prikazanog (keš je u `G.R.textCache`, pa se resetuje sa scenom). Razlog: BBText koji se postavlja svaki frejm se i crta svaki frejm, što je spuštalo fps. Traka `XPBar` se i dalje postavlja svaki frejm (jeftino).

Prikazuje se samo ono što igrač ima, grupisano po boji (zelena, plava, narandžasta, crvena, žuta). Uslovi su u `G.panel`:

| Boja | Uvek | Tek kad igrač ima |
|---|---|---|
| zelena | `health` | `regen` i `regenDelay` (regen > 0), `lifesteal`, `dodge`, `lives` |
| plava | | `shield` (shield > 0 ili uzeta shield kartica), `reflect` |
| narandžasta | `speed` | `jump` (otključan), `phase` (Phase Time uzet), `dash` (otključan) |
| crvena | `melee`, `reach` | posle otključanog pucanja: `shotDamage`, `shotRange`, `shotSpeed`, `shots`, pa `fireRate`, `bounces`, `chain` kad su > 0; `crit`, `exec`, `knock` kad su > 0 |
| žuta | | `orbs`, `orbTime`, `orbDamage` (kad ima orbova), `thorns` |

Lista uzetih kartica (`picked`) je uklonjena. Dash cooldown se prikazuje u celim sekundama da se tekst ne menja 10 puta u sekundi. Da dodaš liniju: dodaj `case` u `G.stat` i `add(...)` red u `G.panel` na pravom mestu u grupi boje.

Brojevi štete uključuju `TotalDamageMultiplier`. Stari objekat `StatsText` (običan Text) više nije povezan i može da se obriše.

## Nove kartice: napomene

- **Hitac se više ne pravi preko `FireBullet3D`** (taj ignoriše količinu metaka i varijansu ugla). `G.fire` pravi `PlayerProjectile` sam, sa istim promenljivama (`damage`, `bouncesLeft`, `chainLeft`, `lifeLeft`).
- **Neranjivost** (dash, oživljavanje): `G.stepPlayer` svaki frejm obnavlja Health-ov damage cooldown, a `DamageCooldown` je tada 0.05 s; posle se vraća na `0.5 + Phase Time`.
- **Dash** postavlja `setCurrentForwardSpeed` / `setCurrentSidewaysSpeed` na `PhysicsCharacter3D` i podiže maksimum na 1400 dok traje.

## Zamke

- `getCenterX/Y/Z()` u JS-u su pomeraji od ugla objekta, ne pozicije u sceni. Koristi `getCenterXInScene()`, `getCenterYInScene()` i `getZ() + getDepth()/2` (u kodu `G.cx`, `G.cy`, `G.cz`).
- Vrednosti iz Health ponašanja kroz JS mogu biti stringovi: obavezno `Number(...)`, inače `+15` spaja tekst.
- Pointer lock se otključava u događaju chesta, a ponovo zaključava događajem `CardJustPicked`. Događaj prvog klika ne radi dok su kartice otvorene.

## Nije proveravano u igri

Sve gore je napisano bez preview-a. Za nove kartice posebno: `setCurrentForwardSpeed` (dash), `p.setAnimationSpeedScale` (Rapid Fire), da `G.rescue` iz `Health::Hit` zaista oživi igrača pre `Death` animacije, da hitac krene sa pravog mesta (pozicija se računa preko centra), guranje ne probija zid, i da `BigExplosion` na kritu ne ostaje u sceni. Takođe treba pogledati: izgled i tekst kartica, `MeleeMoment`, veličinu shield sfere, vatru orbitala (ostaje li iza njih), i da se miš dobro zaključava posle izbora.

## Vizuelni feedback (lebdeći tekst i plusići)

Sve u biblioteci `gdjs.amazeUpg` (verzija 3), modul `// ---- visual feedback`. Tekst je `THREE.Sprite` (billboard) sa canvas teksturom u three sceni, diže se po Z, njiše se i bledi; igračev tekst prati igrača, tekst na neprijatelju ostaje gde je pogođen. Više tekstova na istom objektu se slažu jedan iznad drugog. Najviše 60 sprajtova istovremeno. Efekti se kreću sa vremenom scene, pa stoje dok su kartice otvorene.

| Događaj | Tekst / efekat | Odakle |
|---|---|---|
| XP od pogotka | sivo `+N XP` nad igračem (pogoci unutar 0.3 s se sabiraju) | `G.feedbackStep` (`R.xpFx` puni `trackDamage` i `awardKills`) |
| Krit | crveno `CRITICAL HIT!` nad neprijateljem (max jedan na 0.15 s po neprijatelju) | `G.strike` → `G.critFx` |
| Dodge | narandžasto `DODGE!` nad igračem | kraj `Health::Hit` → `G.afterHit` (čita `IsJustDodged`) |
| Bandage | `+N HP` i zeleni plusići oko igrača 2 s (12 u sekundi) | `G.apply` (`R.healBurst`) |
| Bilo koje lečenje (regen, Soul Catcher) | zeleni plusići (4 u sekundi) dok zdravlje raste | `G.feedbackStep` (`R.healGlow`, drži 0.5 s) |
| Šteta igraču | crveno `-N` | pad `U.hp` između frejmova |
| Štit upije udarac / pukne | plavo `-N` / `SHIELD BROKEN` | pad `U.shield` |
| Reflect | plavo `REFLECT!` | `G.reflect` |
| Second Chance | zlatno `SECOND CHANCE!` + plusići | `G.rescue` |
| Crown of Thorns | žuto `THORNS` nad neprijateljem | `G.enemyMeleeHit` |
| Executioner (ubije) | tamnocrveno `EXECUTE!` | `G.strike` |
| Dash | narandžasto `DASH` | `G.dash` |
| Rank up | ljubičasto `RANK UP!` + `Rank N` | `G.feedbackStep` (`R.lastRank`) |
| Izabrana kartica | `+ Naziv` u boji kategorije | `G.apply` |
| Otvaranje chesta | zlatne zvezdice oko chesta | `G.openChest` / `G.stepCards` |

Veličine, boje i trajanja su argumenti poziva `G.say(objekat, tekst, boja, { size, life, pop })`. Nije urađen tekst za upis XP-a na vortexu, jer se scena odmah učitava ponovo.

## Chest: prvo animacija, pa kartice

`Amaze::OpenChest` više ne otvara kartice odmah. Upisuje `R.pending` i čeka `chestOpen + chestHold` (0.75 s je trajanje klipa `Chest_Open` u `Chest.glb`, plus 0.5 s da se vidi otvoren chest; oba u `G.BASE`). Tek tada `G.showCards` pravi UI i usporava vreme na 0.1x. Pointer lock se otključava tek tada: događaj u sceni ima uslov `Amaze::CardsJustShown` (tačan jedan frejm), a ne više akciju u događaju chesta, pa miš i kamera rade dok se chest otvara. Dok traje otvaranje, napadi nisu blokirani (`AreCardsOpen` ostaje samo za otvorene kartice).

**Nije provereno u preview-u:** da li `h.IsJustDodged()` iz JS-a vraća tačnu vrednost (u suprotnom nema `DODGE!`), izgled i veličina tekstova, i da li je 0.75 s + 0.5 s prijatno za chest.

## Pauza i biranje tastaturom

**Kartice:** ←/→ (ili ↑/↓) menja izabranu karticu, Enter potvrđuje, 1–3 i klik rade kao pre. Na početku je izabrana srednja. Miš menja izbor samo dok se pomera. Za kontroler treba samo pozivati `gdjs.amazeUpg.cardNav(±1)` i `gdjs.amazeUpg.cardConfirm()`.

**Pauza** (grupa događaja "Pause menu" u ssceni, sloj `Pause`, kopirano iz maps3d): otvara se kad se izgubi lock miša (Esc) i na početku nivoa ako miš nije zaključan, ne i stalno dok je otključan. Ne otvara se dok su kartice otvorene ili su se tek zatvorile (`Amaze::CardsRecentlyClosed`, 0.6 s). Gore/dole ili miš biraju dugme, Enter ili klik potvrđuje. Esc u pauzi ne radi ništa (prvi Esc samo oslobodi miš i otvori pauzu), pauza se zatvara samo sa CONTINUE. `BACK TO MENU` namerno nema akciju.

**Nije provereno u preview-u:** imena tastera `Return`/`Up`/`Down`, `SceneWindowWidth()` za centriranje, `Resizable` na `BlackBg`, `setColor` iz JS-a na tekstu, i da li Chrome vrati lock odmah posle Esc-a.

**Death screen:** isti sloj i meni kao pauza (grupa "Pause menu"), ali vreme se ne zaustavlja, pa neprijatelji i dalje slave. Pojavi se 1.5 s posle smrti (`DeathT`), otpusti miš i zameni tekstove u `YOU ARE DEAD` / `RETRY` (JS `setText`). RETRY ponovo učitava isti nivo (`Scene "Untitled scene"`, `Level` se ne menja, a `sceneInit` vraća zdravlje), BACK TO MENU nema akciju. Pauza se ne otvara dok je igrač mrtav. Nije provereno: `setText` na Text objektu, da li `Players` grupa tačno javlja smrt.
