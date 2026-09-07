# async def, def és blokkoló függőségek

## Mit kell tudnod?

- Megkülönböztetni a framework által hívott sync endpointot a közvetlen sync hívástól.
- Blokkoló I/O-t és CPU-munkát külön kezelni.
- Konkurenciát és erőforráskorlátot együtt tervezni.

## Magyarázat

FastAPI a normál def endpointot és sync dependencyt threadpoolon keresztül futtatja. Az async def endpoint az event loopon fut. De ha az async függvényedből közvetlenül meghívsz egy normál blokkoló segédfüggvényt, arra nem kerül automatikus threadpool-varázslat.

Async adatbázis-driverhez vagy HTTP-klienshez awaitet használj. Szinkron I/O könyvtárhoz sync endpoint vagy célzott thread-offload jöhet szóba. Tiszta Python CPU-munka hagyományos CPythonban nem lesz automatikusan gyorsabb a threadektől; külön processz vagy worker lehet indokolt.

## Kódpélda: a hívás helyének ellenőrzése

A teszt nem időt mér, hanem a futási threadet hasonlítja össze.

```python
import threading
from fastapi import FastAPI
from fastapi.testclient import TestClient
from starlette.concurrency import run_in_threadpool

app = FastAPI()

def blocking_library_marker():
    return threading.get_ident()

@app.get('/execution')
async def execution():
    loop_thread = threading.get_ident()
    direct = blocking_library_marker()
    offloaded = await run_in_threadpool(blocking_library_marker)
    return {'direct_is_loop': direct == loop_thread,
            'offloaded_is_loop': offloaded == loop_thread}

def test_explicit_offload():
    with TestClient(app) as client:
        assert client.get('/execution').json() == {
            'direct_is_loop': True, 'offloaded_is_loop': False}
```

A marker nem végez valódi blokkoló I/O-t; csak azt bizonyítja, hol futna a hívás. A threadpool sem végtelen. A hosszú sync hívások a többi threadpool-felhasználót is feltarthatják, ezért timeout, kapacitás és megfigyelés kell.

## Senior döntések és tipikus hibák

Ne hozz létre korlátlan taskot minden bemeneti elemre. A klienspool, a DB-pool és a worker-szám együttesen határozza meg, hol keletkezik várakozás. A 6. témakör korlátos queue-ja és cancellation-szabályai itt is érvényesek.

A kérés cancellationje nem állítja le garantáltan a már futó threadet vagy a külső szolgáltatást. A kliensoldali timeout nem rollback. Külső hívás közben ne tarts indokolatlanul nyitva adatbázis-tranzakciót, mert közben kapcsolatot és lockot foglalhat.

A threadpool konkrét alapkapacitását ne interjútrükként tanuld: a használt Starlette/AnyIO-verzió és konfiguráció számít. A jó válasz megnevezi a szűk keresztmetszetet és a mérendő várakozási időt.

## Interview questions

**Does FastAPI offload every synchronous function?**

Válaszvázlat: Csak az általa sync endpointként vagy dependencyként meghívott függvényeknél automatikus. Saját közvetlen hívásnál a hívó threadjén fut.

**Can cancelling a request undo an external side effect?**

Válaszvázlat: Nem; külön idempotencia, timeout és üzleti helyreállítás kell.

## Önellenőrzés

- [ ] Felismerem a sync SDK-hívást async endpointban.
- [ ] A pool és a taskok számát együtt korlátozom.

## Kapcsolódó gyakorlat

[F04 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f04)

## Forrás és továbbolvasás

[FastAPI async behavior](https://fastapi.tiangolo.com/async/)

[Starlette thread pool](https://starlette.dev/threadpool/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
