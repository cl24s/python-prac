# Worker-életciklus, drain és megszakítás

## Mit kell tudnod?

- Leállításkor az új munka és a folyamatban lévő munka sorsát külön kezelni.
- Sikeres ackot a tartós befejezéshez kötni.
- Cleanupot és üzleti rollbacket megkülönböztetni.

## Magyarázat

Graceful shutdown első lépése, hogy a worker ne vegyen fel új munkát. Az aktív feladatoknak befejezési időkeretet adsz. Ha az elfogy, kooperatív megszakítás következhet; a még be nem fejezett üzenet ne kapjon sikeres ackot csak azért, mert a finally blokk lefutott.

Drain a folyamatban lévő munka rendezett befejezése. Abort a befejezés megszakítása. A broker újrakézbesítése attól függ, milyen acknowledgement/lease protokollt és beállítást használsz. A processz memóriájában lévő queue nem tartós tároló.

## Kódpélda: siker esetén ack, cancel esetén csak cleanup

```python
import asyncio
import pytest

@pytest.mark.parametrize('finish', [True, False])
def test_worker_completion_or_cancel(finish):
    async def scenario():
        started, release, cleaned = asyncio.Event(), asyncio.Event(), asyncio.Event()
        committed, acknowledged = [], []
        async def worker():
            try:
                started.set()
                await release.wait()
                committed.append('job-1')
                acknowledged.append('job-1')
            finally:
                cleaned.set()
        task = asyncio.create_task(worker())
        try:
            await asyncio.wait_for(started.wait(), timeout=2)
            if finish:
                release.set()
                await asyncio.wait_for(task, timeout=2)
            else:
                task.cancel()
                with pytest.raises(asyncio.CancelledError):
                    await task
            assert cleaned.is_set()
            assert committed == acknowledged == (['job-1'] if finish else [])
        finally:
            if not task.done():
                task.cancel()
                await asyncio.gather(task, return_exceptions=True)
    asyncio.run(scenario())
```

A lista csak a műveleti sorrend modellje, nem valódi commit vagy brokerack. A teszt két állapotot vizsgál: a befejezett és a commit előtti megszakított munkát. A commit utáni, ack előtti leállást a deduplikációs és outbox fejezet hibamodelljével együtt kell kezelni.

## Saját shutdown-szerződés

1. Új kézbesítések leállítása vagy a fogyasztó leiratkozása, a broker protokollja szerint.
2. Aktív munkák drainje véges határidővel, megfelelő heartbeat/lease mellett.
3. Maradék munkák megszakítása; befejezetlen állapotuk és újrapróbálásuk rendezése.
4. Taskok tényleges bevárása és erőforrásaik lezárása.
5. Kapcsolatok lezárása és a supervisor végső leállítási keretének betartása.

## Senior döntések és tipikus hibák

A heartbeat az életképesség jele, nem üzleti előrehaladás. Egy végtelen hurok is küldhet heartbeatet. Mérd külön a job korát és előrehaladását; a nagyobb timeout nem feltétlenül javítás.

A threadbe kiszervezett műveletet task cancellation nem állítja le garantáltan. A SIGKILL sem futtat biztos finally-t. Hosszú munkánál tartós checkpoint lehet hasznos, de a checkpoint és már megtörtént mellékhatás közti egyezést is meg kell tervezni.

A task_done az asyncio.Queue helyi számlálóját csökkenti, nem brokerack. A [6. témakör korlátos queue-ja](../06-concurrency/07-bounded-workers-and-shutdown.md) helyi drainhez jó alap, de külön szükséges az elosztott kézbesítési szerződés.

## Interview questions

**Should cleanup acknowledge an unfinished message?**

Válaszvázlat: Nem. A cleanup erőforrást zár, az ack a dokumentált sikeres feldolgozási határt jelenti.

**Does a heartbeat prove progress?**

Válaszvázlat: Nem; csak elérhetőséget/életjelet mutat. Külön jobkor- és előrehaladásmérés kell.

## Önellenőrzés

- [ ] A megszakított feladat nem kap hamis sikeres ackot.
- [ ] A workerhez tartozó taskok a teszt hibája után is rendezettek.

## Kapcsolódó gyakorlat

[R07 – önálló feladat](../../../exercises/senior-python/11-reliable-services.md#r07)

## Forrás és továbbolvasás

[Celery worker shutdown](https://docs.celeryq.dev/en/stable/userguide/workers.html)

[Async cancellation](https://docs.python.org/3.12/library/asyncio-task.html#task-cancellation)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
