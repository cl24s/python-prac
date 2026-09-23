# Tanulási útvonal és haladás

Ez az aktuális tanulási sorrend és állapot egyetlen forrása. A tudástérkép tartalomjegyzék, nem kötelező olvasási sorrend. A cél önálló Python-programozás, majd tesztelhető backend- és adatfeldolgozó alkalmazások; nincs rögzített felkészülési határidő.

**Tanulási mód:** kész gyakorlati feladatkiírás → saját kód és teszt → szükséges elmélet és review → a program következő bővítése. Nem előbb a teljes tananyag végigolvasása.

## Aktuális munkamenet

- Python-futtatás: **kész a felhasználó 2026-09-23-i jelzése alapján**. Az agent ezt nem helyi futtatással ellenőrizte; más készség teljesítését nem igazolja.
- Aktív útvonal: [Feladatkezelő — TM01–TM03](exercises/task-manager/README.md).
- Következő kódolási rész: [TM01 / 1. rész — normalize_title](exercises/task-manager/01-types-and-validation.md). Első saját függvény és tesztek a `work/` mappában.
- Felhasználói implementáció: még nincs bemutatva vagy review-zva. A három kiírás elkészült, ettől a feladatok nem teljesítettek.
- D01 nem lett késznek jelölve és nem kötelező belépő többé; célzott kisegítő gyakorlatként megmarad.
- Elakadás: még nincs megfigyelt konkrétum. A meglévő szakmai háttérből nem következtetünk automatikusan Python-szintre.

## Állapotok és bizonyíték

**Feladat:** tervezett → folyamatban → kész; szükség esetén ismétlendő. Kész csak teljesített elfogadási feltételekkel, rögzített tesztfuttatással vagy kifejezetten megnevezett ellenőrzési korláttal.

**Készség:** még nem mért → segítséggel alkalmazott → önállóan alkalmazott → később új változaton is stabil. A „stabil” nem állásinterjú-garancia vagy senioritási minősítés.

A dokumentáció elkészítése, a példák futtatása és a tanuló teljesítménye három külön adat. Az alábbi környezetbejegyzés felhasználói visszajelzés, nem feladatmegoldás. Egy terület kevesebb feladattal is továbbengedhető, ha új változaton bizonyított; nehezebb pontnál kisebb lépésekre bontunk.

| Dátum | Feladat / készség | Állapot | Mi ment önállóan? | Segítség / elakadás | Ellenőrzés | Következő lépés |
| --- | --- | --- | --- | --- | --- | --- |
| 2026-09-23 | Python-futtatás | kész, felhasználói jelzés | Python fut | nincs jelzett hiba | agent nem futtatta a felhasználó gépén | TM01 első rész |
| — | TM01 / típusok és validáció | tervezett; kiírás kész | még nincs bemutatott kód | még nincs mérés | saját megoldás nem tesztelt | normalize_title |
| — | TM02 / adatszerkezetek | tervezett; kiírás kész | még nincs bemutatott kód | még nincs mérés | saját megoldás nem tesztelt | TM01 után |
| — | TM03 / OOP | tervezett; kiírás kész | még nincs bemutatott kód | még nincs mérés | saját megoldás nem tesztelt | TM02 után |

## Feladatalapú fő útvonal

| Lépés | Megépítendő rész | Továbblépési bizonyíték |
| --- | --- | --- |
| 0. Futtatás — kész | Meglévő Python-környezet használata. | A felhasználó jelzése szerint fut. Nem ismétlendő telepítési feladat. |
| 1. [TM01](exercises/task-manager/01-types-and-validation.md) | Cím normalizálása, prioritás ellenőrzése, feladatrekord létrehozása. | Működő saját függvények, normál és hibás bemenetek tesztjei; type hint és validáció különbsége érthető. |
| 2. [TM02](exercises/task-manager/02-data-structures.md) | ID-index, prioritásos tennivalólista és összesítés. | List/dict/set tudatos használata, rendezési szabály, duplikáció és referencia-megosztás tesztelve. |
| 3. [TM03](exercises/task-manager/03-oop.md) | Task és TaskManager osztály; készre állítás és lekérdezés. | Elkülönített példányállapot, publikus műveletek, saját tesztek és futtatható demo. |
| 4. API — kiírás később | Ugyanehhez a feladatkezelőhöz kicsi FastAPI-felület. | A pontos HTTP-szerződést és elfogadási feltételeket a feladat előtt rögzítjük; most nincs kész API. |
| 5. Tárolás és megbízhatóság — később | Fájl vagy SQL-tárolás, majd indokolt helyen konkurencia és háttérmunka. | A választott új igényhez külön tesztelt rész; nem minden technológia egyszerre. |

A teljes kiírás előre olvasható, de egyszerre egy feladatrésszel dolgozunk. A régi feladatbank nem kötelező párhuzamos útvonal. A TM01–TM03 nem azonos a régi T01/D01/O01 feladatokkal, külön azonosítót kaptak.

## Referencia és választható kiegészítés

| Amikor erre van szükség | Meglévő anyag és gyakorlat |
| --- | --- |
| Függvény, ciklus, string, típus és másolás | [Gyakorlati alapok](knowledge/senior-python/00-foundations/02-writing-small-programs.md), [nyelvi alapok](knowledge/senior-python/01-python-basics/README.md); L01–L04, D01, B01. |
| Kollekciók, rendezés és komplexitás | [Adatszerkezetek](knowledge/senior-python/02-data-structures/README.md); D02–D06. |
| Teszt és hibakeresés | [Korai tesztelés](knowledge/senior-python/00-foundations/03-testing-and-debugging.md), [teljes témakör](knowledge/senior-python/07-testing-and-quality/README.md); L05, Q02, Q05. Már az első saját feladathoz használjuk. |
| Objektumok és interfészek | [OOP](knowledge/senior-python/03-oop/README.md), [typing](knowledge/senior-python/04-types-and-interfaces/README.md); O01–O02, O04. Öröklési trükkök nem belépési feltételek. |
| Fájl és kis CLI | [Fájlok](knowledge/senior-python/00-foundations/04-files-and-programs.md), [hibakezelés](knowledge/senior-python/05-error-handling/README.md); L06–L08, E01, B03, majd választható logelemző. |
| HTTP és adatbázis | [HTTP](knowledge/senior-python/08-http-and-backend/README.md), [FastAPI](knowledge/senior-python/09-fastapi/README.md), [SQL](knowledge/senior-python/10-databases/README.md). |
| Konkurencia és megbízhatóság | [Concurrency](knowledge/senior-python/06-concurrency/README.md), [megbízhatóság](knowledge/senior-python/11-reliable-services/README.md); először C01–C02, aztán indokolt bővítés. |
| Elmélyítés saját programon | [Teljesítmény](knowledge/senior-python/12-performance-and-debugging/README.md), [üzemeltetés](knowledge/senior-python/13-packaging-security-and-operations/README.md), [tervezés](knowledge/senior-python/14-design-and-collaboration/README.md). |

A feladatbank teljes listája az [exercises áttekintésben](exercises/README.md) található. Nem követelmény mind a 94 régi feladat vagy az összes haladó téma teljesítése bármilyen állásjelentkezés előtt.

## Mi van kész, és mi hiányzik még?

| Terület | Állapot | Hatókör / következő konkrét cél |
| --- | --- | --- |
| Feladatalapú típusok, adatszerkezetek és OOP | TM01–TM03 kiírása kész; megoldás nincs | Feladatkezelő három lépésben, pontos szerződéssel, tesztesetekkel és saját demo céllal. |
| Egyszerű függvény, ciklus, string és konténerhasználat | 4 alapozó fejezet részeként kidolgozva | L01–L04 választható kisegítés; mélyebb referenciák a régi fejezetekben. |
| Korai hibakeresés és tesztírás | Kidolgozva | L05; traceback, debugger, saját expected érték; tesztírás a gyakorlati feladatokon is. |
| Futtatás, fájlok, JSON és CSV | Tananyag kidolgozva; csak a futtatás teljesítése jelzett | L06–L08 későbbi fájlkezelési gyakorlatnak. |
| Ismétlés és ismeretlen feladatváltozat | Munkamenet-szabály kidolgozva | Lezárt készséghez külön alkalommal új változat; nem előre publikált megoldás. |
| Algoritmikus problémamegoldási minták | Célzott bővítés még tervezett | Később stabil deduplikáció, stack/queue, kézi binary search, két mutató/csúszóablak; fa/gráf BFS/DFS ezek után, cél szerint. |
| Dátum, időzóna, numerikus pontosság | Rövid gyakorlati bővítés még tervezett | Timestamp-normalizálás; aware/naive idő; float vs. Decimal vagy egész legkisebb egység és kerekítési szerződés. |
| NumPy és Pandas | Opcionális szakirány, leckék még nincsenek kidolgozva | NumPy: array, shape, dtype, mask, broadcasting, view/copy. Pandas: beolvasás, hiányzó adat, csoportosítás, join, dátum. Ismert CSV-feladat összehasonlító megoldásával. |
| Környezet reprodukálhatósága | A program első külső függőségeinél konkretizálandó | Függőségek és futtatás rögzítése, később pyproject/lint/típusellenőrzés. |
| RAG/adat-ingest projekt | Későbbi projektirány, nincs implementálva | Fájl → normalizált dokumentum, majd külön tesztelhető ingest/storage/retrieval; moduláris monolit. Queue/Kubernetes/hexagonális struktúra indokolt lépésben. |

A bővítési lista nem újabb kötelező kezdő tananyag. Hivatalos kiindulópont az opcionális adatirányhoz: [NumPy basics](https://numpy.org/doc/stable/user/absolute_beginners.html), [Pandas tutorials](https://pandas.pydata.org/docs/getting_started/intro_tutorials/).

## Meglévő tananyag és projektállapot

A régi 14 témakör első változata elkészült; mindegyik elsajátítási állapota továbbra is **tervezett**, amíg nincs rögzített bizonyíték. Ez a repó dokumentált állapota, nem tudásszintbecslés. A [tudástérkép](knowledge/senior-python/README.md) tartalmazza a régi témákat. A kódpéldák korábbi ellenőrzése a [VALIDATION](knowledge/senior-python/VALIDATION.md) fájlban maradt; a mostani feladatkiírások ezt nem futtatják újra.

| Projekt | Állapot | Következő mérföldkő |
| --- | --- | --- |
| [Feladatkezelő](exercises/task-manager/README.md) | TM01–TM03 kiírás kész, implementáció tervezett | TM01 első rész; később API-bővítés. |
| [Logelemző CLI](projects/01-log-analyzer/README.md) | tervezett, választható | Fájlkezelés gyakorlásakor M1 pontosítása. Nem kötelező a TaskManager API elé. |
| [Párhuzamos endpoint checker](projects/02-endpoint-checker/README.md) | tervezett, választható | A concurrency témánál szűk, tesztelhető M1. |
| [Job processing API](projects/03-job-processing-api/README.md) | tervezett, későbbi külön projekt | Valódi háttérworker és hibakezelés; ne építsünk egyszerre két bevezető API-t. |

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
| 2026-09-23 | Felhasználó jelzése: Python fut; tanulás kész feladatkiírásokon keresztül. TM01–TM03 elkészült az exercises/task-manager alatt. | Futtatás kész, programozási feladatok nem teljesítettek. Következő: TM01 normalize_title és saját tesztek. |
