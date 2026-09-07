# Timeout, cancellation és kooperatív leállítás

## Mit kell tudnod?

- Megérteni, hogy a cancel kérés, nem azonnali kényszerleállítás.
- Cleanup után továbbengedni a megszakítást.
- Teljes határidőt és egyedi műveleti timeoutot külön kezelni.

## Magyarázat

A task cancel-je CancelledError kivétel bejuttatását kéri a coroutine-ba; az együttműködő felfüggesztési pontoknál tud érvényesülni. Hosszú blokkoló kód közben a loop nem feltétlenül jut el idáig. Ezért a timeout sem kemény, operációs rendszer által kikényszerített CPU-időkorlát.

A CancelledError BaseExceptionből származik, így az `except Exception` nem nyeli el. Ha kifejezetten elkapod, normál esetben cleanup után dobd tovább. Az elnyelés megtörheti a TaskGroup vagy timeout életciklusát. A finally jó hely a takarításra, de egy újabb cancellation a benne awaitelt cleanupot is megszakíthatja: összetett erőforrásnál erre is legyen terv.

## Kódpélda: megszakítás megvárása

```python
import asyncio

async def main() -> None:
    started = asyncio.Event()
    cleaned = asyncio.Event()

    async def worker() -> None:
        try:
            started.set()
            await asyncio.Event().wait()
        finally:
            cleaned.set()

    task = asyncio.create_task(worker())
    await started.wait()
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        pass
    else:
        raise AssertionError('Cancellation must propagate')
    assert task.cancelled()
    assert cleaned.is_set()

if __name__ == '__main__':
    asyncio.run(main())
```

A tulajdonos előbb kéri a leállítást, utána megvárja. A már szándékosan megszakított saját task CancelledError-jának kezelése itt megfelelő; maga a worker nem nyeli el.

## Kódpélda: már lejárt határidő

```python
import asyncio

async def main() -> None:
    cleaned = False
    try:
        async with asyncio.timeout_at(asyncio.get_running_loop().time() - 1):
            try:
                await asyncio.Event().wait()
            finally:
                cleaned = True
    except TimeoutError:
        pass
    else:
        raise AssertionError('The deadline has already expired')
    assert cleaned

if __name__ == '__main__':
    asyncio.run(main())
```

A timeout context saját megszakítását a kijáratán TimeoutError-rá alakítja; ezt a contexten kívül fogjuk el. A múltbeli határidő miatt nem kell rövid sleepre épülő időzítés. Határidőhöz monotonic loop-időt használj, ne faliórát, amely átállítható.

## További döntési szempontok

A wait_for normál esetben megszakítja a várt feladatot, és megvárja a cancellation lezárását; a tényleges idő hosszabb lehet a megadott timeoutnál. A teljes kéréshez egy közös deadline jobb lehet, mint minden rétegben új teljes timeout, amely többszörösen elnyújtja a munkát.

A shield megvédheti a belső taskot a külső várakozó cancellationjétől, de a külső hívó továbbra is CancelledError-t kap. Meg kell őrizni a belső task referenciáját, és gondoskodni a befejezéséről és hibájáról. Ne ezzel alakíts minden munkát felügyelet nélküli háttérfeladattá.

## Tipikus hibák és senior szempontok

- A to_thread awaitjének megszakítása nem öli meg a futó szálat. A sync kódnak saját timeout vagy stop-jelzés kell.
- Timeout után egy távoli írás eredménye bizonytalan maradhat; cancellation nem rollback.
- Leállításkor legyen határidő, és dokumentált döntés a meg nem álló munkáról. Threadet biztonságosan általánosan kényszerleállítani nem lehet.

## Interview questions

**Why might a timeout take longer than its configured duration?**

Válaszvázlat: kooperatív megszakítás, blokkolt loop, cleanup és annak megvárása.

**What should follow task.cancel()?**

Válaszvázlat: a tulajdonos várja meg a task lezárását és kezelje a megfelelő eredményt; a kérés önmagában nem bizonyít leállást.

## Önellenőrzés

- [ ] A task cleanupja ellenőrzött cancellation után.
- [ ] A TimeoutError-t a timeout contexten kívül kezelem.
- [ ] Nem állítom, hogy cancel megöl egy futó threadet.


## Kapcsolódó gyakorlat

[C04 – önálló feladat](../../../exercises/senior-python/06-concurrency.md#c04)

## Forrás és továbbolvasás

[Cancellation, timeouts and shielding](https://docs.python.org/3.12/library/asyncio-task.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
