# Event loop, coroutine, task és blokkoló hívások

## Mit kell tudnod?

- Megkülönböztetni a coroutine létrehozását annak végrehajtásától.
- Átfedő taskokat indítani és minden eredményüket megvárni.
- Felismerni, melyik hívás blokkolja az event loop szálát.

## Magyarázat

Egy `async def` függvény hívása coroutine objektumot ad. A törzs akkor fut, amikor awaitelik vagy taskként ütemezik. Egymás utáni `await first(); await second()` szekvenciális; az átfedéshez több élő task kell. Az event loop egyszerre egy task Python-kódját futtatja a saját szálán.

Az await egy awaitable eredményére vár. Nem minden await jelent tényleges felfüggesztést: egy már kész eredmény azonnal rendelkezésre állhat. Egy hosszú, await nélküli CPU-ciklus vagy sync hálózati hívás addig megakadályozhatja a többi task futását.

## Kódpélda: két task ténylegesen eljut a várakozásig

```python
import asyncio

async def main() -> None:
    release = asyncio.Event()
    started = [asyncio.Event(), asyncio.Event()]

    async def operation(index: int) -> str:
        started[index].set()
        await release.wait()
        return f'result-{index}'

    async with asyncio.TaskGroup() as group:
        first = group.create_task(operation(0))
        second = group.create_task(operation(1))
        await started[0].wait()
        await started[1].wait()
        assert not first.done() and not second.done()
        release.set()
    assert first.result() == 'result-0'
    assert second.result() == 'result-1'

if __name__ == '__main__':
    asyncio.run(main())
```

A release előtt mindkét task elindult, de egyik sem fejeződött be. A TaskGroup kijárata megvárja az összes saját taskot. Az Event csak helyi tesztszinkronizáció; élesben HTTP-válasz, socket vagy queue lehet a várakozás oka.

## Kódpélda: sync kliens kivezetése a loopból

```python
import asyncio
import threading

async def main() -> None:
    loop_thread = threading.get_ident()
    worker_thread = await asyncio.to_thread(threading.get_ident)
    assert worker_thread != loop_thread

if __name__ == '__main__':
    asyncio.run(main())
```

A to_thread külön szálban hívja a sync függvényt; nem alakítja azt natív async műveletté. A kliens threadsafety-je, saját timeoutja és megszakíthatósága továbbra is számít. Tiszta Python CPU-munka esetén a GIL miatt ez nem automatikus többmagos gyorsítás.

## Tipikus hibák és senior szempontok

- **Hiba:** `time.sleep()` async függvényben. **Javítás:** async várakozáshoz `await asyncio.sleep()`, sync I/O-hoz megfelelő adapter.
- **Hiba:** coroutine hívása await és task nélkül. **Javítás:** legyen egyértelmű tulajdonosa és végrehajtási útja.
- Futó event loopon belül ne hívj új `asyncio.run()`-t. Async környezetben a felső függvényt awaiteld.
- Fire-and-forget tasknak is kell életciklus, referencia és hibamegfigyelés; a „Task exception was never retrieved” elveszett felügyeletet jelez.
- A loopban `threading.Lock`-ra blokkolni veszélyes; az asyncio lock és queue más felhasználási körre készült, nem thread-safe eszköz.

## Interview questions

**Why do two consecutive awaits not necessarily run concurrently?**

Válaszvázlat: az első befejeződését várjuk a második elindítása előtt; külön ütemezett taskok adnak átfedő életciklust.

**Does every await yield control to another task?**

Válaszvázlat: nem; már kész awaitable esetén nem feltétlen történik felfüggesztés.

## Önellenőrzés

- [ ] A coroutine és task fogalmat külön használom.
- [ ] Minden task eredményének és hibájának van gazdája.
- [ ] Megtalálom a rejtett sync I/O-t az async kódban.


## Kapcsolódó gyakorlat

[C02 – önálló feladat](../../../exercises/senior-python/06-concurrency.md#c02)

## Forrás és továbbolvasás

[Coroutines and tasks](https://docs.python.org/3.12/library/asyncio-task.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
