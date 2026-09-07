# FastAPI-tesztek, override és integrációs határok

## Mit kell tudnod?

- Friss appot és visszaállított dependency override-ot használni.
- Külön tesztelni a HTTP-határt és a valódi adaptert.
- Az auth, lifespan és hibás válaszok lefedését megtervezni.

## Magyarázat

A TestClient a FastAPI alkalmazást HTTP-szerű felületen hívja meg hálózati szerver nélkül. Így valódi routing, bemeneti validáció és válaszszerializáció fut. Az override egy kiválasztott függőséget helyettesít; a valódi adapter működését ezzel nem teszteltük.

Az app.dependency_overrides kulcsa az eredeti callable objektum, nem a neve szövegként. A módosítást finally-ben vagy fixture teardownban állítsd vissza. Külön app factory tovább csökkenti az állapotszivárgást.

## Kódpélda: fake, majd visszaállított függőség

```python
from typing import Annotated
from fastapi import Depends, FastAPI
from fastapi.testclient import TestClient

class StatusService:
    def status(self):
        return 'real-adapter-placeholder'

class FakeService:
    def status(self):
        return 'queued'

def get_service():
    return StatusService()

def create_app():
    app = FastAPI()
    @app.get('/status')
    def status(service: Annotated[StatusService, Depends(get_service)]):
        return {'status': service.status()}
    return app

def test_override_and_restore():
    app = create_app()
    app.dependency_overrides[get_service] = lambda: FakeService()
    try:
        with TestClient(app) as client:
            assert client.get('/status').json() == {'status': 'queued'}
    finally:
        app.dependency_overrides.clear()
    with TestClient(app) as client:
        assert client.get('/status').json() == {'status': 'real-adapter-placeholder'}
```

A placeholder direkt nem valódi hálózati adapter. A teszt az override bekötését és visszaállítását bizonyítja. Az éles adapterhez külön szerződés-/integrációs teszt kell, például a 8. témakör kontrollált HTTP transportjával.

## Saját tesztmátrix egy job-létrehozáshoz

| Határ | Mit ellenőrizz? |
| --- | --- |
| Domain unit | Engedélyezett állapotváltás, tiltott ismétlés |
| HTTP + fake service | Body, 201/409/422, kimeneti mezők |
| Valódi DB-adapter | Commit, rollback, egyediség, új sessionből látható adat |
| Auth-bekötés | Hiányzó/hibás token, másik tenant, jogosulatlan művelet |
| Lifespan | Erőforrás megnyitás és bezárás |
| Deploy smoke | Proxy, TLS, readiness és leállítás |

## Senior döntések és tipikus hibák

A TestClient alapból továbbdobhatja a szerver exceptionjét a tesztnek. Ha a 500-as HTTP-választ akarod ellenőrizni, a raise_server_exceptions=False beállítást tudatosan használd. Ne cseréld mindenhol erre: az eredeti traceback sokszor gyorsabb diagnózis.

AsyncClient + ASGITransport esetén a lifespan indítását külön biztosítani kell. Ne ossz loophoz kötött klienst vagy AsyncSessiont a TestClient másik loopjával; az async integrációs erőforrás és teszt ugyanazon lifecycle-ban fusson.

A request sikerét ne csak 200-zal igazold: nézd a bodyt, headereket és a fontos mellékhatást is. Auth override mellett külön auth-teszt kell; DB fake mellett külön tárolói teszt. Teljes OpenAPI snapshot helyett sokszor célzott szerződésassertion stabilabb.

## Interview questions

**What does an overridden dependency test not prove?**

Válaszvázlat: A helyettesített adapter valódi I/O-ját, konfigurációját vagy auth-validációját; csak a környező HTTP-bekötést.

**How do you prevent overrides from leaking?**

Válaszvázlat: Friss app és finally/fixture-teardown; az eredeti callable a kulcs, a visszaállítás hibás teszt után is lefut.

## Önellenőrzés

- [ ] Megnevezem, mely részek valódiak a tesztben.
- [ ] Hibás ágon is eltávolítom az override-ot.

## Kapcsolódó gyakorlat

[F10 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f10)

## Forrás és továbbolvasás

[Testing dependency overrides](https://fastapi.tiangolo.com/advanced/testing-dependencies/)

[Async tests](https://fastapi.tiangolo.com/advanced/async-tests/)

[Starlette TestClient](https://starlette.dev/testclient/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
