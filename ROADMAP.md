# Roadmap és haladás

## Állapotok

- **tervezett**: még nem kezdtük el.
- **folyamatban**: aktív tanulás vagy megoldás.
- **kész**: megértés ellenőrizve / elfogadási feltételek teljesültek.
- **ismétlendő**: visszatérünk rá; az okot a megjegyzésben rögzítjük.

A fájlok létrejötte nem jelent tanulási haladást.

## Tananyag és haladás

A [Senior Python tudástérkép](knowledge/senior-python/README.md) az egyetlen tananyagterv. A részletes fejezeteket a knowledge/senior-python/ könyvtárban dolgozzuk ki, külön OOP- és FastAPI-témakörrel.

| Téma | Tananyag | Elsajátítás |
| --- | --- | --- |
| [Python működése és nyelvi alapok](knowledge/senior-python/01-python-basics/README.md) | kidolgozott első változat | tervezett |
| [Adatszerkezetek és algoritmikus gondolkodás](knowledge/senior-python/02-data-structures/README.md) | kidolgozott első változat | tervezett |
| [OOP és objektumtervezés — kiemelt témakör](knowledge/senior-python/03-oop/README.md) | kidolgozott első változat | tervezett |
| [Típusok és interfészek](knowledge/senior-python/04-types-and-interfaces/README.md) | kidolgozott első változat | tervezett |
| [Hibakezelés és erőforrás-kezelés](knowledge/senior-python/05-error-handling/README.md) | kidolgozott első változat | tervezett |
| [Concurrency és párhuzamos végrehajtás](knowledge/senior-python/06-concurrency/README.md) | kidolgozott első változat | tervezett |
| [Tesztelés és kódminőség](knowledge/senior-python/07-testing-and-quality/README.md) | kidolgozott első változat | tervezett |
| [HTTP és backend alapok](knowledge/senior-python/08-http-and-backend/README.md) | kidolgozott első változat | tervezett |
| [FastAPI — kötelező, önálló témakör](knowledge/senior-python/09-fastapi/README.md) | kidolgozott első változat | tervezett |
| [Adatbázisok és adatkezelés](knowledge/senior-python/10-databases/README.md) | kidolgozott első változat | tervezett |
| [Megbízható szolgáltatások és háttérfeldolgozás](knowledge/senior-python/README.md#11-megbízható-szolgáltatások-és-háttérfeldolgozás) | tervezett | tervezett |
| [Teljesítmény és hibakeresés](knowledge/senior-python/README.md#12-teljesítmény-és-hibakeresés) | tervezett | tervezett |
| [Csomagolás, biztonság és üzemeltetés](knowledge/senior-python/README.md#13-csomagolás-biztonság-és-üzemeltetés) | tervezett | tervezett |
| [Senior szintű tervezés és együttműködés](knowledge/senior-python/README.md#14-senior-szintű-tervezés-és-együttműködés) | tervezett | tervezett |

Az első két témához [Python alapok](exercises/senior-python/01-python-basics.md) és [adatszerkezetek](exercises/senior-python/02-data-structures.md) feladatsor készült, konkrét elfogadási feltételekkel. Az [OOP](exercises/senior-python/03-oop.md) és a [típusok és interfészek](exercises/senior-python/04-types-and-interfaces.md) témához is 6–6 konkrét feladat készült. A [hibakezelés](exercises/senior-python/05-error-handling.md) 5, a [concurrency](exercises/senior-python/06-concurrency.md) 7 új feladatot kapott. A [tesztelés](exercises/senior-python/07-testing-and-quality.md) 6, a [HTTP és backend](exercises/senior-python/08-http-and-backend.md) 7 új feladatot kapott. A [FastAPI](exercises/senior-python/09-fastapi.md) 11, az [adatbázisok](exercises/senior-python/10-databases.md) 7 új feladatot kapott. A régebbi [gyakorlatötletek](exercises/README.md) továbbra is vázak. A tananyag elkészülte nem változtatja készre az elsajátítás állapotát.

## Projektek

| Projekt | Állapot | Következő mérföldkő |
| --- | --- | --- |
| [Logelemző CLI](projects/01-log-analyzer/README.md) | tervezett | M1 pontosítása |
| [Párhuzamos endpoint checker](projects/02-endpoint-checker/README.md) | tervezett | M1 pontosítása |
| [Job processing API](projects/03-job-processing-api/README.md) | tervezett | M1 pontosítása |

## Javasolt sorrend

1. Nyelvi alapok, OOP és objektumtervezés, típusok és interfészek, hibakezelés; a tesztelés alapjait már az első gyakorlatokhoz használjuk.
2. Logelemző CLI és a tesztelési téma részletes áttekintése.
3. Concurrency és a backend téma HTTP/timeout része, majd endpoint checker.
4. Teljesítménymérés a meglévő megoldásokon.
5. FastAPI önálló feldolgozása, adatbázisok és a backend további részei, majd job processing API.

## Aktuális munkamenet

- Aktív feladat: még nincs.
- Következő lépés: az első tíz témakör tanulása és a megfelelő önálló feladatok megoldása; a következő kidolgozandó téma a 11. Megbízható szolgáltatások és háttérfeldolgozás.
- Elakadás: nincs rögzítve.

## Munkamenetnapló

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
