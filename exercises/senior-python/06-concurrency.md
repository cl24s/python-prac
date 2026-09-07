# Concurrency – önálló feladatok

Állapot: feladatkiírás kész; megoldás és review még nincs. Kész megoldás nincs mellékelve. A megoldások és tesztek később a `06-concurrency/` almappába kerülhetnek. Standard library, CPython 3.12+ a cél. Az async tesztekben Eventtel vagy más szinkronizációval vezéreld a sorrendet; tetszőleges sleep és kötelező gyorsulási arány nem elfogadási feltétel.

[Tananyag](../../knowledge/senior-python/06-concurrency/README.md)

## C01

**Cél:** végrehajtási modell megindoklása.

Készíts Markdown-döntési táblát három helyzetre: 500 async HTTP-lekérés, blokkoló SDK-val 20 független objektum lekérése, 100 nagy tiszta Python CPU-feladat. Mindegyikhez válassz modellt, nevezd meg a szükséges limiteket és a fő hibalehetőséget.

- [ ] A választás figyelembe veszi a könyvtár API-ját, GIL/buildet és megosztott állapotot.
- [ ] Nincs bizonyíték nélküli request/sec vagy gyorsulási ígéret.
- [ ] Megnevezed, mit mérnél a döntés ellenőrzéséhez.

**Interview:** Which input change could make you choose a different model?

## C02

**Cél:** valódi átfedés és eredménysorrend.

Írj `run_pair(first, second)` async függvényt. Mindkét argumentum paraméter nélküli async callable string eredménnyel. A két munkát egyszer indítsa el, eredménye tuple legyen a first/second sorrendben. Ha egyik hibázik, a társ leállását és cleanupját meg kell várni, és a hiba nem nyelődhet el. TaskGroup használható; ExceptionGroup kimenet elfogadott és dokumentálandó.

- [ ] Mindkettő eléri a teszt release Event előtti pontot, mielőtt bármelyik eredményt adna.
- [ ] A befejezési sorrend megfordítása nem változtatja a visszatérési sorrendet.
- [ ] Nincs árván továbbfutó saját task hibánál.

**Interview:** Why would consecutive await calls fail the overlap requirement?

## C03

**Cél:** taskhiba és testvér-task takarítása.

Készíts három async feladatot: egy indítási Eventre váró, majd ValueError-t dobó feladatot és két hosszú ideig várakozó feladatot. A hibázó csak akkor dobjon, amikor mindkét társa a cleanupot garantáló try blokkon belül van. TaskGroup fogja össze őket.

- [ ] A ValueError-csoport látható a hívónál.
- [ ] Mindkét társ cleanupja pontosan egyszer megtörténik, és mindkét task cancelled állapotú.
- [ ] Eventek vezérlik az indulást; nincs sleepre alapozott feltételezés.
- [ ] Leírod, mi változna alapértelmezett gather használatával.

**Interview:** Does sibling cancellation undo already completed side effects?

## C04

**Cél:** teljes deadline és megszakítás.

Írj `with_deadline(operation, deadline)` async függvényt. A deadline az aktuális loop monotonic idejéhez tartozó abszolút float; az operation paraméter nélküli async callable. Lejárt határidő és valóban felfüggesztett operation esetén TimeoutError legyen; normál esetben az eredmény változatlanul visszatér. Külső cancellation CancelledError-ként maradjon meg.

- [ ] Múltbeli deadline-nal és Eventen váró művelettel tesztelsz.
- [ ] A művelet finally cleanupja timeout után megtörténik.
- [ ] Külső cancellation nem alakul tévesen timeouttá vagy sikerré.
- [ ] Dokumentálod, miért nem kemény CPU-időkorlát a megoldás.

**Interview:** Why should nested operations share one deadline?

## C05

**Cél:** threadbiztos állapot és kooperatív stop.

Készíts Counter osztályt `increment()` és `value()` művelettel, Lockkal védve. Négy thread egyenként 1000 növelést végezzen, minden hiba legyen megfigyelve, minden thread legyen megvárva. Külön rövid workerpéldában stop Event irányítsa a kilépést, és egy finished Event jelezze a cleanup végét.

- [ ] A számláló pontosan 4000; a value is konzisztens olvasási szerződést ad.
- [ ] Nincs daemon thread és nincs főprogramvégére bízott leállás.
- [ ] A stop teszt nem feltételez minimális futási időt vagy iterációszámot.
- [ ] Leírod, miért nem elég egy futó future cancel-je.

**Interview:** Which operations must be inside the same critical section?

## C06

**Cél:** spawn-kompatibilis processzpool.

Készíts önálló Python-scriptet felső szintű `count_even(limit)` workerrel. A worker a `range(limit)` páros elemeit számolja; limit nem negatív int. A script explicit spawn kontextussal, két workerrel dolgozza fel `[0, 1, 10, 100]` bemenetet. Negatív limit ValueError-ja jusson a szülőhöz.

- [ ] Eredmény input-sorrendben `[0, 1, 5, 50]`.
- [ ] Main guard és importálható worker; nincs lambda/closure a processzhatáron.
- [ ] A pool lezárul és a hibás future eredménye megfigyelt.
- [ ] Nem állítasz gyorsulást ebből a kis példából.

**Interview:** What is copied between the parent and worker processes?

## C07

**Cél:** bounded workerpool és drain/abort.

Írj `consume(source, process, worker_count, capacity)` async függvényt. A source véges, gyors, szinkron int-iterable, nem blokkoló I/O; process async callable int argumentummal, None eredménnyel. Worker_count és capacity pozitív int, bool nélkül. Fix számú worker és bounded asyncio.Queue használható. Ne készíts egy taskot minden inputelemhez, és ne gyűjts teljes eredménylistát.

- [ ] Minden normál input pontosan egyszer kerül processhez; üres inputnál is minden worker leáll.
- [ ] Egyszerre legfeljebb worker_count aktív process-hívás, a queue kapacitása legfeljebb capacity.
- [ ] Normál befejezés megvárja a teljes drain-t és minden worker kilépését.
- [ ] Workerhiba vagy külső cancellation esetén a többi saját task leállását megvárja, a hiba továbbterjed; teljes feldolgozást ilyenkor nem ígér.
- [ ] Sikeresen kivett queue-elemhez task_done tartozik hiba esetén is.
- [ ] Eventes fake processszel teszteled a korlátot, a hibát és a cleanupot; nem int paraméter TypeError, nem pozitív érték ValueError.

**Interview:** Which state could still grow even when the queue is bounded?
