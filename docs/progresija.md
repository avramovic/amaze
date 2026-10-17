# Progresija: oprema, oružja, meta nivo i seed sistem

Dopuna za `neprijatelji.md`. Igra: VR roguelite u seed-ovanom 3D lavirintu (GDevelop + WebXR, cilj: Meta Quest).

---

## 1. Oružja za igrača

Princip: **svako oružje ima drugačiju VR radnju** (nišaniš, zamahneš, usmeriš, baciš), a ne samo drugačije brojke.

| Oružje | Šta igrač radi | Kako se računa pogodak |
|---|---|---|
| **Luk** (već postoji u ekstenziji) | zategni i pusti, jedna ruka drži luk, druga strelu | strela je fizički objekat |
| **Magija: munja** | usmeri ruku | raycast + lanac na 2-3 najbliža neprijatelja |
| **Magija: plamen / led** | drži i prskaj | konus: na ~0.1 s proveri ko je u dometu i uglu ispred ruke |
| **Melee: mač / čekić** | zamahni | brzina ruke (razlika pozicije po frejmu), ispod praga brzine nema štete |
| **Bacanje: noževi, bombe** | uhvati i baci | fizički objekat, bomba eksplodira na udar |
| **Štit** | blokiraj | fizički blokira spore projektile (Cactoro, Ghost...) |

### Napomene

- **Particle emitter** je samo vizuelni efekat, čestice nemaju koliziju. Štetu računa konus ili raycast iz ruke.
- **Melee:** bez praga brzine igrač samo drži mač u neprijatelju i dobija štetu. Praviti: šteta zavisi od brzine zamaha, svaki zamah pogađa neprijatelja jednom, vibracija kontrolera pri pogotku. Roj blob neprijatelji su idealni za melee.
- **Lavirint daje oružjima dodatnu vrednost:**
  - rikošet strela i magije od zidova (perk)
  - bačene bombe oko ćoška
  - konus plamena u uskom hodniku pogađa više neprijatelja
  - korisne postojeće GDevelop ekstenzije: Boomerang, Homing projectile, Explosion force, Bounce
- **Hibrid ruke / automatika** (Megabonk elementi): ruke drže **aktivna** oružja, a **pasivni** predmeti rade automatski (orbitirajući noževi, pomoćnik koji puca, aura). Pasivnih držati do 3-4 radi performansi.

### Prototip: tri oružja

Luk (već postoji), jedna magija (munja, najlakša: raycast + lanac) i mač. To pokriva nišanjenje, usmeravanje i zamah.

---

## 2. Zone za opremu (bodyzones / holsters)

Svaka zona zakači **samo jedan** objekat. Postoji i **holster behavior** koji može da zakači više objekata.

| Zona | Predlog | Zašto |
|---|---|---|
| Rame (×2) | luk, štit, dvoručno oružje | velika oružja se prirodno nose na leđima |
| Kuk (×2) | mač, štap (magija), bodež | brzo izvlačenje u borbi |
| Glava | kaciga / šešir (pasivni relikt) | perk koji se vidi i oseća |
| Grudi | amajlija (pasivni relikt) ili aktivna sposobnost | aktivacija sa cooldown-om |
| Usta | napitak, gljiva, hrana | prineseš ustima i popiješ |

**Usta** nije skladište nego tačka upotrebe: predmet se prinese ustima i potroši (lečenje). Možda ne treba da bude zona, nego samo detekcija "predmet blizu glave".

### Holster behavior (više predmeta): inventar i resursi

- **Tobolac:** strele kao fizički objekti koji se troše (strele kao drop daju smisao pretrage lavirinta)
- **Pojas sa napicima:** 3-4 napitka, svaki se nosi do usta
- **Torbica za bombe i noževe** (bacanje)
- **Alka sa ključevima** (ako se uvedu ključevi): vizuelni podsetnik koliko ih još fali

### Prazna ruka = magija

Ako ruka ništa ne drži, ispaljuje magiju (munja, plamen) uz cooldown. Kombinacije bez ikakvog menija:

- mač + magija: borac-čarobnjak
- štit + magija: odbrana uz napad sa distance
- mač + štit: klasika
- luk: obe ruke, najjača distanca, najranjiviji
- dvoručni čekić: obe ruke, spor, ogroman damage

Kad igrač uzme oružje iz zone, magija u toj ruci se gasi.

### Zone kao progresija i perkovi

- **Zone su mesta za perkove:** kaciga i amajlija su pasivni perkovi koje fizički nosiš. Jedan predmet po zoni znači pravi izbor (kaciga sa +oklopom ili +brzinom?).
- **Broj zona kao progresija:** start sa 2-3 zone (npr. kuk + rame + glava), ostale se otključavaju kroz meta nivo ili usput.
- **Brzina vađenja** kao perk (quick draw).
- **Dropovi su predmeti:** neprijatelj ispusti kacigu, igrač je uhvati i stavi na glavu. Ako je zona zauzeta, stari predmet pada na pod (i nestaje posle par sekundi).

### Tehničke napomene

- **Dok je oružje u ruci,** luk zauzima obe ruke (luk u jednoj, strela u drugoj), pa ne može istovremeno sa mačem ili štitom. Zato je izbor oružja fizička radnja.
- **Praćenje kontrolera:** Quest ponekad izgubi kontroler kad je ruka daleko iza leđa. Zone držati blizu tela (kuk, rame, ispred grudi); iza ramena samo tobolac.
- **Slučajni grab:** zona prihvata oružje samo dok je dugme za hvatanje pritisnuto, plus kratka vibracija kad ruka uđe u zonu.
- **Levoruki:** zone ogledati u podešavanjima.
- **Predmeti u zonama** neka ne učestvuju u fizici dok su zakačeni (kinematic).
- **Predmeti na podu** troše fiziku: ograničiti broj i uništavati stare.
- **Prenos između nivoa:** svaki nivo je nova scena. Za svaku zonu sačuvati u globalnoj varijabli koji je predmet u njoj (ID tipa) i pri startu nivoa ih ponovo napraviti.

---

## 3. Run i meta progresija

Odluka: **run se resetuje (oprema nestaje), a nivo igrača (meta) ostaje**, pa je sledeći run jači. Jednostavnije je i dovoljno za roguelite.

| Resetuje se (run) | Ostaje (meta) |
|---|---|
| oprema u zonama | meta nivo igrača |
| HP, štit, municija | otključane zone, oružja, perkovi u pool-u dropova |
| XP i level unutar runa | najbolji nivo / rekordi |
| trenutni nivo lavirinta | podešavanja |

- **Run level** (Megabonk stil, level-up sa izborom 1 od 3) počinje od 1 u svakom runu.
- **Meta nivo** raste od XP-a skupljenog kroz sve runove.

### Kako da meta nivo ne pojede izazov

1. **Mali, ograničeni bonusi** (npr. +2% štete i +3% HP po meta nivou) sa cap-om ili opadajućim prinosom.
2. **Bolje: meta nivo otključava opcije, a ne snagu:**
   - nova zona za opremu (od 2-3 do 7)
   - novo startno oružje ili amajlija
   - novi perkovi i oružja u pool-u dropova
   - novi neprijatelji i biomi

Mogu i oba: mali procenat snage plus otključavanja.

### Čuvanje

GDevelop ima ugrađenu **Storage** funkciju (core features). Meta nivo, XP i otključavanja čuvati u par ključeva ili jednoj JSON varijabli. **Čuvati na kraju svakog runa** (smrt ili pobeda), ne samo na izlazu iz igre, jer na Quest-u igra često ne dobije priliku da se lepo ugasi.

---

## 4. Seed sistem: daily, weekly, više bioma i veličina

Dnevni seed ne mora da bude jedan lavirint nego **jedan master seed iz kog se izvode svi nivoi**.

```
masterSeed   = daily ? hash(UTC datum) : nasumičan / uneti
levelSeed(n) = hash(masterSeed, "level", n)
spawnSeed(n) = hash(masterSeed, "spawn", n)
lootSeed(n)  = hash(masterSeed, "loot", n)
perkSeed(k)  = hash(masterSeed, "perk", k)   // k = redni broj level-up-a
```

| Režim | masterSeed |
|---|---|
| Normalan run | nasumičan (prikazati ga da se može deliti) |
| Daily | hash od UTC datuma |
| Weekly | hash od UTC broja nedelje |
| Vežbanje | seed koji igrač unese |

**Koristiti UTC datum**, ne lokalni, da igrači u različitim zonama dobiju iste nivoe.

### Veličina i biom su funkcije indeksa nivoa

```
config(n):
  biom     = redosled_bioma[(n-1) div 5]      // redosled meša masterSeed
  veličina = f(n)                             // rast po formuli, uz mali ±jitter iz seed-a
  boss     = (n mod 5 == 0)
  budžet   = 12 + 4*n
```

Dnevni seed meša redosled bioma i bira dnevni modifikator, a veličina i težina rastu predvidivo. Pošto se nivoi računaju iz `n`, daily je beskonačan: nivo `n` se pravi tek kad igrač do njega stigne.

### Gde se determinizam lako pokvari

1. **Zasebni tokovi nasumičnosti** za svaku svrhu (level, spawn, loot, perk). Jedan PRNG za sve znači da ranije ubijen neprijatelj pomera sve naredne vrednosti.
2. **Ne vezivati nasumičnost za vreme ili ponašanje igrača.** Spawn red i dropovi se izračunaju unapred pri startu nivoa (vidi `neprijatelji.md`).
3. **Level-up ponude** iz `perkSeed(k)`: isti level-up svima daje iste tri opcije. Kasnije se putevi razilaze jer igrači biraju različito, što je željeno.
4. **Ne koristiti `random()` iz GDevelopa** za ono što utiče na ishod nivoa (ne može da se seeduje). Koristiti isključivo svoj seed-ovani generator.

### Rang lista

Rezultat (nivo do kog si stigao, ili vreme) je uporediv jer svi igraju iste nivoe. **Meta bonusi se isključuju u daily režimu**, da bi rang lista bila čista veština. Alternativa: daily sa meta bonusima na zasebnoj listi.

---

## 5. Otvorena pitanja

- Da li igrač od starta ima fiksan broj zona, ili počinje samo sa lukom i ostalo nalazi usput? (Menja izgled prvog nivoa.)
- Koja tačno oružja idu u prvu verziju (predlog: luk, munja, mač)?
- Veličina lavirinta po nivou: formula i cap (npr. linearan rast dimenzije do ~25×25, dalje težina preko neprijatelja).
- Koliko meta nivoa ima i šta svaki otključava?
