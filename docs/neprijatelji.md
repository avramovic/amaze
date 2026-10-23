# Neprijatelji: dizajn i spawn sistem

Asset: Quaternius "3D colorful monsters" (low-poly, CC0, proveriti licencu na stranici paketa pre objave).
Igra: VR roguelite u seed-ovanom 3D lavirintu (GDevelop + WebXR, cilj: Meta Quest).

---

## 1. Grupe modela i animacije

Autor deli modele u tri grupe. Modeli iste grupe dele iste animacije, a animacije se u GDevelopu mogu preimenovati, pa se može napisati **jedan generički event sistem** za sve neprijatelje (npr. `Idle`, `Walk`, `Attack`, `Hit`, `Death`).

| Grupa | Broj animacija | Uloga u igri |
|---|---|---|
| **Blob** | 8 | rojevi, jeftini, najviše ih može biti odjednom |
| **Standing** | 14 | glavni neprijatelji, više animacija = više ponašanja (napad, udarac, skok, dash...) |
| **Flying** | 8 | ignorišu zidove lavirinta, ređi, dolaze pravolinijski |

### Modeli po grupama

- **Blob:** Alien, Cat, Cactoro, Birb, Dog, GreenBlob, Chicken, Mushnub, Fish, Mushnub Evolved, GreenSpikyBlob, PinkBlob, Ninja, Pigeon, Orc, Wizard, Yeti
- **Standing:** BlueDemon, Birb, Alien, Dino, Demon, Frog, Cactoro, Ninja, MushroomKing, Orc Skull, Monkroose, Tribal, Yeti, Fish, Orc, Bunny
- **Flying:** Alpaking (+Evolved), Armabee (+Evolved), Dragon (+Evolved), Glub (+Evolved), Goleling (+Evolved), Pigeon, Ghost, Ghost Skull, Hywirl, Tribal, Squidle, Demon

### Porodice (isti lik u više oblika)

Neki likovi postoje i kao Blob i kao Standing (ili Flying) verzija:

- **Orc:** Orc (Blob) → Orc (Standing) → Orc Skull (elite)
- **Mushnub:** Mushnub → Mushnub Evolved → MushroomKing (mini-boss)
- **Blob + Standing parovi:** Ninja, Yeti, Fish, Cactoro, Birb, Alien
- **Standing + Flying:** Demon, Tribal, Dragon
- **Flying + Evolved:** Alpaking, Armabee, Dragon, Glub, Goleling

---

## 2. Mehanika "evolucija"

1. Nivo počinje blob verzijom porodice (roj).
2. Tajmer kreće kad neprijatelj **prvi put primeti igrača** (aggro) i teče dalje i ako igrač pobegne. Ako ga igrač ne ubije u roku od X sekundi (npr. 25 s), on **evoluira** u standing verziju (zameni se objekat, zadrži se procenat HP-a). Sistem tako gura igrača da bude agresivan.
3. Blob koji može da evoluira **košta punu cenu Standing verzije** (npr. Orc blob = 3, ne 1), iako se pojavi kao blob. Budžet se rezerviše pri generisanju reda, pa evolucija ne donosi težinu preko budžeta, a "kazna" za sporo igranje je prava pretnja. Blob tipovi koji ne evoluiraju (Cat, Dog, Chicken, GreenBlob) koštaju 1.
4. Leteći *Evolved* modeli su elite varijante (brži, jači, ili sa modifikatorom).
5. "Kraljevi porodica" (MushroomKing, Orc Skull...) su mini-bossevi ili elite sa blob slugama. Glavni bosevi (astronaut → mech) su u poglavlju 8.

---

## 3. Uloge

- **Roj:** mali, brzi, slabi. Glave bez tela (Cat, Dog, Chicken) i kuglice.
- **Ganjač:** prati igrača preko flow-field mape, melee napad.
- **Skakač / brzi:** Frog (skok + jezik), Bunny, Ninja (dash, pa pauza).
- **Strelac:** drži distancu, puca niz hodnik (Cactoro, Ghost, Squidle; Wizard je poseban slučaj, vidi poglavlje 7). Projektili treba da su **spori**, da igrač može da ih izbegne naginjanjem u VR-u. Nijedna grupa nema animaciju pucanja, pa se projektil ispaljuje usred postojeće napadačke animacije (vidi poglavlje 7).
- **Stacionarni:** Mushnub (oblak spora, usporava), kao "turret" u slepim ulicama.
- **Leteći:** ne koriste flow-field, lete pravo ka igraču i ignorišu zidove. Držati ih malo.
- **Tenk / elite:** spori, mnogo HP (Yeti, Orc Skull, BlueDemon, Dino).
- **Boss:** ručno postavljen, svaki 5. nivo, dve faze (astronaut → mech), vidi poglavlje 8.
- **Varijante:** isti model u drugoj boji = viši tier (običan, brži, jači, elite sa modifikatorom).

---

## 4. Tabela neprijatelja

HP je za nivo 1. Brzina je relativna u odnosu na brzinu hodanja igrača. Osnovno oružje računato kao 10 štete po metku. **Sve vrednosti su početne i služe za tuning.**

| Uloga | Model (grupa) | HP | Brzina | Napad | Cena | Od nivoa |
|---|---|---|---|---|---|---|
| Roj | Cat, Dog, Chicken (Blob) | 10 | 0.9 | dodir 5 | 1 | 1 |
| Roj, spor | GreenBlob (Blob) | 15 | 0.6 | dodir 5 | 1 | 1 |
| Roj → evolucija | Orc, Ninja, Fish, Yeti... (Blob → Standing) | 12 | 0.8 | dodir 5 | cena Standing verzije (Orc 3, Yeti 8...) | 2 (Yeti blob tek od 5, kao Standing) |
| Stacionarni | Mushnub (Blob) | 25 | 0 | oblak spora | 2 | 2 |
| Leteći | Armabee, Goleling, Glub (Flying) | 20 | 0.8 | dodir 6 | 2 | 2 |
| Ganjač | Orc (Standing) | 40 | 0.7 | melee 10 | 3 | 2 |
| Skakač | Frog (Standing) | 30 | skok 1.2 | 10 | 3 | 3 |
| Strelac | Cactoro (Standing) | 30 | 0.4 | projektil 8 | 4 | 3 |
| Brzi | Bunny, Ninja (Standing) | 25 | 1.2 | 8 | 3 | 4 |
| Leteći strelac | Ghost, Squidle (Flying) | 25 | 0.5 | projektil 8 | 4 | 5 |
| Tenk | Yeti (Standing) | 150 | 0.4 | 20 | 8 | 5 |
| Elite leteći | svi *Evolved (Flying) | 50 | 1.0 | 12 | 5 | 6 |
| Elite zemaljski | Orc Skull, BlueDemon, Dino | 100 | 0.8 | 15 | 6 | 6 |
| Mini-boss (opciono) | MushroomKing, Demon, Tribal, Dragon Evolved | 300 | 0.6 | 20 | 12 | 8 |
| Boss | Žaba, Lisica, Flamingo, Pčela (redosled određuje biom/seed): astronaut → mech, vidi poglavlje 8 | ~600 ukupno | n/a | ručno | n/a (ručno) | svaki 5. |

### Skaliranje po nivou

- HP: `HP × (1 + 0.12 × (nivo − 1))`
- Šteta: `šteta × (1 + 0.08 × (nivo − 1))`

### Biomi po porodicama

Biom traje 5 nivoa i završava se bossom tog bioma (poglavlje 8). Umesto mešanja svega, svaki biom ima 2-3 porodice plus sluge svog bossa (npr. podrazumevano, bez daily seeda: nivoi 1-5: Orc + Mushnub + Pigeon + sluge Žabe; nivoi 6-10: Ninja + Yeti + Ghost + sluge Lisice). Igrač uči neprijatelje u grupama, a težina je pod kontrolom.

**Redosled bioma (pa i bossova) određuje seed.** Podrazumevani redosled je Žaba, Lisica, Flamingo, Pčela, a daily seed ga meša. Porodice i sluge su vezane za biom, ne za broj nivoa.

**Kako se biom i kolona "Od nivoa" kombinuju:** pool za spawn je presek. Tip ulazi u pool samo ako (1) njegova porodica je u listi aktivnog bioma (ili je sluga njegovog bossa) i (2) `minLevel <= nivo`. "Od nivoa" je dakle globalna donja granica težine, a biom bira šta se uopšte pojavljuje. Pri podeli porodica po biomima paziti da svaki biom ima bar roj, ganjača i jednog daljinca, inače pool ostane bez uloge.

---

## 5. Cena i budžet

**Budžet** je "novčanik" nivoa, a **cena** je koliko je neprijatelj težak. Pri pravljenju nivoa sistem "kupuje" neprijatelje dok ne potroši budžet. Jači neprijatelj košta više, pa ga ima manje.

- Budžet nivoa: `12 + 4 × nivo`
- Cena: iz tabele iznad (Cat = 1, Orc = 3, Yeti = 8...)

### Primer: nivo 3 (budžet 24)

| Izvučeno | Cena | Preostalo |
|---|---|---|
| Orc | 3 | 21 |
| Cat, Cat, Cat | 1+1+1 | 18 |
| Cactoro | 4 | 14 |
| Armabee, Armabee | 2+2 | 10 |
| Frog | 3 | 7 |
| Orc | 3 | 4 |
| Cat ×4 | 4 | 0 |

Na nivou 10 budžet je 52, pa isti sistem sam napravi više i teže neprijatelje.

### Boss nivoi

Boss **nema cenu**, jer se ne bira nasumično. Svaki 5. nivo je boss nivo:
- boss se postavlja ručno i vezan je za biom (poglavlje 4 i 8). Podrazumevani redosled je Žaba na 5, Lisica na 10, Flamingo na 15, Pčela na 20, pa ciklus ponovo, a daily seed meša redosled bioma, pa i bossova
- budžet se i dalje troši, ali samo ~30% običnog budžeta, na manje neprijatelje (sluge)

### Prednosti

- težina raste glatko i sama
- novi neprijatelj se doda u tabelu i automatski se uklopi
- seed daje isti raspored svima (daily izazov)

### Za prvi prototip

Budžet nije obavezan. Dovoljno je ručno: "nivo N ima N×3 Orc-ova i N×2 Cat-ova". Budžet postaje koristan tek kad ima 6-7 različitih tipova.

---

## 6. Spawn sistem

**Ključna ideja: red neprijatelja se napuni pri startu nivoa**, a ne tokom igre. Tako daily seed daje svima iste neprijatelje bez obzira na to kako ko igra.

```
// pri generisanju nivoa (isti PRNG kao za lavirint, ali drugi stream: seed + 777)
budget = 12 + 4 * nivo
pool   = tipovi sa minLevel <= nivo
dok je budget >= najjeftinija cena:
    t = težinski_izbor(pool gde cena <= budget)
    queue.push({tip: t, ćelija: izaberi_ćeliju()})
    budget -= t.cena
(roj tipovi se guraju u grupama od 3-5)
```

**Ćelije za spawn** se izračunaju jednom: bez početnog radijusa oko igrača, po mogućnosti slepe ulice i veći prostori. U runtime-u se izbace ćelije koje igrač trenutno vidi ili su preblizu. "Vidi" znači ćelije koje je dosegao BFS kuling lavirinta (uslov `Amaze::IsMazeCellVisible`, poglavlje 9).

**Runtime:**
- Svake ~2 s: ako `alive < cap` i red nije prazan, pogledaj prvih nekoliko elemenata reda i napravi prvi čija je ćelija u **opsegu aktivacije** igrača (npr. 8-20 ćelija putanje) i koju igrač ne vidi. Red i ćelije ostaju isti za sve igrače, menja se samo trenutak spawna. Ako nijedan nije u opsegu duže od ~10 s, uzmi najbliži dozvoljeni, da nivo ne stane.
- `cap` oko 10-12 za Quest (leteći se računaju kao 1).
- Elite šansa: `5% + 1% × nivo`, zamenjuje običan tip iste uloge (tvrđi tint + veća skala).
- Boss nivo: boss + ~30% budžeta za sluge.

### Organizacija u GDevelopu

- Svaki tip je poseban 3D Model objekat.
- Svi idu u **Object Group "Enemies"** sa istim ponašanjima i istim object variables (`Speed`, `Damage`, `XP`, `Role`), pa isti događaji važe za sve.
- Tabela (cena, minLevel, težina) ide u jednu structure varijablu, samo za pravljenje reda.

---

## 7. Animacije

Sve animacije su iste unutar grupe. `no` i `yes` su samo odmahivanje i klimanje glavom.

| Grupa | Animacije |
|---|---|
| **Blob** (8) | idle, walk, jump, death, bite, dance, no, yes |
| **Standing** (14) | idle, walk, run, duck, jump, punch, death, jumpidle, jumpland, hitreact, wave, weapon, yes, no |
| **Flying** (8) | idle, punch, death, fastflying, hitreact, headbutt, no, yes |

### Mapiranje na stanja

| Stanje | Blob | Standing | Flying |
|---|---|---|---|
| Mirovanje | idle | idle | idle |
| Kretanje | walk | walk (spori tipovi) / run (jurnjava) | fastflying |
| Napad (melee) | bite | punch ili weapon | punch |
| Poseban napad | jump (skok na igrača) | jump → jumpidle → jumpland | headbutt (nalet) |
| Pogođen | nema | hitreact | hitreact |
| Smrt | death | death | death |
| Zaklon | nema | duck | nema |
| Emote | dance, yes, no | wave, yes, no | yes, no |

Rupe: Blob nema `hitreact`, Flying nema `walk` ni `jump` (ne trebaju mu).

### Standing napadi: punch i weapon

- `punch` je napad slobodnom rukom.
- `weapon` je napad rukom u kojoj model drži oružje. Ako model nema oružje, to je samo druga ruka.
- **Oba su melee.** Nijedna grupa nema pravu animaciju pucanja ili bacanja.

Predlog: dodati object varijablu `HasWeapon` (ili `AttackAnim`) po tipu. Modeli sa oružjem koriste `weapon` kao glavni napad (duži domet, sporiji, jači), a ostali `punch`. Može i kombo: `punch` pa `weapon`, ili izbor po udaljenosti igrača. Proveriti u GDevelopu koji modeli stvarno drže oružje (sa slike izgleda da buzdovan ima Orc, katanu Ninja, a palicu jedan plavi lik).

### Neprijatelji na daljinu bez animacije pucanja

Projektil se pravi **usred napadačke animacije** (npr. u trenutku kad ruka ide napred), a ne na njenom početku. Igrač vidi zamah i projektil izlazi iz ruke, što izgleda kao bacanje.

- **Cactoro (Standing):** `punch` kao bacanje bodlji.
- **Ghost, Squidle (Flying):** `punch` kao ispaljivanje čarolije.
- **Wizard (Blob):** blob ima samo `bite`, pa kao kasting može da posluži `dance`. Čarobnjak koji "pleše" i ispaljuje projektil je smešan i jasno čitljiv.
- **Mushnub (Blob, stacionarni):** spore se ispuštaju na `dance` (ili `bite`).

### Pravila ponašanja

1. **Animacije se ne preimenuju.** Behavior `Enemy` (poglavlje 10) sam bira animaciju po stanju i property-ju `BodyType` (Blob, Standing, Flying). Standing ima `Walk` i `Run`, pa property `MoveAnim` bira koji se koristi za kretanje (spori tipovi kao Yeti i Cactoro koriste `Walk`), a `AttackAnim` bira `Punch` ili `Weapon`. GDevelop razlikuje velika i mala slova, a imena u behavior-u su ista kao u modelima (`Idle`, `Walk`, `Bite`, `FastFlying`, `HitReact`...).
2. **Prioritet stanja:** `Death > Hit > Attack > Move > Idle`.
3. **Blob nema `hitreact`**, pa ga lažiraj: bljesak ili tresenje objekta pri pogotku (ekstenzije "Flash object" i "3D object shake") plus kratak squash skale.
4. **Bez stunlock-a:** ako svaki metak pokreće `hitreact`, neprijatelj pod brzom paljbom nikad ne napada. Hit animacija samo ako nije usred napada, najviše jednom u ~0.5 s, ili samo kad pogodak skine više od ~15% HP-a. Tenkovi, elite i bossevi je ignorišu (super armor).
5. **Smrt:** pri smrtonosnom pogotku odmah isključi koliziju i AI, ispusti XP, pusti `Death`, a objekat uništi tek kad animacija prođe (tajmer sa poznatom dužinom).
6. **Telegrafisanje napada** je u VR-u jako važno, igrač mora da vidi šta dolazi:
   - Flying `headbutt`: kratko zastajanje (idle), pa nalet.
   - Frog: `jump` → `jumpidle` (igrač vidi gde sleće) → `jumpland` (kratka ranjivost).
   - Napadi na daljinu: projektil tek na sredini animacije, nikad na početku.
7. **Emote animacije** daju karakter skoro besplatno:
   - **Evolucija:** blob igra `dance` ~1 s pre evolucije (upozorenje igraču da ga ubije u tom prozoru), a Standing verzija posle toga odigra `wave`.
   - **Boss intro:** `wave` ili `yes` pri pojavljivanju.
   - **Igrač umre:** preživeli neprijatelji odigraju `yes` (slavlje), što poraz čini manje frustrirajućim.
   - `no` može da se odigra kad igrač izbegne projektil.

---

## 8. Bosevi: astronaut → mech

Četiri boss-a, svaki u dve varijante (astronaut i mech), plus svemirski brod bez animacija. Svaki boss je vezan za jedan biom. Likovi: Žaba, Lisica, Pčela i Flamingo (za Flaminga zamena: Chicken iz Quaternius seta ako se oboji u roze, ili Pink Dino za astronaut fazu).

### Animacije

| Model | Animacije |
|---|---|
| **Mech** (17) | dance, death, hello, hitreceive1, hitreceive2, idle, jump, jump_landing, Jump_NoHeight, Kick, no, yes, pickup, run, Shoot_Small, Shoot_Big, walk |
| **Astronaut** (18) | death, duck, HitReact, idle, idle_gun, jump, jump_idle, jump_land, no, yes, punch, run, run_gun, run_gun_shoot, walk, walk_gun, wave, Weapon |
| **Brod** | bez animacija (kreće se putanjom ili tween-om) |

Napomena: imena su ujednačena u samom GDevelopu (npr. `Shoot_Big`, `Shoot_Small`, `Kick`, `Weapon`, `HitReact`, `Jump_NoHeight`). Imena u tabelama ispod su pisana približno, pa u eventima koristiti tačna imena iz GDevelopa, jer razlikuje velika i mala slova. Ako se želi jedan zajednički skup događaja za oba modela, preimenovati u zajedničke nazive (npr. `Idle`, `Move`, `Hit`, `Death`).

Za razliku od običnih neprijatelja, bossevi **imaju prave animacije pucanja** (`Shoot_Small`, `Shoot_Big`, `Weapon`, `run_gun_shoot`), pa projektil ne mora da se lažira iz `punch`. Pažnja: `Weapon` kod astronauta je pucanje iz pištolja, a kod običnih Standing monstruma je melee.

### Dve faze, jedna borba

Ukupni HP: osnova ~600 (astronaut 40%, mech 60% + štit), uz isto skaliranje po nivou kao za ostale (`× (1 + 0.12 × (nivo − 1))`).

1. **Intro:** brod preleti arenu i ispusti astronauta. Astronaut `jump_idle` → `jump_land`, pa `wave`.
2. **Faza 1, astronaut** (osnova HP 240).
3. **Prelaz** (~4 s, boss neranjiv): astronaut odigra `no` i pozove brod, brod spusti mech, mech `Jump_NoHeight` (visinu pada daje kod) → `jump_landing`, zatim `pickup` (mech pokupi astronauta u kokpit; proveriti kako animacija izgleda) i `hello`. Astronaut objekat se uništi.
4. **Faza 2, mech** (osnova HP 360 + štit ~120): na početku *Activate shield* sa trajanjem 0 (trajni štit, po dokumentaciji Health ekstenzije), pa igrač prvo mora da probije štit.
5. **Smrt:** mech `death`, brod sleti i postaje **izlaz iz nivoa** (vodi u sledeći biom).

**Neranjivost u prelazu** preko Health ekstenzije: postaviti *Damage cooldown* na ~4 s pa *Trigger damage cooldown* (dok traje, objekat ne prima štetu), a posle prelaza vratiti vrednost na 0.

Dve odvojene Health "trake" (astronaut pa mech) su jednostavnije od jedne zajedničke. Boss bar u UI-u može da bude jedna traka podeljena na 40/60.

### Astronaut: napadi

| Potez | Animacija | Napomena |
|---|---|---|
| Kretanje | `walk_gun`, `run_gun` | drži distancu, kruži oko igrača |
| Pucanje u trku | `run_gun_shoot` | brzi projektili, mala šteta |
| Melee | `punch` | samo kad igrač priđe |
| Pucanje u mestu | `Weapon` | pucanje iz pištolja koji drži; `idle_gun`, `walk_gun` i `run_gun` su poze sa izvučenim pištoljem; projektil se pravi iz mesta pištolja (offset) |
| Izbegavanje | `duck` | povremeno, sa cooldown-om (nagrada za timing) |
| Pogođen | `HitReact` | super armor, samo na pragovima HP-a |
| Preskok | `jump` → `jump_idle` → `jump_land` | prelazi preko igrača ili zatvara razdaljinu |

### Mech: napadi

| Potez | Animacija | Napomena |
|---|---|---|
| Intro | `hello` | |
| Rafal | `Shoot_Small` | serija sporih projektila (Fire bullets: *Bullets per shot* + *Firing arc*) |
| Teški napad | `Shoot_Big` | dugo telegrafisanje (1-2 s nišanjenja, linija ili krug), jedan veliki spor projektil; igrač izbegava naginjanjem ili blokira štitom |
| Melee | `Kick` | kratak domet, kad igrač priđe |
| Skok na igrača | `Jump_NoHeight` → `jump_landing` | luk leta (visinu i pomeranje) računa kod; krug na podu pokazuje zonu udara, šteta u radijusu pri sletanju |
| Stomp u mestu | `jump` → `jump_landing` | visinu daje sama animacija (mesh ide u vis), kod ne pomera model; udarni talas oko mecha |
| Juriš | `run` | prava linija kroz arenu |
| Pogođen | `hitreceive1` / `hitreceive2` | nasumično biraj, samo na pragovima HP-a (super armor) |
| Taunt | `dance` | kad igrač umre, ili kao poziv sluga |
| Reakcija | `yes` / `no` | `no` kad igrač izbegne `Shoot_Big`, `yes` kad ga pogodi |

### Skokovi: Jump i Jump_NoHeight

- **`Jump`** pomera mesh u vis (visinu daje animacija). Koristiti kad se model skače u mestu (stomp). Ako kod istovremeno dodaje visinu, skok se udvostruči.
- **`Jump_NoHeight`** ne pomera mesh u vis, pa visinu i putanju računa kod (fizika, tween ili parabola). Koristiti kad skok ima luk preko prostora: skok na igrača, pad mecha sa broda.
- Astronaut ima samo `jump` / `jump_idle` / `jump_land`. Ako njegov `jump` takođe pomera mesh u vis, kod mu treba da pomera samo horizontalno (ili da ga koristi samo u mestu).

### Po bossu (predlog na osnovu izgleda sa slike)

| Boss | Izgled mecha | Potpis | Sluge |
|---|---|---|---|
| **Žaba** (podrazumevano nivo 5) | zeleni, 4 topa | `Shoot_Small` kao lepeza (~5 metaka, luk ~40°), `Shoot_Big` kao salva 4 topa | Frog, GreenBlob |
| **Lisica** (podrazumevano nivo 10) | narandžasti, lagan, na 4 noge | najbrža: `run` juriš + `Kick`, `Shoot_Small` brzi pojedinačni | Cat, Dog, Bunny |
| **Flamingo** (podrazumevano nivo 15) | roze, sanduk sa malim topovima | `Shoot_Big` kao spori navođeni projektili (ekstenzija Homing projectile) | PinkBlob, Squidle |
| **Pčela** (podrazumevano nivo 20) | žuti, bočni topovi | `jump` stomp sa udarnim talasom, na `dance` priziva sluge | Armabee (+Evolved) |

Šteta (početne vrednosti): astronaut `Weapon` i `run_gun_shoot` 5, `punch` 10. Mech `shoot_small` 6, `shoot_Big` 20, `Kick` 18, sletanje 15.

### Svemirski brodovi

Brodovi nemaju animacije, pa se kreću putanjom (tween, ekstenzije Linear movement, Curved movement, Advanced 3D tween). Upotrebe:

1. **Intro i prelaz** (spuštanje astronauta i mecha), kako je opisano gore.
2. **Izlaz iz nivoa:** posle pobede brod sleti i vodi u sledeći biom (četiri broda = četiri bioma).
3. **Opciono: faza 3 (enrage)**, ispod ~20% HP-a mecha: brod preleti i bombarduje liniju (telegraf: senka ili linija na podu). Jeftino jer nema animacija.
4. **Dekor** (srušen brod u lavirintu).

### Arena

Mech je veliki, a lavirint uzak, pa boss nivo mora da ima **otvorenu arenu** (ručno dizajnirana soba ubačena u generator, ili veći izrezani prostor). Ostaviti nekoliko stubova ili zidova kao zaklon od `shoot_Big`. Boss se u areni kreće direktno prema igraču (bez flow-field mape).

### Biomi i redosled

U paketu postoje planete, kupole, roveri i biljke. Planete se mogu postaviti daleko kao pozadina (jeftino, a u VR-u daje dubinu):

| Boss | Biom (predlog) |
|---|---|
| Žaba | zeleni lavirint (prvi biom) |
| Lisica | narandžasta pustinja |
| Flamingo | ljubičasta planeta |
| Pčela | ledena planeta |

Tabela prikazuje podrazumevani redosled. Dnevni seed meša redosled bioma, a boss ide uz svoj biom, pa se meša i redosled bossova (nivo bossa je i dalje svaki 5.). Zato HP i šteta skaliraju sa **brojem nivoa**, a ne sa identitetom bossa.

### Ciklus

Posle četvrtog bossa (nivo 20) ciklus se ponavlja: bossevi preskaču astronauta i počinju u mechu, ili dobijaju modifikator.

### Tehnički: jedan generički boss sistem

Sva četiri boss-a dele isti skup animacija, pa je dovoljan **jedan state machine** i tabela parametara po bossu (koji napad, *Bullets per shot*, *Firing arc*, cooldown, šteta, brzina). Tokom boss borbe smanjiti `cap` sluga (npr. na 6), jer mech i projektili troše performanse.

---

## 9. Performanse (Quest)

- Max ~10-15 animiranih neprijatelja odjednom.
- Spawn samo u opsegu aktivacije oko igrača (poglavlje 6), ne cela mapa odjednom.
- Udaljenima smanji ili zamrzni animacije.
- Pathfinding: jedna flow-field (BFS) mapa od igrača, osvežena svakih 0.2-0.5 s, svi ganjači prate gradijent.
- Većina modela deli jednu paletu boja, pa pokušati da dele materijal (manje draw call-ova).
- Proveriti broj poligona po modelu u GDevelopu.

### Kuling lavirinta (BFS po ćelijama)

`webxr` ekstenzija radi samo frustum culling (sakriva ono iza igrača i sa strane). Zidovi ništa ne zaklanjaju, pa se sve u vidnom konusu iza zidova i dalje crta. Lavirint je graf, a pogled prolazi samo kroz otvorene prolaze, pa ekstenzija Amaze radi BFS od ćelije igrača kroz otvorene zidove do dubine N i **sakriva sve izvan dosegnutih ćelija** (`object.hide`; skriveni 3D objekat se uopšte ne crta). Fizika i logika skrivenih objekata rade dalje.

- **Dve Object Group:** `MazeStatic` (zidovi, podovi, stubovi) se osvežava samo kad igrač pređe u drugu ćeliju, a `MazeDynamic` (neprijatelji, leševi) svaki frejm.
- **Poziv svaki frejm:** `Amaze::UpdateMazeCulling("maze", MazeStatic, MazeDynamic, glava.X, glava.Y, 6)` (bez uslova na grupe). Posle generisanja novog lavirinta ili izmene zidova pozvati `ResetMazeCulling("maze")`.
- **Ćelija objekta** se računa iz pozicije centra, ne iz taga. Zidovi i stubovi leže na ivicama i uglovima, pa pripadaju do 4 ćelije i vidljivi su ako je bilo koja od njih vidljiva.
- **Magla** istog dometa krije pop-in: Scene3D efekat "Fog (linear)" na sloju, početak oko `(N−2) × širina ćelije`, kraj oko `N × širina ćelije`.
- **Debug:** `MazeCullVisibleCells("maze")`, `MazeCullHiddenObjects("maze")`, `SetMazeCullingEnabled(Ne)` za poređenje.
- `webxr` frustum culling ostaje uključen i radi kao dodatni sloj.

---

## 10. Ekstenzija Amaze: behavior `Enemy`

Fajl: `eventsFunctionsExtensions/amaze.json`. Zahteva ekstenziju "Health points and damage" (HP i šteta primljena preko nje). Behavior se kači na 3D Model objekte, a ceo tuning je u Properties panelu.

**Properties:** `BodyType`, `Role`, `Family`, `Speed`, `Damage`, `XP`, `AttackRange`, `AttackCooldown`, `AttackDuration`, `AttackMomentTime`, `AttackAnim`, `MoveAnim`, `AggroRange`, `EvolvesTo` (ime objekta, prazno = ne evoluira), `EvolveTime`, `SuperArmor`, `HitCooldown`, `DeathTimeout`, `LeaveCorpse`. `Health` je required behavior.

**Smrt i leševi:** kad animacija `Death` završi (uslov `HasAnimationEnded`), ne upisuje se dužina po modelu. U editoru objekta `Death` mora da ima isključen **Loop**, inače nikad ne završi. `DeathTimeout` (5 s) je sigurnosna mreža za tu grešku.
- `LeaveCorpse` uključen (podrazumevano): neprijatelj prelazi u stanje `Dead`. Završena animacija ostaje na poslednjem frejmu (GDevelop je drži), a `Enemy` više ništa ne radi osim brojanja `CorpseTime()`. Leš je trag kojim se igrač seća gde je već bio.
- `LeaveCorpse` isključen: objekat se briše čim animacija završi.
- Slobodna akcija `Amaze::LimitCorpses(Enemies, ..., Max)` jednom po frejmu briše najstariji leš kad ih ima više od `Max` (npr. 15). Uslov `IsCorpse` služi za sve ostalo.

**Stanja** (prioritet `Death > Hit > Attack > Move > Idle`, plus `Evolving`, `Intro`, `Celebrate`) menjaju se u `doStepPostEvents`. Razlog: Health briše svoju zastavicu "just damaged" u `doStepPreEvents`, pa je jedino posle događaja scene još vidljiva.

**Šta event sheet radi:**

| Šta | Kako |
|---|---|
| Meta za kretanje | akcija `SetTarget(x, y)` svaki frejm (igrač, kasnije ćelija flow-field-a) |
| Udarac / projektil | uslov `AttackMoment` (jedan frejm, na `AttackMomentTime` unutar napada); melee proverava domet, ranged pravi projektil |
| Šteta neprijatelju | `Health::Hit` na objektu; behavior sam reaguje (`Hit`, `Death`) |
| XP i dropovi | uslov `JustDied` (jedan frejm) |
| Skaliranje | akcija `ApplyLevelScaling(nivo)` pri spawnu (+12% HP, +8% štete po nivou) |
| Evolucija | slobodna akcija `Amaze::ProcessEvolutions(Enemies, ...)` jednom po frejmu |
| Igrač umre | akcija `Celebrate` |
| Leševi | slobodna akcija `Amaze::LimitCorpses(Enemies, ..., 15)` jednom po frejmu |

**Evolucija:** tajmer kreće pri prvom aggrou. U poslednjoj sekundi neprijatelj stane i igra `Dance` (Blob). `ProcessEvolutions` zatim pravi objekat iz `EvolvesTo` (akcija "Create an object from its name"), prenosi poziciju, ugao, Z, procenat HP-a i aggro, pušta `Wave` (Standing) i briše stari. Svi objekti koji mogu da nastanu moraju da budu u grupi "Enemies".

**Ograničenja prve verzije:**
- Kretanje je pravo ka meti i **ne poštuje zidove**. To je u redu za leteće, a kopnene čeka flow-field.
- `Ranged` ne drži distancu, to se dodaje u event sheet-u ili kasnije u behavior-u.
- Blob nema `HitReact`, pa pri pogotku samo nastavlja trenutnu animaciju. Bljesak ili tresenje ide preko ekstenzija iz pravila 3.
- Cena, `minLevel` i težina ostaju u spawn tabeli (poglavlje 6), jer se properties ne mogu čitati bez instance.

---

## 11. Spawner i navigacija (implementirano)

**Behavior `Amaze::Spawner`** (na objektu `Spawner`, ime behaviora `Spawning`): talasi. Spawner se budi kad je igrač na `ActivationRange` ćelija putanje (podrazumevano 9999 = uvek, pa prvi talas kreće odmah na početku; smanji vrednost da se budi tek kad igrač priđe), napravi `WaveSize` neprijatelja (jedan na `SpawnInterval`), a sledeći talas kreće tek kad su svi iz prethodnog mrtvi plus `WaveCooldown`. Posle `Waves` talasa (0 = dok ima reda) je `Exhausted`. Stanja: `Waiting`, `Spawning`, `Active`, `Cooldown`, `Exhausted`.

**`Amaze::RunSpawners`** (jednom po frejmu, scena): broji žive neprijatelje (po `SpawnerId` i ukupno), pa spawneri koji žele da spawnuju uzimaju sledećeg iz reda (`PeekEnemyQueue`). **Globalni limit** živih je poslednji parametar (12). Novi neprijatelj dobija `ApplyLevelScaling`, `SpawnerId`, Z iz `FlyHeight` i `StartIntro`. `ProcessEvolutions` prenosi `SpawnerId` na evoluiranog.

**Red i tabela:** `SetEnemySpawnInfo(ime, cena, minNivo, težina, grupaMin, grupaMax)` na startu scene, pa `BuildEnemyQueue(Level, Seed)` (budžet `12 + 4 × nivo`, PRNG `seed + 777`). Isti seed daje isti red (ispis u konzoli).

**Navigacija:** `UpdateFlowField` (BFS od ćelije igrača, samo kad igrač promeni ćeliju), `FlowTargetX/Y` (tačka 0.75 ćelije iza vrata ka igraču, na sredini hodnika), `MazeLineOfSight` (1 ako su u istoj ćeliji ili u istom redu/koloni bez zida). Behavior `Enemy` dobija `SetMoveTarget` (kuda hoda) i `SetCanAttack` (napad samo sa linijom vidljivosti), a `SetTarget` ostaje igrač (aggro, napad). `CanCrossWalls` preskače flow-field (za Evolved letače).

**Napad:** `AttackMoment` u sceni: unutar `MeleeRange` je udarac (`Health::Hit` na igraču), a van njega ranged uloga ispaljuje `EnemyProjectile` (300 px/s, 8 štete, nestaje na zidu ili igraču).

---

## 11b. Progresija tipova po nivoima (implementirano)

`SetEnemySpawnInfo` u početnom eventu scene je jedina tabela (cena, minimalni nivo, težina, veličina grupe). Redosled otvaranja:

| Nivoi | Šta se pojavljuje |
|---|---|
| 1-5 | samo blobovi koji ne evoluiraju (Cat, Dog, Chicken, GreenBlob, PinkBlob, GreenSpikyBlob), pa stacionarni Mushnub (4) i Wizard (5) |
| 6-9 | blobovi koji evoluiraju (Orc, Cactoro, Birb, Ninja, Fish, Pigeon, Alien) i njihovi Standing oblici (Orc 7, Cactoro i Birb 8, Frog, Bunny, Ninja, Fish 9) |
| 10-19 | prvi leteći (Goleling, Armabee 10; Glub, Alpaking 11; Pigeon 12; Hywirl 13; TribalFly 14), strelci (Squidle 15, Ghost 16), Dragon 17; Standing Alien i Monkroose (10), Yeti (12), elite (BlueDemon 14, Dino 15, OrcSkull 16) |
| 20+ | elite leteći (*Evolved, preleću zidove), Demon2 (18), GhostSkull (19), mini-bossevi (MushroomKing, Demon, Tribal 20, DragonEvolved 22) |

Napomena: pool je "od nivoa naviše", stariji tipovi ostaju. Ako blobovi na visokim nivoima predominiraju, spusti im težinu.

---

## 11c. Balans (implementirano)

- **Slabljenje starih tipova:** `SetEnemySpawnInfo` ima sedmi parametar `FadeAfter`. Posle tog nivoa težina tipa pada na 10% kroz 10 nivoa (0 = nikad). Roj (Cat, Dog, Chicken, Green, Pink) slabi posle 8, ostali obični tipovi `minNivo + 12..14`. Elite, Yeti i mini-bossevi nikad ne slabe, pa se mešavina sa nivoom pomera ka težim tipovima.
- **Nivo 1 je bez neprijatelja.** Od nivoa 2: talas ima `WaveSize (1) + floor(max(nivo - 2, 0) / 4)` neprijatelja, broj talasa je `Waves (3) + floor(nivo / 10)`, a globalni cap živih je `min(12, 2 + nivo / 2)` (nivo 2: 3, nivo 10: 7, nivo 20+: 12).
- **Skaliranje statistika** (`ApplyLevelScaling`): HP +10% po nivou do 20, pa +5%. Šteta +4% po nivou do 20, pa +2%. Na nivou 100 je to HP x6.9 i šteta x3.4 (ranije HP x12.9 i šteta x8.9, što bi bilo neodrživo uz igrača sa 100 HP).
- **Igrač:** `Health` igrača ima 0.5 s neranjivosti posle pogotka, što ograničava štetu od gomile na `šteta / 0.5 s`, bez obzira koliko ih ima.

---

## 12. Odloženo (sledeći korak)

- Sistem perkova i level-up **pre** podešavanja skaliranja težine: HP +12% i budžet +4 po nivou su samo početne vrednosti, a prava kriva se bira tek kad se zna koliko jak igrač postaje
- Damage i health preko GDevelop ekstenzija ("Health points and damage" i "Fire bullets / 3D fire bullet")
- Sistem perkova i level-up
- Ranged koji drži distancu, Evolved letači preko zidova (`CanCrossWalls`)
