# 🔍 Bűnügyi Krónikák – A Thor-híd rejtélye

> Egy önálló, böngészőben futó nyomozós játék, amely Arthur Conan Doyle „A Thor-híd rejtélye" című Sherlock Holmes-novellájára épül.

---

## Játékleírás

### Mi a játék célja?

Maria Gibson holttestét a Thor-hídnál találják meg – és a gyanú azonnal Grace Dunbarra, a birtok nevelőnőjére terelődik. A játékos feladata, hogy a helyszíneket alaposan megvizsgálva, a tanúkat kihallgatva kiderítse, mi történt valójában, ki a tettes és mi volt az indítéka – mielőtt az idő lejár.

### Szabályok

Az időkeret **200 perc** (játékidő), amelyből minden vizsgálati akció levon:

- **Helyszíni nyom megvizsgálása:** 5 perc
- **Tanú/gyanúsított kihallgatása egy nyomról:** 5 perc

Ha az idő elfogy vádolás előtt, az ügyet lezárják – a játékos veszít. A végső megoldáshoz egyrészt a **helyes tettest**, másrészt a **helyes indítékot** is meg kell jelölni.

### Helyszínek és nyomok

A játék négy vizsgálható helyszínt tartalmaz:

| Helyszín | Leírás |
|---|---|
| 🌉 Thor-híd | A gyilkosság (látszólagos) helyszíne |
| 🌊 Tópart | A víz alatti bizonyítékok |
| 🚪 Grace szobája | A gyanúsított szobája |
| 📚 Gibson dolgozója | A birtok urának magánszobája |

Minden helyszínen **pulzáló ✦ jelzők** mutatják a vizsgálható pontokat. Összesen 11 nyom gyűjthető össze; ezeket a nyomokat aztán 4 különböző tanúnak lehet felmutatni (Grace Dunbar, Neil Gibson, Marlow Bates, Rendőrfelügyelő).

### Irányítás

**Asztali böngészőn:**
- Kattintás a ✦ jelzőre → azonnali vizsgálat (időt von le)
- A bal oldali navigációs panelen lehet helyszínt és lapot váltani

### A megoldás menete

1. Gyűjts össze minél több nyomot a helyszíneken
2. A „Tanúk" lapon kérdezd ki a szereplőket az összegyűjtött bizonyítékokról
3. A „Vádolás" lapon jelöld meg a gyanúsítottat, majd válaszd ki az indítékát
4. Erősítsd meg a vádat – a játék feloldja a megoldást

---

## Magas szintű logika

### Architektúra

A játék egyetlen, önálló HTML fájlban fut – nincs szerver, nincs külső adatbázis. Az összes játékadat, logika és megjelenítés egy fájlba van csomagolva.

### Állapotkezelés

A teljes játékállapotot a `G` nevű globális objektum tárolja:

```js
G = {
  t: 200,           // maradék idő (perc)
  clues: Set,       // összegyűjtött nyomok azonosítói
  examined: Set,    // már megvizsgált hotspot-ok
  asked: {},        // melyik tanúnak melyik nyomot mutattuk meg
  susp: null,       // kiválasztott gyanúsított
  accuse: null,     // vádolt személy azonosítója
  motive: null,     // kiválasztott indíték azonosítója
  over: false,      // vége van-e a játéknak
  actions: 0        // végrehajtott akciók száma (statisztika)
}
```

### Főbb adatstruktúrák

**`CLUES`** – az összes összegyűjthető nyom leírója (név, ikon, helyszín, részletes leírás, nyomozói megjegyzés).

**`SCENES`** – helyszínenként a hotspot-ok listája: pozíció (bal/felső %-ban), felirat, és hogy melyik `CLUES` bejegyzéshez tartoznak.

**`SUSPECTS`** – a négy kihallgatható szereplő adatai és válaszai. Minden szereplőnek minden nyomra egyedi, karakteres reakciója van (`cr` objektum, nyom-azonosítóval kulcsolva).

**`ACCUSE_OPTS`** és **`MOTIVES`** – a vádolható személyek listája és az egyes személyekhez tartozó lehetséges indítékok (köztük pontosan egy helyes).

### Főbb JavaScript függvények

| Függvény | Feladata |
|---|---|
| `initG()` | Játékállapot nullázása (új játék indításakor) |
| `startGame()` | Játék indítása: helyszínek, gyanúsítottak, vádlista felépítése |
| `buildScenes()` | A négy helyszín SVG-alapú vizuális jelenetének legenerálása |
| `addHotspots(cid, spots)` | Kattintható/tapintható ✦ jelzők elhelyezése az adott helyszínen; kezeli az asztali és mobilos interakciót külön |
| `examineSpot(sp)` | Egy hotspot megvizsgálása: időlevonás, nyom hozzáadása a gyűjteményhez, modális ablak megjelenítése |
| `isMobileDevice()` | Megállapítja, hogy mobilon fut-e a játék (viewport-szélesség alapján) |
| `closeAllMobilePopups()` | Az összes nyitott mobilos popup bezárása |
| `selectSusp(sid)` | Gyanúsított kiválasztása kihallgatáshoz; betölti az elérhető nyomgombokat |
| `doClueQ(sid, cid, btn)` | Egy nyomról kérdezi az adott tanút: időlevonás, válasz megjelenítése és előzménynaplóba írás |
| `proceedToMotive()` | Átlépés a vádolás második fázisára (indíték kiválasztása) |
| `confirmAccuse()` | Végleges vád megerősítése, helyes/helytelen eredmény kiértékelése |
| `spendT(m)` | Időlevonás és a kijelző frissítése; lejáráskor automatikusan befejezi a játékot |
| `updateDisp()` | Időkijelző, időcsík és nyomszámláló frissítése |
| `showModal(ico, ti, tx, clue)` | Modális ablak megjelenítése (vizsgálati eredmény) |
| `refreshCI()` | Nyomkészlet lap frissítése az aktuális `G.clues` alapján |
| `doReveal(opt, mv)` | Megoldás képernyő megjelenítése az eredménnyel és a teljes sztorival |
| `restartGame()` | Teljes játékállapot visszaállítása és újraindítás |
| `showTab(id)` / `showScreen(id)` | Navigáció lapok és képernyők között |

### Képernyőfolyam

```
device-select → [mobil: rotate-screen] → intro → game → reveal
```

A `game` képernyőn belül 5 lap érhető el: Bridge · Lake · Grace · Study · Clues · Suspects · Accuse.

---

## AI eszközök használata

A projekt fejlesztése során **Claude (Anthropic)** mesterséges intelligencia-asszisztenst használtunk, amely a következő területeken nyújtott kiemelkedő segítséget:

### Tartalom és sztori
Claude segített a játék narratívájának kidolgozásában: a helyszínek részletes leírásait, a nyomok szövegét, a tanúk egyedi, karakteres válaszreakcióit és a teljes megoldás-sztorit Claude generálta, összhangban az eredeti Conan Doyle-novella logikájával.

### Kódgenerálás és architektúra
Az egész játék egyetlen HTML fájlban valósult meg; Claude segített a fájlstruktúra megtervezésében, az állapotkezelési logika kialakításában és a JavaScript függvények megírásában.

### Vizuális dizájn
A sötét, arany-kontrasztos, retró krimis esztétika (CSS változók, animációk, betűtípus-párosítások) kialakításában Claude designtanácsokat adott és a CSS kódot is generálta.

### Debugging és iteráció
A fejlesztés során felmerülő hibák (pl. popup nem jelenik meg, idő nem vonódik le a helyes pillanatban, mobilos és asztali viselkedés szétválasztása) diagnosztizálásában és javításában Claude volt az elsődleges eszköz.

