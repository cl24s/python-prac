# Lefagyás, threadstack és async taskok vizsgálata

## Mit kell tudnod?

- A látszólag lefagyott processzről bizonyítékot gyűjteni.
- Threadstack és coroutine/task állapot különbségét ismerni.
- Diagnosztika után biztonságosan rendezni az erőforrásokat.

## Magyarázat

A processz fut, de nem válaszol: lehet lockvárakozás, külső I/O, telített pool, CPU-hurok vagy blokkolt event loop. Az újraindítás helyreállíthat, de eltüntetheti a bizonyítékot. Ha az üzemi helyzet engedi, előbb időponttal és terhelési adatokkal együtt rögzíts állapotot.

A faulthandler Python-threadstackeket írhat fájlba. Ismételt stackminták segítenek elkülöníteni az előrehaladást és a tartós várakozást. Async appban sok coroutine osztozik egy threaden; a threadstack mellett a taskok neve és stackje is szükséges lehet.

## Kódpélda: kontrolláltan váró thread diagnosztikája

```python
import asyncio
import faulthandler
import threading

def test_waiting_thread_stack(tmp_path):
    started, release = threading.Event(), threading.Event()
    def wait_until_released():
        started.set()
        release.wait()
    worker = threading.Thread(target=wait_until_released, name='diagnostic-worker')
    worker.start()
    path = tmp_path / 'threads.txt'
    try:
        assert started.wait(timeout=2)
        with path.open('w+', encoding='utf-8') as stream:
            faulthandler.dump_traceback(file=stream, all_threads=True)
            stream.seek(0)
            report = stream.read()
        assert 'wait_until_released' in report
    finally:
        release.set()
        worker.join(timeout=2)
    assert not worker.is_alive()

def test_pending_async_task_is_visible():
    async def scenario():
        started, release = asyncio.Event(), asyncio.Event()
        async def wait_for_work():
            started.set()
            await release.wait()
        task = asyncio.create_task(wait_for_work(), name='waiting-for-work')
        try:
            await asyncio.wait_for(started.wait(), timeout=2)
            pending = {t.get_name(): t for t in asyncio.all_tasks() if t is not asyncio.current_task()}
            assert pending['waiting-for-work'] is task
            assert task.get_stack()
        finally:
            release.set()
            await asyncio.wait_for(task, timeout=2)
    asyncio.run(scenario())
```

A teszt nem idéz elő valódi deadlockot: olyan ismert várakozást hoz létre, amely végül feloldható. A stackben látható wait önmagában nem hiba, csak állapot. A diagnózis azt is megkérdezi, ki fogja felszabadítani, és miért nem tette még meg.

## Saját incident-menet

1. Határozd meg a hatókört: minden endpoint, egy worker, egy tenant vagy egy upstream?
2. Rögzíts CPU-, memória-, kapcsolat-, queue- és lockadatot azonos időablakból.
3. Nézz több stackmintát; keresd a közös várakozási helyet és az azt feloldó szereplőt.
4. Ellenőrizd a friss változásokat és a konkrét request/job összefüggését.
5. Állíts helyre indokoltan, majd reprodukáld a legkisebb hibaforgatókönyvet és adj célzott regressziós tesztet.

## Senior döntések és tipikus hibák

Az asyncio debug lassú callbackekre és egyes életciklushibákra adhat jelet. De ha maga az event loop blokkolt, egy ugyanazon loopra ütemezett diagnosztikai coroutine sem biztos, hogy elindul. Ilyenkor külső processz-/threadszintű megfigyelés kellhet.

A faulthandler időzített dumpjához a célfájl maradjon nyitva, és a diagnosztikai időzítést le kell állítani, ha már nem kell. Ne tegyél kontrollálatlan stackdump endpointot nyilvános API-ba. A stack, lokális változók és profiladat belső működést vagy érzékeny adatot tartalmazhatnak.

A timeout emelése nem magyarázza meg a deadlockot. A „CPU alacsony, ezért minden rendben” szintén hibás: a teljes rendszer várhat ugyanarra a lockra.

## Interview questions

**Why is a thread dump insufficient for some async incidents?**

Válaszvázlat: Sok coroutine egy threaden várakozik; task-szintű állapot is kell, blokkolt loopnál pedig külső diagnosztika.

**Does a stack frame in Event.wait prove a deadlock?**

Válaszvázlat: Nem; lehet szabályos várakozás. Az állapotot és a feloldó szereplőt együtt kell vizsgálni.

## Önellenőrzés

- [ ] Több időpont stackje alapján keresek előrehaladást.
- [ ] A diagnosztikai teszt végén nem marad élő thread vagy task.

## Kapcsolódó gyakorlat

[P05 – önálló feladat](../../../exercises/senior-python/12-performance-and-debugging.md#p05)

## Forrás és továbbolvasás

[faulthandler](https://docs.python.org/3.12/library/faulthandler.html)

[Asyncio task introspection](https://docs.python.org/3.12/library/asyncio-task.html#introspection)

[Asyncio debug mode](https://docs.python.org/3.12/library/asyncio-dev.html#debug-mode)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
