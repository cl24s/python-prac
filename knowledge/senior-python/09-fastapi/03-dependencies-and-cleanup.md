# Depends, függőségláncok és erőforrás-életciklus

## Mit kell tudnod?

- A dependency feloldását elkülöníteni a normál függvényhívástól.
- Kérésen belüli cache-t és yield-cleanupot megérteni.
- A cleanup határát tudatosan választani.

## Magyarázat

A Depends nem általános Python-szintű automatikus injektálás. FastAPI a kérés feldolgozásakor járja be a függőségeket. Ha közvetlenül meghívod a függvényt, a szükséges argumentumot te adod át. Az alkalmazásmagban ezért egyszerű paraméterek vagy interfészek maradjanak.

Ugyanaz a függőség alapértelmezés szerint egy kérésen belül újrahasznosítható eredményt ad, nem alkalmazásszintű singletont. A use_cache=False külön feloldást kér. Egy drága HTTP-kliens ezért inkább lifespan-erőforrás, egy tranzakciós session tipikusan rövidebb életű.

## Kódpélda: egy megnyitás, két fogyasztó, takarítás hibánál is

```python
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException
from fastapi.testclient import TestClient

app = FastAPI()
events = []

def get_resource():
    events.append('open')
    try:
        yield object()
    finally:
        events.append('close')

Resource = Annotated[object, Depends(get_resource, scope='function')]

@app.get('/check')
def check(first: Resource, second: Resource, fail: bool = False):
    assert first is second
    if fail:
        raise HTTPException(status_code=409, detail='rejected')
    return {'same': first is second}

def test_cleanup_and_request_cache():
    events.clear()
    with TestClient(app) as client:
        assert client.get('/check').json() == {'same': True}
        assert client.get('/check?fail=true').status_code == 409
    assert events == ['open', 'close', 'open', 'close']
```

A scope='function' explicit korai takarítást választ: a path operation vége után, a válasz kiküldése előtt zárul a függőség. A yield dependency alapértelmezett request scope-ja későbbi, a válasz kiküldése utáni életciklust enged. Streamingnél döntő, használja-e még a body-generátor az erőforrást. Ezek verzióérzékeny részletek; a README rögzíti a kipróbált környezetet.

## Senior döntések és tipikus hibák

Ne nyelj el kivételt a yield körül. Ha fordítod a hibát, dobj értelmes új exceptiont; ha csak cleanup kell, finally a megfelelő hely. A yield utáni commit veszélyes lehet, ha a sikeres válasz már elindult. A tranzakció eredménye derüljön ki a sikeres válasz előtt.

A dependency-gráf lezárási sorrendje is függőség: egy hosszabb életű erőforrás cleanupja nem támaszkodhat egy már megszűnt rövidebb életű erőforrásra. Ne használd a kérés sessionjét későbbi háttérmunkában; adj át azonosítót, és a worker szerezzen saját erőforrást.

## Interview questions

**Is dependency caching global?**

Válaszvázlat: Nem; az alapértelmezett újrahasznosítás a kérés függőségfeloldásán belül történik.

**Why can cleanup timing matter for streaming?**

Válaszvázlat: A stream a handler visszatérése után is olvashat erőforrást. A túl korai zárás hibát okoz, a túl hosszú élet viszont kapcsolatot foglal.

## Önellenőrzés

- [ ] A függvényt közvetlenül is explicit argumentumokkal tudom hívni.
- [ ] Commit és cleanup időzítését nem keverem össze.

## Kapcsolódó gyakorlat

[F03 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f03)

## Forrás és továbbolvasás

[Dependencies with yield and scopes](https://fastapi.tiangolo.com/tutorial/dependencies/dependencies-with-yield/)

[Sub-dependency caching](https://fastapi.tiangolo.com/tutorial/dependencies/sub-dependencies/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
