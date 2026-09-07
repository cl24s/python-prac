# Concurrency és párhuzamos végrehajtás

Állapot: kidolgozott első változat. A tananyag és az elsajátítás állapotát külön a [ROADMAP](../../../ROADMAP.md) követi.

## Tanulási sorrend

| Sorrend | Fejezet |
| --- | --- |
| 1 | [Concurrency, parallelism és a végrehajtási modell](01-concurrency-models.md) |
| 2 | [Event loop, coroutine, task és blokkoló hívások](02-event-loop-and-tasks.md) |
| 3 | [TaskGroup, gather és több feladat hibája](03-task-groups-and-failures.md) |
| 4 | [Timeout, cancellation és kooperatív leállítás](04-timeouts-and-cancellation.md) |
| 5 | [Threadek, race condition, lock és deadlock](05-threads-and-synchronization.md) |
| 6 | [Processzek, CPU-munka és spawn](06-processes-and-cpu-work.md) |
| 7 | [Korlátos párhuzamosság, backpressure és graceful shutdown](07-bounded-workers-and-shutdown.md) |

## Hogyan dolgozz vele?

1. Olvasd el a magyarázatot, és jósolj eredményt a példához.
2. Futtasd az egész kódblokkot önálló `.py` fájlként.
3. Válaszolj angolul az interjúkérdésekre, majd ellenőrizd a magyar válaszvázlattal.
4. Oldd meg a [kapcsolódó feladatsort](../../../exercises/senior-python/06-concurrency.md). A megoldások nincsenek előre mellékelve.
5. Review után a tényleges megértés és feladatteljesítés alapján frissítsük a roadmapet.

## Futtatás és korlátok

- CPython 3.12.13-on ellenőrzött, standard library-alapú példák. TaskGroup és asyncio timeout miatt legalább Python 3.11 szükséges, a tananyag célverziója 3.12.
- A processzpoolos blokkot fájlból futtasd, ne REPL-ből vagy stdinből. Explicit spawn és main guard szerepel benne.
- Async kódblokk önálló scriptként asyncio.run-t használ; már futó event loopban a main coroutine-t awaiteld.
- Az Event/Barrier a tesztsorrendet vezérli; a rövid védő timeout nem sebességígéret.
- Az asyncio.sleep(0) a példákban kooperatív átadás, nem valós I/O szimulálására használt időmérés.
- A példák helyi, kontrollált erőforrásokkal futnak. Nem bizonyítanak éles HTTP-, adatbázis-, Windows- vagy free-threaded működést.
- Az assertionök oktatási ellenőrzések; éles inputvalidáció explicit hibát adjon.
- A pontos eredmények az [ellenőrzési jegyzetben](../VALIDATION.md) szerepelnek.

## Önellenőrzés

- [ ] A sikeres és hibás végrehajtást is elmagyarázom.
- [ ] Megnevezem az erőforrások és taskok tulajdonosát.
- [ ] A cleanupot és az elmaradó mellékhatásokat is ellenőrzöm.
- [ ] Nem keverem a cancellationt a rollbackkel vagy a kényszerleállítással.

[Teljes tudástérkép](../README.md)
