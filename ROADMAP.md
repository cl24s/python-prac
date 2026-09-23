# Tanulási útvonal és haladás

Ez az aktuális tanulási sorrend egyetlen forrása. A tudástérkép tartalomjegyzék, nem kötelező olvasási sorrend. A cél önálló Python-programozás, majd tesztelhető backend- és adatfeldolgozó alkalmazások; nincs rögzített felkészülési határidő.

## Aktuális munkamenet

- Aktív szakasz: indulás és kiindulópont tisztázása.
- Aktív kódolási feladat: még nincs elkezdettként rögzítve.
- Következő lépés: működő futtatókörnyezet esetén [D01](exercises/senior-python/02-data-structures.md#d01), első önálló próbálkozás és saját tesztek.
- Ha a környezet hiányzik: [futtatás](knowledge/senior-python/00-foundations/01-running-python.md). Ha a függvény/ciklus nem áll össze: [alapozás](knowledge/senior-python/00-foundations/02-writing-small-programs.md) és [L01–L02](exercises/senior-python/00-foundations.md#l01).
- Elakadás: még nincs megfigyelt és rögzített konkrétum. A meglévő szakmai háttérből nem következtetünk automatikusan Python-szintre.

## Állapotok és bizonyíték

**Feladat:** tervezett → folyamatban → kész; szükség esetén ismétlendő. Kész csak teljesített elfogadási feltételekkel, rögzített tesztfuttatással vagy kifejezetten megnevezett ellenőrzési korláttal.

**Készség:** még nem mért → segítséggel alkalmazott → önállóan alkalmazott → később új változaton is stabil. A „stabil” nem állásinterjú-garancia vagy senioritási minősítés.

A dokumentáció elkészítése, a példák futtatása és a tanuló teljesítménye három külön adat. Egy terület akár kevesebb feladattal is továbbengedhető, ha új változaton bizonyított; egy nehezebb pontnál kisebb lépésekre bontunk.

| Dátum | Feladat / készség | Állapot | Mi ment önállóan? | Segítség / elakadás | Ellenőrzés | Következő ismétlés |
| --- | --- | --- | --- | --- | --- | --- |
| — | D01 / szűrés és bemenetmegőrzés | tervezett | még nincs mérés | még nincs mérés | még nem futott | első próbálkozás után kijelölendő |

## Fokozatos sorrend

A felsorolt feladatok útvonaljelölők, nem egyszerre kiadott házi feladatok. Tesztelés és a saját megoldás elmagyarázása minden szakaszban jelen van. Az időkorlátos interjúszimuláció későbbi, külön üzemmód.

| Szakasz | Fókusz és meglévő anyag | Gyakorlati útvonal | Továbblépési bizonyíték |
| --- | --- | --- | --- |
| 0. Indulás | [Interpreter és környezet](knowledge/senior-python/00-foundations/01-running-python.md) | Script futtatása, majd D01 mint kiindulópont. | Tudod, melyik Python fut; nem az eszközbeállítás akadályoz. |
| 1. Rövid függvények | [Gyakorlati nyelvi alapok](knowledge/senior-python/00-foundations/02-writing-small-programs.md), célzottan referenciák és másolás. | L01, L02, D01, B01; a már bizonyított részeket nem ismételjük mechanikusan. | Önálló szűrés, határérték, return, új konténer és bemenetmegőrzés egy új változaton is. |
| 2. Adatfeldolgozás | [Adatszerkezetek](knowledge/senior-python/02-data-structures/README.md), [tesztelés és hibakeresés](knowledge/senior-python/00-foundations/03-testing-and-debugging.md). | L03, L04, D02, D04, D05, L05; saját assert után pytest és Q02. | Index, aggregáció és rendezés; hibajavítás regressziós teszttel; saját megoldás költségének magyarázata. |
| 3. Használható kis program | [Fájlok](knowledge/senior-python/00-foundations/04-files-and-programs.md), hibakezelés, iterálás. | L06, L07, L08, E01, B03, D06; majd [logelemző CLI](projects/01-log-analyzer/README.md). | Valódi mesterséges fájlból tesztelt program; I/O és logika külön; üres/hibás input és fájlhiba szabályozott. |
| 4. Strukturált Python | [OOP](knowledge/senior-python/03-oop/README.md), [típusok](knowledge/senior-python/04-types-and-interfaces/README.md), [erőforrások](knowledge/senior-python/05-error-handling/README.md). | O01, O02, O04; a többi feladat a tényleges hiányok alapján. A logelemző kis refaktorálása Q05-tel. | Indokolt osztály vagy függvény; külön tesztelhető függőségek; állapot és hibautak érthetők. |
| 5. Backend és adatbázis | [HTTP](knowledge/senior-python/08-http-and-backend/README.md), [FastAPI](knowledge/senior-python/09-fastapi/README.md), [SQL](knowledge/senior-python/10-databases/README.md). | Kicsi szinkron use case → HTTP-határ → helyi adatbázis és teszt; [job processing API](projects/03-job-processing-api/README.md) első, szűk mérföldköve. | Működő, tesztelt API-rész; validáció, hibák, tranzakcióhatár és SQL alapok megmagyarázhatók. |
| 6. Konkurencia és megbízhatóság | [Concurrency](knowledge/senior-python/06-concurrency/README.md), [megbízhatóság](knowledge/senior-python/11-reliable-services/README.md). | Először C01–C02, majd szükség szerint cancellation/limitek; [endpoint checker](projects/02-endpoint-checker/README.md), később worker a job API-hoz. | Megindokolt sync/async/thread/process választás; korlátok és hiba esetén cleanup tesztelve. |
| 7. Elmélyítés és irányválasztás | [Teljesítmény](knowledge/senior-python/12-performance-and-debugging/README.md), [üzemeltetés](knowledge/senior-python/13-packaging-security-and-operations/README.md), [tervezés](knowledge/senior-python/14-design-and-collaboration/README.md). | Mérés a saját projekten; opcionális adatfeldolgozás/AI irány és külön interjúgyakorlat. | Új igényt önállóan végigviszel, tesztelsz és megmagyarázol; a hiányok célzottan azonosíthatók. |

Az 5–6. szakasz részletei a választott projekt alapján összefonódhatnak. Nem követelmény mind a 94 régi feladat vagy az összes haladó téma teljesítése bármilyen állásjelentkezés előtt.

## Mi került most be, és mi hiányzik még?

| Terület | Állapot | Hatókör / következő konkrét cél |
| --- | --- | --- |
| Egyszerű függvény, ciklus, string és konténerhasználat | 4 alapozó fejezet részeként kidolgozva | L01–L04; a mélyebb referenciák a régi fejezetekben. |
| Korai hibakeresés és tesztírás | Kidolgozva | L05; traceback, debugger, saját expected érték; nem csak teszteléselmélet. |
| Futtatás, fájlok, JSON és CSV | Kidolgozva | L06–L08; lokális környezet és önálló kis programok. |
| Ismétlés és ismeretlen feladatváltozat | Munkamenet-szabály kidolgozva | Minden lezárt készséghez külön alkalommal új változat; nem előre publikált megoldás. |
| Algoritmikus problémamegoldási minták | Célzott bővítés még tervezett | Hash-alapú keresés már van; később stabil deduplikáció, stack/queue, kézi binary search, két mutató/csúszóablak; fa/gráf BFS/DFS csak ezek után, cél szerint. Nem LeetCode-maraton. |
| Dátum, időzóna, numerikus pontosság | Rövid gyakorlati bővítés még tervezett | Timestamp-normalizáló feladat, aware/naive idő különbsége; összegzésnél float vs. Decimal vagy egész legkisebb egység és kerekítési szerződés. |
| NumPy és Pandas | Opcionális szakirány, leckék még nincsenek kidolgozva | NumPy: array, shape, dtype, mask, broadcasting és view/copy. Pandas: beolvasás, hiányzó adat, csoportosítás, join, dátum. Egy ismert CSV-feladat megoldása standard libraryvel, majd Pandasszal, eredmény- és memória-összehasonlítással. |
| Környezet reprodukálhatósága | Első projektnél konkretizálandó | Függőségek és futtatási parancsok rögzítése, később pyproject/lint/típusellenőrzés. Nem minden eszköz az első napon. |
| RAG/adat-ingest projekt | Későbbi projektirány, nincs implementálva | Először fájl → normalizált dokumentum pipeline, majd külön tesztelhető ingest/storage/retrieval; moduláris monolit. Queue/Kubernetes/hexagonális szerkezet csak indokolt lépésben. |

A fenti bővítési lista nem újabb kötelező kezdő tananyag. Csak a következő szükséges részt dolgozzuk ki. Hivatalos kiindulópontok az opcionális adatirányhoz: [NumPy basics](https://numpy.org/doc/stable/user/absolute_beginners.html), [Pandas tutorials](https://pandas.pydata.org/docs/getting_started/intro_tutorials/).

## Meglévő tananyag és projektállapot

A régi 14 témakör első változata elkészült; mindegyik elsajátítási állapota továbbra is **tervezett**, amíg nincs rögzített bizonyíték. Ez a repó dokumentált állapota, nem tudásszintbecslés. A [tudástérkép](knowledge/senior-python/README.md) és [feladatjegyzék](exercises/README.md) tartalmaz minden régi témát. A kódpéldák korábbi ellenőrzése a [VALIDATION](knowledge/senior-python/VALIDATION.md) fájlban maradt.

| Projekt | Állapot | Következő mérföldkő |
| --- | --- | --- |
| [Logelemző CLI](projects/01-log-analyzer/README.md) | tervezett | A 3. szakasznál M1 pontosítása: logformátum, input/output, tesztek. |
| [Párhuzamos endpoint checker](projects/02-endpoint-checker/README.md) | tervezett | A 6. szakasznál szűk, tesztelhető M1. |
| [Job processing API](projects/03-job-processing-api/README.md) | tervezett | Az 5. szakasznál kis szinkron use case; valódi háttérworker később. |

## Munkamenetnapló

A korábbi sorok történeti feljegyzések; az akkori „következő lépés” nem írja felül a fenti aktuális tervet.

| Dátum | Elvégzett munka | Tanulság / következő lépés |
| --- | --- | --- |
| 2026-09-06 | Induló Markdown-struktúra elkészült | Tanulás még nem kezdődött; első téma következik |
| 2026-09-06 | Senior Python témalista rögzítve; OOP és FastAPI külön kiemelve | Részletes kidolgozás még tervezett; első téma következik |
| 2026-09-06 | Külső knowledge-témavázak eltávolítva; egyetlen senior-python tananyagstruktúra maradt | Hivatkozások és haladáskövetés az egységes tervhez igazítva |
| 2026-09-06 | Első két témakör kidolgozva: 14 fejezet, 29 ellenőrzött kódblokk, 12 önálló feladat | CPython 3.12.13; a tanulási állapot továbbra is tervezett; stílus/mélység áttekintése következik |
| 2026-09-06 | OOP és típusok kidolgozva: 13 új fejezet, 21 futtatott kódblokk, 12 önálló feladat | 13 típusos blokk ellenőrizve mypy strict módban, ebből 2 elvárt negatív minta; elsajátítás továbbra is tervezett |
| 2026-09-07 | Hibakezelés és concurrency kidolgozva: 12 új fejezet, 16 futtatott kódblokk, 12 önálló feladat | CPython 3.12.13, asyncio debug, explicit spawn; elsajátítás továbbra is tervezett; következő tananyag: 7. Tesztelés és kódminőség |
| 2026-09-07 | Tesztelés és HTTP/backend kidolgozva: 13 új fejezet, 11 ellenőrzött kódblokk (24 pytest-eset és 5 script), 13 önálló feladat | CPython 3.12.13, pytest 9.1.1, HTTPX 0.28.1; elsajátítás továbbra is tervezett; következő: 9. FastAPI |
| 2026-09-07 | FastAPI és adatbázisok kidolgozva: 18 új fejezet, 17 ellenőrzött kódblokk, 18 sikeres pytest-eset, 18 önálló feladat | SQLite/SQLAlchemy integráció és FastAPI TestClient; PostgreSQL/Redis/deploy nem futott; következő: 11. Megbízható szolgáltatások és háttérfeldolgozás |
| 2026-09-07 | Megbízhatóság és teljesítmény kidolgozva: 13 új fejezet, 13 ellenőrzött kódblokk, 19 sikeres pytest-eset, 13 önálló feladat | Helyi deduplikáció/outbox, cProfile, tracemalloc, stackdiagnosztika és timeit; broker/deploy/load teszt nem futott; következő: 13. Csomagolás, biztonság és üzemeltetés |
| 2026-09-07 | A 13–14. témakör kidolgozva: 14 új fejezet, 8 futtatott Python-blokk, 27 sikeres pytest-eset, 14 önálló feladat | A teljes 14 témakör első változata elkészült: 97 fejezet, 94 feladat; tanulás még tervezett. Következő: Python alapok és B01, majd review |
| 2026-09-23 | Fokozatos tanulási útvonal, START_HERE, 4 alapozó fejezet, 8 új kiírás és tanulás/ellenőrzés szétválasztása | Dokumentációs bővítés, nem felhasználói teljesítés. Következő: futtatókörnyezet ellenőrzése és D01; szükség esetén L01–L02. |
