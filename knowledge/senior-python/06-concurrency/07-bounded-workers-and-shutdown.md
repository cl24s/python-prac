# Korlátos párhuzamosság, backpressure és graceful shutdown

## Mit kell tudnod?

- Az aktív munka és a várakozó feladatok számát külön korlátozni.
- Queue-ból feldolgozott elemeket siker és hiba esetén is elszámolni.
- Leállítási sorrendet választani: drain vagy abort.

## Magyarázat

A Semaphore korlátozhatja az aktív műveleteket, de nem a már létrehozott taskok számát. Ha egymillió coroutine-ból egymillió task készül, és mind egy semaphore-ra vár, a memóriaigény továbbra is nagy lehet. Korlátos queue és fix workerszám a várakozó állományt is kézben tartja.

Backpressure esetén a termelő vár, amikor a fogyasztók nem győzik a munkát. Ez helyi folyamatban korlátos queue-val kifejezhető. Külső HTTP-beküldőnél elutasítás, lassítás vagy tartós queue is szóba jöhet; a queue nem oldja meg a tartós túlterhelést, csak explicit helyet ad a várakozásnak.

## Kódpélda: fix workerpool és szabályos drain

```python
import asyncio

async def main() -> None:
    queue: asyncio.Queue[int | None] = asyncio.Queue(maxsize=2)
    results: list[int] = []
    tasks: list[asyncio.Task[None]] = []
    active = 0
    peak = 0

    async def worker() -> None:
        nonlocal active, peak
        while True:
            item = await queue.get()
            try:
                if item is None:
                    return
                active += 1
                peak = max(peak, active)
                try:
                    await asyncio.sleep(0)
                    results.append(item * 2)
                finally:
                    active -= 1
            finally:
                queue.task_done()

    async with asyncio.TaskGroup() as group:
        tasks = [group.create_task(worker()) for _ in range(2)]
        for item in range(6):
            await queue.put(item)
        for _ in tasks:
            await queue.put(None)
        await queue.join()

    assert sorted(results) == [0, 2, 4, 6, 8, 10]
    assert 1 <= peak <= 2
    assert active == 0 and queue.empty()
    assert all(task.done() and not task.cancelled() for task in tasks)

if __name__ == '__main__':
    asyncio.run(main())
```

A `sleep(0)` kifejezett kooperatív átadás a demonstrációhoz, nem egy sebességi elvárás. A None sentinel a feladatban nem lehet érvényes munka; minden workerhez egy leállító elem tartozik. A task_done a sentinelt és minden sikeresen kivett elemet is elszámolja. A queue.join az unfinished-számlálóra vár, nem egyszerűen az ürességre.

A results lista itt a kis példa kimenetét gyűjti, ezért O(n). Nagy pipeline-ban a kimenetet is tovább kell írni vagy korlátozni; a queue-limit önmagában nem korlátozza az összes állapotot.

## Leállítási szerződés

**Drain:** új munka fogadásának leállítása → sorban álló munka feldolgozása → workerek lezárása → kliens- és egyéb erőforrások lezárása. **Abort:** taskok megszakítása → cleanup megvárása → félkész munkák sorsának rögzítése. Mindkettőhöz kell határidő és túlcsordulási döntés.

A példában egy workerhiba a TaskGroup miatt a többi task és a csoportban futó termelői várakozás megszakításához vezethet. Ilyenkor nem állítjuk, hogy a queue minden eleme feldolgozott; a join csak a normál drain út része. Tartós feladatkezelőnél ack/requeue/DLQ szabály kell, amit a későbbi háttérfeldolgozási témakör mélyít el.

## Tipikus hibák és senior szempontok

- **Hiba:** task_done hiányzik exceptionnél. **Javítás:** a sikeres get után finally blokkban elszámolás.
- **Hiba:** csak egy sentinel több workerhez. **Javítás:** egyértelmű broadcast/worker-szám szerinti leállítás.
- Python 3.12-ben a példák sentinelre építenek; későbbi queue shutdown API-k verziófüggők.
- Az eredmények befejezési sorrendje nem garantált input-sorrend. Ha erre szükség van, indexelj, de számolj a rendezőpufferrel.

## Interview questions

**Why does a semaphore not guarantee bounded total memory?**

Válaszvázlat: csak az aktív kritikus szakaszt limitálja; a várakozó taskok és az eredménygyűjtés tovább nőhet.

**What does queue.join actually wait for?**

Válaszvázlat: minden beputolt elemhez tartozó task_done elszámolásra, nem pusztán arra, hogy üres legyen a sor.

## Önellenőrzés

- [ ] A várakozó és aktív munka külön korlátos.
- [ ] Megnevezem a félkész munkák sorsát abort esetén.
- [ ] A normál leállás után minden saját worker befejezett.


## Kapcsolódó gyakorlat

[C07 – önálló feladat](../../../exercises/senior-python/06-concurrency.md#c07)

## Forrás és továbbolvasás

[asyncio queues](https://docs.python.org/3.12/library/asyncio-queue.html) · [Synchronization primitives](https://docs.python.org/3.12/library/asyncio-sync.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
