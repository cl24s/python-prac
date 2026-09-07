# Teljesítmény és hibakeresés – önálló feladatok

A feladatokhoz kész megoldás nincs mellékelve. A kód és tesztek később saját almappába kerüljenek; a leírás és a review Markdown legyen. Minden megoldásnál rögzítsd a futtatott parancsot, eredményt és a nem ellenőrzött környezetet.

## P01

### Latencyjelentés torzítás nélkül

**Cél és szerződés:** Készíts summarize(samples) függvényt latency_ms és outcome mezős mintákhoz. Adjon darabszámot, siker/hiba számot, összes minta átlagát és nearest-rank p95/p99-et. Minden latency véges, nem negatív; az outcome success vagy error.

**Elfogadási feltételek:**

- 95 darab 10 ms-os siker és 5 darab 1000 ms-os hiba p95=10, p99=1000, átlag=59,5; a hibákat ne hagyd ki.
- Üres minta és érvénytelen latency/outcome dokumentált hibát ad.
- Írd le, hogyan választanád szét a sikeres/hibás populációt, és miért nem átlagolható worker-p99.

**Korlát és review:** A mesterséges minta nem az alkalmazás éles teljesítménye. A jelentés a percentilis módszerét is nevezze meg.

## P02

### Profiler alapján választott javítás

**Cél és szerződés:** Válassz egy saját, már helyesen működő feldolgozó függvényt a korábbi feladatokból. Profilozd reprezentatív, rögzített bemeneten cProfile-lal; készíts profile-review.md-t.

**Elfogadási feltételek:**

- A profilfájl menthető és pstats-szal visszaolvasható; a saját és kumulatív költséget megkülönbözteted.
- Nevezz meg egy konkrét hipotézist hívásszám vagy algoritmikus költség alapján.
- Egy indokolt változtatás után az eredmény változatlan, a profilt azonos bemenettel megismétled.
- Ha nincs indokolt optimalizálás, ezt bizonyítékkal dokumentáld; ne találj ki gyorsulást.

**Korlát és review:** Nem kell új profilerkönyvtár. A profilerrel mért futásidőt ne kezeld profiler nélküli benchmarkként.

## P03

### Memóriában gyűjtés és stream

**Cél és szerződés:** Implementálj két azonos eredményű bájtblokk-feldolgozást: egy teljes listába gyűjtőt és egy streamet fogyasztót. Az összegzés a blokkok összes bytehosszát adja, nem őrzi meg a payloadot.

**Elfogadási feltételek:**

- Az eredmény több batchméreten azonos.
- Tracemalloc-kal külön, friss mérésben rögzíts peak allokációt, és mondd ki a mért adat korlátját.
- Írj le legalább két referenciatulajdonost, amely a streamet mégis korlátlan memóriájúvá tehetné.

**Korlát és review:** Ne állíts RSS-javulást pusztán tracemalloc alapján. Ne tegyél szűk, gépfüggő abszolút byteküszöböt a tesztbe.

## P04

### Batch lekérdezés szerződésmegőrzéssel

**Cél és szerződés:** Készíts SQLite-adaptert ID-listához. A kimenet a bemenet sorrendjében nevek listája, duplikált ID ismételt eredményt ad; hiányzó ID esetén saját MissingJob hiba. Hasonlítsd össze a soronkénti és legfeljebb 100 ID-s batch változatot.

**Elfogadási feltételek:**

- Üres lista, ismételt ID, fordított sorrend és hiányzó rekord is tesztelt.
- A trace_callback megmutatja a SELECT-hívások számát; legalább 250 bemeneti ID legyen a batch-tesztben.
- Az értékek minden esetben bind paraméterek maradnak.
- A mérési jegyzet ne állítson hálózati gyorsulást a helyi queryszám alapján.

**Korlát és review:** Definiáld, a batch-es deduplikálás hogyan befolyásolja a hívásszám elvárását. A hiányzó ID ne változzon véletlen IndexErrorrá.

## P05

### Lefagyás diagnosztikai próba

**Cél és szerződés:** Készíts kontrollált, Eventre váró threadet és egy váró async taskot. Gyűjts threadstacket fájlba és taskállapotot név alapján; majd mindkettőt rendezetten fejezd be.

**Elfogadási feltételek:**

- A várt függvény megjelenik a thread-dumpban, a task névvel és nem üres stackkel látszik.
- A bizonyíték összegyűjtése után nincs élő segédthread vagy pending task.
- Incident-notes.md különítse el a szabályos várakozást a deadlocktól, és adjon következő vizsgálati lépést blokkolt event loopra.

**Korlát és review:** Ne hozz létre feloldhatatlan deadlockot és ne nyiss publikus diagnosztikai endpointot. A dumpból csak szükséges, érzékeny adat nélküli rész kerüljön dokumentumba.

## P06

### Összehasonlítható benchmark és kapacitásterv

**Cél és szerződés:** Egy korábbi feladat két azonos viselkedésű implementációját mérd timeit-tal. Készíts benchmark.md-t bemenettel, környezettel, setuphatárral, ismétlésekkel és nyers eredményekkel.

**Elfogadási feltételek:**

- A helyességet külön ellenőrzöd; legalább három ismétlés, hívásonkénti idő és azonos bemenet szerepel.
- Hideg/bemelegített állapot, előfeldolgozás és GC-beállítás dokumentált.
- Külön tervezd meg egy job API terheléspróbáját: érkezési ráta, konkurencia, tail latency, hibák, memória és queue-kor.
- A 100 kérés/s, 0,2 s átlagidő stabil példáját értelmezd, és indokold, miért nem egyenlő 20 DB-kapcsolattal.

**Korlát és review:** Valódi terheléspróbát nem kell most indítani. Mikrobenchmarkból ne adj általános request/sec ígéretet.

[Tananyag](../../knowledge/senior-python/12-performance-and-debugging/README.md) · [Gyakorlatok](../README.md)
