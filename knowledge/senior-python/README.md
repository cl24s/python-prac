# Senior Python fejlesztő – tudástérkép

Állapot: mind a 14 témakör kidolgozott első változata elkészült (összesen 97 fejezet és 94 önálló feladat); az elsajátítás állapotát a ROADMAP vezeti.

Cél: senior Python backend/platform interjúkra felkészülni. Minden témánál tudni kell, hogyan működik, mikor használnád, milyen hibákhoz vezethet, és hogyan ellenőriznéd.

## Elkészült témakörök

- [1. Python működése és nyelvi alapok](01-python-basics/README.md) — 8 fejezet, 6 önálló feladat.
- [2. Adatszerkezetek és algoritmikus gondolkodás](02-data-structures/README.md) — 6 fejezet, 6 önálló feladat.

- [3. OOP és objektumtervezés](03-oop/README.md) — 7 fejezet, 6 önálló feladat.
- [4. Típusok és interfészek](04-types-and-interfaces/README.md) — 6 fejezet, 6 önálló feladat.

- [5. Hibakezelés és erőforrás-kezelés](05-error-handling/README.md) — 5 fejezet, 5 önálló feladat.
- [6. Concurrency és párhuzamos végrehajtás](06-concurrency/README.md) — 7 fejezet, 7 önálló feladat.

- [7. Tesztelés és kódminőség](07-testing-and-quality/README.md) — 6 fejezet, 6 önálló feladat.
- [8. HTTP és backend alapok](08-http-and-backend/README.md) — 7 fejezet, 7 önálló feladat.

- [9. FastAPI](09-fastapi/README.md) — 11 fejezet, 11 önálló feladat.
- [10. Adatbázisok és adatkezelés](10-databases/README.md) — 7 fejezet, 7 önálló feladat.

- [11. Megbízható szolgáltatások és háttérfeldolgozás](11-reliable-services/README.md) — 7 fejezet, 7 önálló feladat.
- [12. Teljesítmény és hibakeresés](12-performance-and-debugging/README.md) — 6 fejezet, 6 önálló feladat.

- [13. Csomagolás, biztonság és üzemeltetés](13-packaging-security-and-operations/README.md) — 7 fejezet, 7 önálló feladat.
- [14. Senior szintű tervezés és együttműködés](14-design-and-collaboration/README.md) — 7 fejezet, 7 önálló feladat.

[A példák ellenőrzési jegyzete](VALIDATION.md)

## 1. Python működése és nyelvi alapok

- Mutability, referenciák, objektumazonosság: `is` vs. `==`.
- Mutable default argument, shallow copy vs. deep copy.
- Scope, closure, late binding, `global`, `nonlocal`.
- Iterable, iterator, generator és lazy feldolgozás.
- Comprehensionök, unpacking, `*args`, `**kwargs`.
- Decoratorok és context managerek működése, saját implementációjuk.
- Fontos speciális metódusok: `__eq__`, `__hash__`, `__iter__`, `__enter__`, `__exit__`.
- Importok, modulok, package-ek és körkörös függőségek.
- Memóriakezelés alapjai: referenciaszámlálás, ciklusok és garbage collection.

## 2. Adatszerkezetek és algoritmikus gondolkodás

- `list`, `tuple`, `dict`, `set`: mikor melyiket választod.
- `deque`, `Counter`, `defaultdict` és heap használati helyzetei.
- Hash-elhetőség és hash-alapú kollekciók működése.
- Gyakori műveletek idő- és memóriaigénye.
- Rendezés, keresés, csoportosítás és deduplikáció.
- Nagy adathalmazok feldolgozása teljes memóriába olvasás nélkül.

## 3. OOP és objektumtervezés — kiemelt témakör

- Osztályok, példányok, példány- és osztályattribútumok.
- Instance method, `classmethod`, `staticmethod`: mikor melyik indokolt.
- Encapsulation, property-k, invariánsok és érvényes objektumállapot.
- Öröklődés, overriding, `super()` és method resolution order.
- Composition vs. inheritance; merev öröklődési hierarchiák felismerése.
- Polimorfizmus, duck typing, `Protocol` és ABC.
- `dataclass` vs. viselkedést is tartalmazó domainobjektum.
- Objektumazonosság, értékegyenlőség és hash-elhetőség.
- Dependency injection és könnyen tesztelhető objektumok.
- SOLID elvek konkrét Python-példákon; hasznuk és a túltervezés veszélye.
- Strategy, Adapter, Factory, Repository: használati helyzetek és kompromisszumok.
- Mikor elég egy függvény vagy modul, és mikor indokolt osztály.
- Gyakorlati irány: rosszul felépített osztályhalmaz refaktorálása; felelősségek és függőségek megindoklása.

## 4. Típusok és interfészek

- Type hint-ek olvasása és következetes használata.
- `Any`, `object`, unionök, opcionális értékek és generikus típusok.
- `Protocol` vs. ABC; strukturális és névleges típusosság.
- `dataclass`, egyszerű osztály és validált adatmodell közötti választás.
- Statikus típusellenőrzés és runtime validáció különbsége.
- Érthető interfészek tervezése túlzott absztrakció nélkül.
- Kapcsolódás az OOP témához: az objektumtervezési döntések típusokkal való kifejezése.

## 5. Hibakezelés és erőforrás-kezelés

- Mikor dobj exceptiont, és hol kezeld.
- Saját exceptiontípusok és exception chaining.
- Üzleti hiba, hibás bemenet és infrastruktúrahiba megkülönböztetése.
- Fájlok, kapcsolatok és lockok megbízható lezárása.
- Részleges hibák kezelése; hibák elnyelésének veszélyei.
- Használható hibajelzések és kontextust megőrző naplózás.

## 6. Concurrency és párhuzamos végrehajtás

- Concurrency vs. parallelism; CPU-bound vs. I/O-bound munka.
- Mikor `asyncio`, thread vagy process a megfelelő választás.
- GIL: mit korlátoz, mit nem; a Python-implementáció és build szerepe.
- Event loop, coroutine, task és `await`.
- Blokkoló hívások felismerése async kódban.
- Timeout, cancellation és a feladatok életciklusa.
- Race condition, deadlock, lock, semaphore és queue.
- Korlátozott párhuzamosság, backpressure és rendezett leállítás.

## 7. Tesztelés és kódminőség

- Unit-, integrációs és end-to-end tesztek megfelelő határai.
- `pytest`: fixture-ök, parametrizálás és exceptiontesztelés.
- Mikor mockolj, és mikor használj valódi komponenst vagy fake-et.
- Async kód, időfüggő működés és hibás végrehajtási utak tesztelése.
- Determinisztikus tesztek; flaky tesztek okai.
- Refaktorálás viselkedésváltozás nélkül.
- Code review: helyesség, olvashatóság, összetettség és tesztelhetőség.

## 8. HTTP és backend alapok

- HTTP-metódusok, státuszkódok, headerek és idempotencia.
- Request-életciklus, middleware, validáció és hibaválaszok.
- WSGI és ASGI szerepe; alkalmazás, szerver és reverse proxy kapcsolata.
- Authentication vs. authorization.
- Lapozás, API-verziózás és visszafelé kompatibilitás.
- Connection pooling, timeoutok és külső szolgáltatások kezelése.
- A választott, mélyen feldolgozandó framework: FastAPI.

## 9. FastAPI — kötelező, önálló témakör

- Routing, path/query paraméterek, request- és response-modellek.
- Pydantic: validáció, szerializáció, bejövő és kimenő adatmodellek.
- Dependency injection: `Depends`, függőségláncok és erőforrás-életciklus.
- `async def` vs. `def`; blokkoló műveletek és konkurens végrehajtás.
- Alkalmazásindítás és leállítás, lifespan.
- Hibakezelés, egységes hibaválaszok és middleware.
- Authentication és authorization.
- Adatbázis-integráció: sessionök, tranzakciók és connection pool.
- Háttérfeladatok: mi maradhat az alkalmazáson belül, mikor kell külön worker.
- Tesztelés: dependency override, HTTP-tesztek, adatbázis és külső szolgáltatások.
- OpenAPI és az API-szerződés tudatos kialakítása.
- Éles futtatás: ASGI-szerver, workerek, timeoutok, logging és graceful shutdown.

## 10. Adatbázisok és adatkezelés

- SQL: joinok, aggregációk, indexek és lekérdezési tervek.
- Tranzakciók, izoláció, lockolás és konkurens módosítások.
- ORM működése; N+1 lekérdezések és rejtett adatbázis-műveletek.
- Sémamigrációk és kompatibilis adatmodell-változtatások.
- Cache használata, invalidálás és konzisztencia.
- Adatbázis-kapcsolatok és tranzakcióhatárok kezelése.

## 11. Megbízható szolgáltatások és háttérfeldolgozás

- Retry: mikor biztonságos, milyen limittel, backoffal és jitterrel.
- Idempotencia és duplikált üzenetek kezelése.
- Queue-k, workerek, acknowledgement és dead-letter queue.
- Mi történik, ha a folyamat két művelet között leáll.
- Külső függőségek lassulása, túlterhelés és részleges kiesés.
- Graceful shutdown: folyamatban lévő kérések és feladatok sorsa.

## 12. Teljesítmény és hibakeresés

- Mérés optimalizálás előtt; a valódi szűk keresztmetszet azonosítása.
- CPU-, memória-, I/O- és adatbázisproblémák megkülönböztetése.
- Profiling és reprezentatív benchmarkok.
- Memóriaszivárgás, túlzott objektumképzés és korlátlan cache felismerése.
- Lassú vagy lefagyó alkalmazás módszeres vizsgálata.
- Áteresztőképesség, válaszidő és erőforrásigény közötti kompromisszumok.

## 13. Csomagolás, biztonság és üzemeltetés

- Virtuális környezetek, `pyproject.toml`, függőségek és reprodukálható telepítés.
- Konfiguráció és secret-kezelés.
- Strukturált logok, metrikák, tracing és health checkek.
- CI: tesztelés, lintelés, típusellenőrzés és csomagépítés.
- Konténeres futtatás, processzek/workerek száma és erőforráskorlátok.
- SQL injection, command injection, veszélyes deszerializáció és SSRF felismerése.
- Érzékeny adatok védelme a naplókban és hibaválaszokban.

## 14. Senior szintű tervezés és együttműködés

- Követelmények és hibaforgatókönyvek tisztázása kódolás előtt.
- Modulhatárok, loose coupling, cohesion és függőségi irányok.
- Egyszerű megoldás választása, a későbbi bővítés helyének felismerése.
- Technikai döntések és kompromisszumok világos magyarázata.
- Meglévő rendszer biztonságos továbbfejlesztése és migrálása.
- Mások kódjának megértése, review és mentorálás.
- Saját éles példák: probléma → döntés → eredmény → tanulság.

## Tanulási sorrend és projektkapcsolat

- Alapozás: Python alapok és adatszerkezetek → OOP → típusok és interfészek → hibakezelés és tesztelés.
- Backend: concurrency alapok és HTTP → FastAPI → adatbázisok és háttérfeldolgozás.
- Elmélyítés: teljesítmény, biztonság, üzemeltetés és tervezési döntések a projektekben.
- Az OOP és FastAPI közös gyakorlótere a [job processing API](../../projects/03-job-processing-api/README.md).
- A FastAPI-réteg a HTTP-t kezeli; az alkalmazáslogika a felhasználási eseteket.
- A domainobjektumok az üzleti szabályokat őrzik, ahol ez indokolt.
- A tárolás és háttérfeldolgozás jól meghatározott interfészeken kapcsolódik.
- A lényegi logika HTTP-kérés nélkül is tesztelhető.

## Kidolgozási elv és következő lépés

- Ez a fájl a teljes tervezett tartalom áttekintése; nem elsajátítási igazolás.
- A knowledge/ alatt kizárólag a senior-python/ tananyagstruktúrát használjuk; a részletes fejezetek itt találhatók, külön OOP- és FastAPI-résszel.
- Minden fejezet: működés → használati helyzet → példa → tipikus hibák → angol interjúkérdések → gyakorlat.
- Következő lépés: a tanulás indítása az 1. Python alapok fejezeteivel és a B01 önálló feladattal; review után rögzítjük az elsajátítást. A teljes tartalom első változata elkészült.
- Nem cél minden framework, metaclass-trükk vagy a teljes standard library fejből ismerete.

[Vissza a repó áttekintéséhez](../../README.md) · [Haladáskövetés](../../ROADMAP.md)
