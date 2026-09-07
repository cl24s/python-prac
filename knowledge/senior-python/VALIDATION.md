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
