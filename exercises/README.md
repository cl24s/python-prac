# Gyakorlatok

**Aktuális feladat és állapot: [ROADMAP](../ROADMAP.md). Kezdés: [START_HERE](../START_HERE.md).** A tanulás belépési pontja egy kész feladatkiírás, az elmélet a megoldás közben használható referencia.

## Aktuális gyakorlati feladatsor — Feladatkezelő

Egy kis program három egymásra épülő változatban. Kész követelmények, mintaadatok, elvárt eredmények és elfogadási feltételek; kész implementáció nélkül.

| Kiírás | Fókusz |
| --- | --- |
| [TM01 — Típusok és validáció](task-manager/01-types-and-validation.md) | String, int, bool, None, függvények, bemenetellenőrzés és új rekord. |
| [TM02 — Adatszerkezetek](task-manager/02-data-structures.md) | List, dict, set, keresés, rendezés, összesítés és referenciák. |
| [TM03 — OOP](task-manager/03-oop.md) | Task, TaskManager, példányállapot, metódusok és composition. |

[Feladatkezelő áttekintése és saját fájlok helye](task-manager/README.md). Az első konkrét rész a TM01 normalize_title függvénye. FastAPI ugyanennek a programnak későbbi bővítése lehet, nem mostani előfeltétel.

## Meglévő feladatbank — célzott kiegészítés

Ezek megmaradtak, de nem második kötelező útvonal. Akkor válasszunk belőlük, ha egy konkrét fogalmat külön kell gyakorolni.

- [Gyakorlati alapozás — L01–L08](senior-python/00-foundations.md)
- [Nyelvi alapok — B01–B06](senior-python/01-python-basics.md)
- [Adatszerkezetek — D01–D06](senior-python/02-data-structures.md)
- [OOP — O01–O06](senior-python/03-oop.md)
- [Típusok és interfészek — T01–T06](senior-python/04-types-and-interfaces.md)
- [Hibakezelés — E01–E05](senior-python/05-error-handling.md)
- [Concurrency — C01–C07](senior-python/06-concurrency.md)
- [Tesztelés — Q01–Q06](senior-python/07-testing-and-quality.md)
- [HTTP és backend — H01–H07](senior-python/08-http-and-backend.md)
- [FastAPI — F01–F11](senior-python/09-fastapi.md)
- [Adatbázisok — DB01–DB07](senior-python/10-databases.md)
- [Megbízhatóság — R01–R07](senior-python/11-reliable-services.md)
- [Teljesítmény — P01–P06](senior-python/12-performance-and-debugging.md)
- [Csomagolás és üzemeltetés — OP01–OP07](senior-python/13-packaging-security-and-operations.md)
- [Tervezés — SD01–SD07](senior-python/14-design-and-collaboration.md)

## Saját megoldások

A feladatkezelő kódját az `exercises/task-manager/work/` mappában bővíted. A javasolt fájlokat csak a tényleges munka kezdetén hozd létre. A régi önálló feladatok saját almappái is megmaradnak: például `senior-python/02-data-structures/d01/`, illetve `senior-python/00-foundations/l01/`.

Az első rövid feladatokat saját assert-ellenőrzések is kísérhetik; pytestre fokozatosan térünk át. Nem kell minden feladathoz package, container, CI vagy DESIGN.md. Kész megoldás alapból nincs mellékelve.

Rögzítsd: mi ment önállóan, mihez kellett dokumentáció vagy segítség, mely teszteket futtattad és mi maradt bizonytalan. Egy későbbi, új változat külön ellenőrzés, nem a régi kód újramásolása.

## Régi ötletvázak — nem az aktuális útvonal

Megőrzött korai vázak: [nyelvi alapok](01-language-basics/README.md), [absztrakciók](02-abstractions/README.md), [hibakezelés](03-error-handling/README.md), [concurrency](04-concurrency/README.md), [tesztelés](05-testing/README.md), [teljesítmény](06-performance/README.md), [backend](07-backend-operations/README.md). Ezekhez új munka előtt konkrét szerződés kell; nem második, párhuzamos tananyag.

[Feladatkiírás-sablon](TEMPLATE.md)
