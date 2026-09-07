# Ellenőrzési jegyzetek

## OOP és típusok – 2026-09-06

Dátum: 2026-09-06. Hatókör: a 3. és 4. témakör jelenlegi első változata.

### Környezet

- CPython 3.12.13.
- mypy 2.3.1, `--strict --no-incremental --show-error-codes`.
- Pydantic 2.13.4 az egyetlen Pydantic-példához.

### Eredmények

| Ellenőrzés | Eredmény |
| --- | --- |
| Önálló Python-blokkok futtatása | 21/21 sikeres |
| A 4. témakör statikus ellenőrzése | 13 blokk ellenőrizve |
| Pozitív típusos minták | 11/11 hiba nélkül |
| Szándékosan hibás típusos minták | 2/2 a várt hibakategóriával |

Az értékadási negatív minta `assignment`, a listavarianciás negatív minta `arg-type` hibát ad. Ezek runtime is futnak, hogy a statikus és dinamikus viselkedés különbsége látható legyen.

### Mit jelent ez?

- A példák önálló folyamatban futottak, a saját assertionjeikkel; nincsen közöttük rejtett futási állapot.
- Az OOP-példákra runtime ellenőrzés történt; a mypy-ellenőrzés a 4. témakör kódblokkjaira vonatkozik.
- A feladatoknak kiírása van, megoldásuk nincs; felhasználói feladatteljesítést ez a jegyzet nem igazol.
- A Pydantic-példa a fenti verzióban ellenőrzött. A többi példa nem igényel külső runtime csomagot.
- A típusellenőrzés nem bizonyít minden üzleti szabályt, adatbázis-garanciát vagy konkurens viselkedést.
- Az első két témakör korábbi 29 runtime-példájának ellenőrzése az előző munkamenethez tartozik.

[OOP](03-oop/README.md) · [Típusok és interfészek](04-types-and-interfaces/README.md) · [Tudástérkép](README.md)

## Hibakezelés és concurrency – 2026-09-07

Hatókör: az 5. és 6. témakör 12 fejezete.

### Környezet és eredmények

- CPython 3.12.13, Linux, standard library; új külső függőség nélkül.
- **16/16 Python-kódblokk sikeresen lefutott**, minden blokk külön ideiglenes `.py` fájlban és önálló folyamatban, saját assertionjeivel.
- `PYTHONASYNCIODEBUG=1` és `-W error` aktív volt; a futtató blokkonként 20 másodperces felső korlátot alkalmazott.
- Az async példák ellenőrzik a taskok együttfutását, a TaskGroup hibaterjedését, a cancellation utáni cleanupot és a queue sikeres kiürítését.
- A threadpéldák determinisztikusan előidézett elvesző módosítást, lockkal védett számlálót és a futó thread asyncio future-től független életciklusát ellenőrzik.
- A processzpoolos példa explicit `spawn` móddal, fájlból, main guarddal futott; sikeres eredményt és a worker kivételének továbbítását is ellenőrzi.

### Határok

- Ez a futtatás az új 16 blokkra vonatkozott; a korábbi 50 példát ebben a munkamenetben nem futtattuk újra.
- Az új témakörökön külön statikus típusellenőrzés nem történt.
- A tesztek helyi, kontrollált hibaforgatókönyveket vizsgálnak; nem bizonyítanak éles szolgáltatási, tartóssági vagy teljesítménygaranciát.
- Windows és free-threaded CPython nem volt tesztelve. A Linuxon végzett explicit spawn-ellenőrzés nem helyettesít platformonkénti ellenőrzést.
- A 12 új önálló feladat csak kiírás; megoldás és felhasználói teljesítés nincs hozzá rögzítve.

[Hibakezelés](05-error-handling/README.md) · [Concurrency](06-concurrency/README.md) · [Tudástérkép](README.md)

## Tesztelés és HTTP/backend – 2026-09-07

Hatókör: a 7. és 8. témakör 13 fejezete. A két HTTP-tervezési fejezet nem tartalmaz futtatható Python-blokkot.

### Környezet és eredmények

- CPython 3.12.13, Linux; külön virtuális környezet.
- pytest 9.1.1 és HTTPX 0.28.1.
- **11/11 teljes Python-kódblokk sikeresen ellenőrizve.**
- A 7. témakör 6 tesztmodulját valódi pytest-futtatás vizsgálta: **24/24 teszteset sikeres** (fejezetenként 2 + 8 + 3 + 2 + 7 + 2).
- A 8. témakör 5 scriptje külön folyamatban futott, saját assertionjeivel.
- Minden blokk önálló ideiglenes fájlt és külön futtatási folyamatot kapott; blokkonként 20 másodperces felső futtatási korlát.
- `-W error`, `PYTHONASYNCIODEBUG=1` és a pytest-pluginok automatikus betöltését kikapcsoló `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1` aktív volt.
- Az első fájlpélda Markdownba írásakor keletkezett sortörés-escape hibát a pytest collection kimutatta; javítás után a teljes új példakészlet sikeresen lefutott.

### Mit ellenőriztünk?

- Határértékek, exceptionök, valódi ideiglenes fájl olvasása, mockolt mellékhatás és annak elmaradása, exception chaining.
- Async cancellation utáni cleanup, várakozás nélküli időhatárteszt, refaktorálás ismert bemeneti szerződése.
- Problem Details hibafordítás, in-process WSGI/ASGI válasz, tenant-határos hozzáférési policy, összetett kulcsos lapozás.
- HTTP-kliensadapter sikeres válasza, injektált timeoutja, hibastátusza, sémája és klienslezárása.

### Határok

- A HTTPX WSGI/ASGI és mock transportjai nem tesztelik a valódi DNS-t, TLS-t, proxyt, socketeket, hálózati timeoutot vagy pool-kimerülést.
- A lifespan, valódi tokenhitelesítés, adatbázis-atomikusság és terhelés nem volt vizsgálva. A fejezetekben szereplő policy és lapozás kontrollált oktatási modell.
- Az új anyagon nem futott mypy, linter, coverage vagy mutation testing; ezek tananyagként szerepelnek, nem elvégzett ellenőrzésként.
- A korábbi 66 kódblokk ellenőrzését nem ismételtük meg ebben a munkamenetben.
- A 13 önálló feladat kiírás; nincs kész megoldás vagy igazolt felhasználói teljesítés. A feladatban kért SQLite-integráció még nem készült el.

[Tesztelés](07-testing-and-quality/README.md) · [HTTP és backend](08-http-and-backend/README.md) · [Tudástérkép](README.md)

## FastAPI és adatbázisok – 2026-09-07

Hatókör: 18 új fejezet; a deployment-fejezet tervezési anyag, kódblokk nélkül.

### Környezet és eredmény

- CPython 3.12.13, Linux, SQLite 3.53.1.
- fastapi 0.141.1, starlette 1.6.0, pydantic 2.13.5, sqlalchemy 2.0.52, pytest 9.1.1, httpx 0.28.1, httpx2 2.12.0, anyio 4.15.1.
- **17/17 önálló Python-kódblokk, összesen 18/18 sikeres pytest-teszteset.**
- A teljes blokkok külön ideiglenes fájlból és folyamatból futottak, blokkonként 20 másodperces felső korláttal.
- `PYTHONASYNCIODEBUG=1`, `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1`, a RuntimeWarning és ResourceWarning hibának számított.
- A Starlette TestClienthez a környezetbe felkerült a httpx2; a korábbi httpx-alapú fallback deprecation warningja így megszűnt.
- A Starlette 1.6.0 belső `anyio.abc.BlockingPortal` alias-használata AnyIO 4.15.1 mellett deprecation warningot ad. Ezt az első, minden warningot hibának tekintő próbafuttatás azonosította. A végső futtatás az általános deprecation warningokat nem kezelte tesztbukásként; a RuntimeWarning és ResourceWarning kategóriát továbbra is hibának tekintette.

### Mit igazol a futtatás?

- FastAPI routing, validáció, response filtering, célzott OpenAPI-szerződés.
- Pydantic strict értékek, hiányzó/null mező, dependency cache és cleanup.
- Explicit thread-offload helye, lifespan kliensnyitás/lezárás, kezelt hibák request ID-ja.
- Demonstrációs bearer hitelesítési határ és objektumpolicy, dependency override visszaállítása, in-process background callback.
- Valódi SQLite + SQLAlchemy HTTP-integráció: egyediség, sikeres commit és konfliktus után új kérés.
- SQL LEFT JOIN, paraméterezés, helyi indexterv, részleges hiba rollbackje, elavult verzió elutasítása.
- ORM N+1: a rögzített három owneres mintán 4 SELECT helyett 2, friss sessionökben.
- Ismételhető backfill és késői régi író esete; lokális cache TTL/tenant/invalidation szabályai.

### Korlátok

- Nem futott PostgreSQL-szerver, Redis, Alembic-migráció, valódi JWT-verifier vagy éles ASGI-deployment.
- A SQLite stale-version teszt kontrollált írássorrend, nem valódi párhuzamos PostgreSQL lock/serialization teszt.
- A tesztek nem bizonyítják hálózati poolok működését, terhelési kapacitást, crash recoveryt vagy tartós üzenetfeldolgozást.
- Nem futott mypy, linter, coverage vagy teljesítménybenchmark. A korábbi 77 példát nem futtattuk újra.
- A 18 önálló feladatnak kiírása készült; nincs kész megoldás vagy igazolt felhasználói teljesítés.

[FastAPI](09-fastapi/README.md) · [Adatbázisok](10-databases/README.md) · [Tudástérkép](README.md)

## Megbízhatóság, teljesítmény és hibakeresés – 2026-09-07

Hatókör: a 11–12. témakör 13 új fejezete.

### Környezet és eredmények

- CPython 3.12.13, Linux; pytest 9.1.1, SQLite 3.53.1.
- **13/13 önálló Python-kódblokk, 19/19 sikeres pytest-teszteset.** A 11. témakör 12, a 12. témakör 7 tesztesetet tartalmaz.
- Külön ideiglenes fájl és folyamat minden blokkhoz, 20 másodperces felső korlát; `-W error`, `PYTHONASYNCIODEBUG=1`, `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1` aktív.
- Nem szükséges új futási függőség: a minták standard libraryt, a tesztek pytestet használnak.

### Ellenőrzött viselkedés

- Fogyó közös deadline, jitter és próbálkozási limit, utolsó próba utáni retry elutasítása.
- SQLite-deduplikáció újrakapcsolódás után, eltérő tartalom konfliktusa és marker előtti hibából rollback.
- Ack/retry/dead-letter policy; outbox publikálás utáni hibából szándékos duplikáció; soros breaker cooldown és próba.
- Worker befejezés és commit előtti cancellation, cleanup és ack külön kezelése.
- Explicit nearest-rank percentilis, mentett/visszaolvasott cProfile-fájl és hívásszám.
- Tracemalloc relatív peak összehasonlítás a teljes buffer és stream között.
- SQLite SELECT-szám 4-ről 1-re csökkenése a rögzített adaton, megőrzött sorrenddel és ismétléssel.
- Feloldható threadvárakozás faulthandler-dumpja és async task introspection; nincs hátrahagyott segédthread/task.
- Valódi, kisméretű timeit-futtatás: három ismétlés, ismétlésenként öt hívás; a set előfeldolgozása beleszámít.

### Helyi timeit-minták

Az alábbi értékek a dokumentum példájának egy ellenőrzési futásából származó **másodperc/hívás** adatok. Nem éles kapacitásmérés, nem hordozható sebességígéret. A futtató környezet CPU-kapacitása nincs benchmarkcélra rögzítve.

| Változat | 1. ismétlés | 2. ismétlés | 3. ismétlés |
| --- | --- | --- | --- |
| list | 0.002452299 | 0.002454152 | 0.002434000 |
| set | 0.000035470 | 0.000033565 | 0.000033461 |

### Korlátok

- RabbitMQ, Celery worker, hálózati brokerack és DLQ-konfiguráció nem futott. A policy modellek nem brokerintegrációs tesztek.
- Az injektált exception és az SQLite újrakapcsolódás nem processz-/gépkiesés, disk-loss vagy elosztott tranzakció ellenőrzése.
- A breaker soros modell; több relay, konkurens half-open és fencing csak tervezési anyag.
- A memóriaadat Python-allokációs peak, nem teljes RSS vagy natív memória. Nincs leakbizonyítás az éles alkalmazásra.
- A profiler és microbenchmark nem load test. Nem mértünk ASGI request/sec kapacitást, éles p99-et, pool-kimerülést vagy többgépes skálázást.
- Nem futott mypy, linter vagy coverage. A korábbi 94 kódblokkot nem futtattuk újra.
- A 13 új önálló feladatnak kiírása van; nincs kész megoldás vagy igazolt felhasználói teljesítés.

[Megbízhatóság](11-reliable-services/README.md) · [Teljesítmény](12-performance-and-debugging/README.md) · [Tudástérkép](README.md)

## Csomagolás, biztonság, üzemeltetés és senior tervezés – 2026-09-07

Hatókör: a 13–14. témakör 14 új fejezete és 14 önálló feladatkiírása.

### Környezet és eredmények

- CPython 3.12.13, Linux; pytest 9.1.1, SQLite 3.53.1.
- **8/8 Python-blokk és 27/27 pytest-teszteset sikeres.** A 13. témakörben 6 blokk / 17 eset, a 14.-ben 2 blokk / 10 eset.
- Minden blokk külön ideiglenes fájlban, külön folyamatban futott, 20 másodperces korláttal. `-W error`, `PYTHONASYNCIODEBUG=1`, `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1` aktív volt.
- A példákhoz standard library és a már használt pytest elegendő; új csomag telepítése nem kellett.

### Ellenőrzött viselkedés

- A pyproject-minta TOML-ként beolvasható, az entry point és src-keresési beállítás megfelel az oktatási szerződésnek.
- Settings: helyes konverzió, secret nélküli repr, hiányzó secret és hat hibás timeout elutasítása, köztük NaN és infinity.
- JSON logesemény: explicit mezők, fizikai sortörés escape-elése és nyers útvonal elutasítása.
- Connection-budget: normál és surge állapot, valamint hibás poolméret.
- Valódi SQLite-paraméterezés és külön, veszélytelen Python-subprocess: a shellkarakterek adatargumentumok maradnak.
- Publikus hibaválasz: belső részlet és ismeretlen belső kód nem kerül a válaszba.
- Porton át tesztelt use case: tulajdonos olvashat, idegen tenant és hiányzó job azonos kivételkategória.
- Üzenetkompatibilitás: három támogatott bemenet és négy hibás/ismert szabályt sértő eset.

### Ellenőrzési határok

- A TOML-teszt nem épített wheelt vagy sdistet; a build, artifact smoke teszt és CI a későbbi önálló feladat része.
- Nem futott lint, mypy, coverage, Docker, Kubernetes, valódi deployment, collector, secret manager vagy terhelésvizsgálat.
- SSRF-hez fenyegetési modell készült, nem teljes hálózati kliens vagy valódi hálózati biztonsági teszt.
- A migrációs minta parserkompatibilitást tesztel, nem DB-backfillt, többverziós rolloutot vagy crash recoveryt.
- A portpélda fake tárolót használ; nem igazol DB-versenykezelést vagy teljes hitelesítést.
- Windows, eltérő Python-verzió és egyéb operációs rendszer nem volt futtatással ellenőrizve.
- A korábbi 107 blokkot nem futtattuk újra. Az összesített 115 ellenőrzött blokk a külön munkamenetek eredménye.
- A kód nélküli fejezetek és feladatok a terv eredeti bullet pontjaihoz, a belső linkekhez és a roadmaphez ellenőrzöttek; nincs hozzájuk automatikus teszttel igazolt „jó architektúra”.
- A feladatoknak nincs kész megoldása vagy igazolt felhasználói teljesítése. Az elsajátítás továbbra is tervezett.

[Csomagolás és üzemeltetés](13-packaging-security-and-operations/README.md) · [Senior tervezés](14-design-and-collaboration/README.md) · [Tudástérkép](README.md)
