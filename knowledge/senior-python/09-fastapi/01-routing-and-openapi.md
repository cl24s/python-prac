# Routing, HTTP-modellek és OpenAPI

## Mit kell tudnod?

- Path, query és body forrását egyértelműen deklarálni.
- APIRouterrel szervezni a HTTP-réteget.
- A válaszmodell és OpenAPI szerepét elkülöníteni az üzleti logikától.

## Magyarázat

A FastAPI a függvényszignatúrából, annotációkból és deklarált modellekből építi fel a HTTP-határt. A route-ban szereplő job_id path paraméter; a Query-vel megjelölt limit query paraméter; a Pydantic-modell body lehet. Az Annotated a Python-típus mellé helyezi a framework metaadatait.

Az APIRouter a kapcsolódó endpointok szervezését segíti. Nem kell minden route mögé automatikusan új osztály. A route feladata: bemenet átvétele, felhasználási eset meghívása, eredmény és hiba HTTP-re fordítása. A domain közvetlenül is tesztelhető maradjon.

## Kódpélda: routing és kimeneti szerződés

A teljes blokk önálló `test_example.py`; futtatás: `python -m pytest -q test_example.py`.

```python
from typing import Annotated
from fastapi import APIRouter, FastAPI, Path, Query
from fastapi.testclient import TestClient
from pydantic import BaseModel

class JobView(BaseModel):
    id: int
    status: str

app = FastAPI()
router = APIRouter(prefix='/jobs', tags=['jobs'])

@router.get('/{job_id}', response_model=JobView, operation_id='getJob')
def get_job(job_id: Annotated[int, Path(gt=0)],
            verbose: Annotated[bool, Query()] = False):
    return {'id': job_id, 'status': 'queued', 'internal_note': 'private'}

app.include_router(router)

def test_contract_and_filtering():
    with TestClient(app) as client:
        response = client.get('/jobs/7?verbose=true')
        assert response.status_code == 200
        assert response.json() == {'id': 7, 'status': 'queued'}
        assert client.get('/jobs/0').status_code == 422
        assert client.get('/jobs/seven').status_code == 422
        schema = client.get('/openapi.json').json()
        operation = schema['paths']['/jobs/{job_id}']['get']
        assert operation['operationId'] == 'getJob'
        assert '200' in operation['responses']
```

A belső mező nem kerül a válaszba, mert a response_model kijelöli a publikus szerződést. Ettől még a domainnek helyes adatot kell visszaadnia: kimeneti validációs hiba szerverhiba, nem a kliens hibás bemenete. Közvetlen Response/JSONResponse visszaadása más út: ne feltételezd, hogy arra is ugyanaz a modellvalidáció fut.

## Senior döntések és tipikus hibák

A konkrét /jobs/stats útvonalat a /jobs/{job_id} előtt regisztráld, ha különben dinamikus route fogná meg. A sorrend és az útvonalak átfedése szerződéskérdés; a típusellenőrzés nem automatikus route-visszalépés.

Az OpenAPI a deklarált szerződés gépi leírása. A responses metaadat dokumentálhat egy 409-et, de nem implementálja annak runtime előállítását. Legyen teszt a tényleges válaszra is. Az operation_id változása generált kliensnél törést okozhat; a dokumentációt is verziózott interfészként kezeld.

A példában a verbose deklarációját mutatjuk, de nem változtatja az eredményt; éles endpointnál a paraméternek legyen dokumentált szerepe vagy maradjon el.

## Interview questions

**What does response_model protect, and what does it not?**

Válaszvázlat: Kijelöli és validálja a normál kimeneti szerződést; nem jogosultságkezelés, és a közvetlen Response útját külön kell kezelni.

**Does documenting a response implement it?**

Válaszvázlat: Nem; az OpenAPI deklaráció és a tényleges HTTP-viselkedés külön tesztelendő.

## Önellenőrzés

- [ ] A dinamikus és konkrét route-ok sorrendjét ellenőrzöm.
- [ ] A publikus modellből kihagyom a belső mezőket.

## Kapcsolódó gyakorlat

[F01 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f01)

## Forrás és továbbolvasás

[Response models](https://fastapi.tiangolo.com/tutorial/response-model/)

[Path parameters and order](https://fastapi.tiangolo.com/tutorial/path-params/)

[Additional OpenAPI responses](https://fastapi.tiangolo.com/advanced/additional-responses/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
