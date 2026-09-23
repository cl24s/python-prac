# Python tudástérkép

**Kezdés: [START_HERE](../../START_HERE.md). Aktuális tanulási sorrend: [ROADMAP](../../ROADMAP.md).** Ez a fájl tartalomjegyzék, nem második tanulási terv és nem elsajátítási igazolás.

A `senior-python` könyvtárnév megmaradt, hogy a régi hivatkozások működjenek. Nem jelenti azt, hogy a tanuláshoz már senior Python-tudás kell.

## Belépő: gyakorlati alapozás

[00. Gyakorlati alapozás](00-foundations/README.md) — 4 rövid fejezet a futtatásról, rövid programokról, tesztelésről és fájlkezelésről; [8 új gyakorlat](../../exercises/senior-python/00-foundations.md).

Ez ad átmenetet a szintaxis és a korábbi mélyebb fejezetek közé. Az egyszerű függvényeket, ciklusokat és teszteket a closure, dekorátor és MRO elé vesszük.

## Meglévő 14 témakör

A 97 részletes fejezet és a hozzájuk tartozó 94 feladat megmaradt. Az alábbi számozás referencia, nem kötelező haladási sorrend.

| Témakör | Fő tartalom |
| --- | --- |
| [01. Python működése és nyelvi alapok](01-python-basics/README.md) | Referenciák, másolás, függvények, iterálás; később dekorátorok, importok, memória. |
| [02. Adatszerkezetek](02-data-structures/README.md) | List, tuple, dict, set, specializált kollekciók, komplexitás, rendezés, streaming. |
| [03. OOP és objektumtervezés](03-oop/README.md) | Példányállapot, invariánsok, composition, dataclass; később öröklés, MRO, minták. |
| [04. Típusok és interfészek](04-types-and-interfaces/README.md) | Type hint, statikus ellenőrzés, runtime validáció, Protocol, ABC. |
| [05. Hibakezelés és erőforrások](05-error-handling/README.md) | Hibahatárok, kivételek, cleanup, részleges hibák. |
| [06. Concurrency](06-concurrency/README.md) | Asyncio, thread, process, cancellation, limitek, queue. |
| [07. Tesztelés és kódminőség](07-testing-and-quality/README.md) | Pytest, fixture, parametrizálás, teszthatárok, refaktorálás, review. |
| [08. HTTP és backend](08-http-and-backend/README.md) | API-szerződés, HTTP, WSGI/ASGI, auth, külső szolgáltatások. |
| [09. FastAPI](09-fastapi/README.md) | Routing, validáció, függőségek, lifespan, adatbázis, tesztek, futtatás. |
| [10. Adatbázisok](10-databases/README.md) | SQL, join, index, tranzakció, ORM, migráció, cache. |
| [11. Megbízható szolgáltatások](11-reliable-services/README.md) | Retry, idempotencia, queue, részleges kiesés, graceful shutdown. |
| [12. Teljesítmény és hibakeresés](12-performance-and-debugging/README.md) | Profiling, CPU/memória/I/O, mérés és diagnosztika. |
| [13. Csomagolás, biztonság, üzemeltetés](13-packaging-security-and-operations/README.md) | Környezet, konfiguráció, CI, megfigyelhetőség, biztonsági határok. |
| [14. Tervezés és együttműködés](14-design-and-collaboration/README.md) | Követelmények, modulhatárok, trade-offok, biztonságos változtatás, review. |

A meglévő fejezetek továbbra is működés → használati helyzet → példa → tipikus hibák → angol kérdés → gyakorlat felépítésűek. A FastAPI és az OOP önálló témakör marad, de nem kezdési feltétel.

## Későbbi bővítések

A még kidolgozandó algoritmikus minták, adatfeldolgozási és NumPy/Pandas-irány pontos hatóköre a [ROADMAP-ben](../../ROADMAP.md) található. Ezeket ne tekintsd kész leckéknek vagy kötelező előfeltételeknek.

[Korábbi ellenőrzési jegyzet](VALIDATION.md) · [Gyakorlatok](../../exercises/README.md) · [Projektek](../../projects/README.md)
