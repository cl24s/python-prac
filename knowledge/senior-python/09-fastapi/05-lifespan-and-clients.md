# Lifespan, alkalmazásállapot és klienspoolok

## Mit kell tudnod?

- App-szintű erőforrást startup/shutdown határhoz kötni.
- App factoryval izolálható alkalmazást létrehozni.
- Startuphibát és részleges erőforrásnyitást kezelni.

## Magyarázat

Lifespan alatt jönnek létre az alkalmazás közösen használt erőforrásai: például HTTP-kliens vagy adatbázis-engine. A yield előtti rész startup, az utána következő shutdown. A context manager a részleges hibák takarítását is segíti; több erőforrásnál AsyncExitStack lehet indokolt.

Az app factory új FastAPI-példányt készít. Tesztekhez külön erőforrást adhat, importáláskor nem kell hálózati kapcsolatot nyitnia. Az app.state megosztott objektumokat tartalmazhat; a benne tárolt módosítható üzleti dict ettől még nem lesz többworkerű adatbázis.

## Kódpélda: kliens megnyitása és igazolt lezárása

```python
from contextlib import asynccontextmanager
import httpx
from fastapi import FastAPI, Request
from fastapi.testclient import TestClient

def create_app():
    @asynccontextmanager
    async def lifespan(app):
        transport = httpx.MockTransport(lambda request: httpx.Response(200, json={'ok': True}))
        async with httpx.AsyncClient(transport=transport, base_url='https://example.test',
                                     timeout=2.0) as client:
            app.state.upstream = client
            yield

    app = FastAPI(lifespan=lifespan)

    @app.get('/upstream')
    async def upstream(request: Request):
        response = await request.app.state.upstream.get('/health')
        response.raise_for_status()
        return response.json()

    return app

def test_lifespan_client():
    app = create_app()
    with TestClient(app) as client:
        assert not app.state.upstream.is_closed
        assert client.get('/upstream').json() == {'ok': True}
    assert app.state.upstream.is_closed
```

A TestClient context manager aktiválja a lifespan-t. Az egyszerű konstruktorhívásra ne alapozd a startup bizonyítását. A példa mock transporttal fut: a tulajdonlást és bezárást ellenőrzi, nem a valódi kapcsolatok újrahasználatát.

## Senior döntések és tipikus hibák

Minden worker saját startupot és saját poolokat kap. Ha négy worker mindegyike húsz DB-kapcsolatot enged, az nem összesen húsz. Több replika tovább szorozza a kapacitást; az adatbázis teljes kerete és más szolgáltatások használata is számít.

Egy opcionális upstream kiesése ne tegye szükségképpen indulásképtelenné az egész appot. Kötelező konfigurációs hiba viszont legyen látható startuphiba. Döntsd el, melyik függőség readiness-feltétel, és melyiknél lehet korlátozott szolgáltatást nyújtani.

Ne futtass minden worker startupjában koordinálatlan sémamigrációt. A migráció külön, ellenőrzött deployment-lépés. Shutdownkor először a folyamatban lévő munka tulajdonlását tisztázd, csak azután zárd el az általa használt erőforrást.

## Interview questions

**Why use an application factory in tests?**

Válaszvázlat: Új appot és elkülönített függőségeket ad, importkori mellékhatások nélkül.

**Is lifespan executed once for the whole deployment?**

Válaszvázlat: Nem; processzenként/alkalmazáspéldányonként kell számolni vele, ezért a pool és startupműveletek többszöröződnek.

## Önellenőrzés

- [ ] A kliens lezárását a teszt is ellenőrzi.
- [ ] Nem kezelem globálisnak a worker memóriáját.

## Kapcsolódó gyakorlat

[F05 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f05)

## Forrás és továbbolvasás

[Lifespan events](https://fastapi.tiangolo.com/advanced/events/)

[Testing lifespan](https://fastapi.tiangolo.com/advanced/testing-events/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
