# Async kód, idő és determinisztikus tesztek

## Mit kell tudnod?

- Eseménnyel vezérelni a sorrendet önkényes sleep helyett.
- Külön ellenőrizni a megszakítást és a takarítást.
- Az időfüggő szabályt befecskendezett órával tesztelni.

## Magyarázat

Egy sleep(0.1) utáni assertion azt feltételezi, hogy a gép elég gyors volt. Egy Event azt mondja ki, hogy a szükséges állapot bekövetkezett. A tesztben használt felső timeout a beragadást korlátozza; nem a termék sebességének bizonyítéka.

Az alábbi szinkron pytest-teszt asyncio.run-nal hoz létre saját event loopot. Így nem kell async pytest-plugin. Ha a projekt később async fixture-öket és plugin által kezelt loopot használ, annak lifecycle-szabályait kell követni; már futó loopon belül nem hívható asyncio.run.

## Kódpélda: ismert állapot után cancel, majd cleanup

```python
import asyncio
from dataclasses import dataclass
import pytest

def test_cancel_waits_for_cleanup():
    async def scenario():
        started = asyncio.Event()
        cleaned = asyncio.Event()

        async def worker():
            try:
                started.set()
                await asyncio.Event().wait()
            finally:
                cleaned.set()

        task = asyncio.create_task(worker())
        try:
            await asyncio.wait_for(started.wait(), timeout=2)
            task.cancel()
            with pytest.raises(asyncio.CancelledError):
                await task
            assert cleaned.is_set()
        finally:
            if not task.done():
                task.cancel()
                await asyncio.gather(task, return_exceptions=True)

    asyncio.run(scenario())

@dataclass
class FakeClock:
    now: float = 0.0
    def __call__(self):
        return self.now

def is_expired(deadline, clock):
    return clock() >= deadline

def test_deadline_boundary_without_waiting():
    clock = FakeClock(9.9)
    assert not is_expired(10.0, clock)
    clock.now = 10.0
    assert is_expired(10.0, clock)
```

Az órás teszt a saját összehasonlításunkat vizsgálja; nem állítja át az asyncio belső óráját. Az event loop timeoutjának ellenőrzéséhez annak saját API-ját használd, ahogy a [cancellation-fejezet](../06-concurrency/04-timeouts-and-cancellation.md) mutatja.

## Flaky hibák felderítése

Ha a teszt néha bukik, rögzítsd a seedet, az időzónát, a függőség verzióját és a futási sorrendet, amennyiben relevánsak. Keress megosztott adatbázis-rekordot, globális cache-t, tesztek közt élő taskot, fix portot vagy időmérésre épített elvárást. Az automatikus újrafuttatás elfedheti a hibát; az okot kell megszüntetni.

A cancellationt ellenőrző teszt saját hibás ága is takarítson. Egy elfelejtett task a következő tesztben okozhat látszólag összefüggéstelen hibát. A virtuális órával gyorsított üzleti teszt mellé akkor kell valódi időzítési integrációs teszt, ha az időzítő adapter bekötése is kockázat.

## Interview questions

**How would you make an asynchronous test deterministic?**

Válaszvázlat: Explicit Event/Barrier és ellenőrzött életciklus; sleep helyett állapotátmenetre várok, a timeout csak felső védőkorlát.

**Does a fake clock test the event loop timeout implementation?**

Válaszvázlat: Nem. Csak a befecskendezett órát használó saját logikát ellenőrzi.

## Önellenőrzés

- [ ] A teszt hibája után sem marad gazdátlan task.
- [ ] Nem azonosítom a faliórát a monoton órával.

## Kapcsolódó gyakorlat

[Q04 – önálló feladat](../../../exercises/senior-python/07-testing-and-quality.md#q04)

## Forrás és továbbolvasás

[asyncio tasks](https://docs.python.org/3.12/library/asyncio-task.html)

[pytest flaky tests](https://docs.pytest.org/en/stable/explanation/flaky.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
