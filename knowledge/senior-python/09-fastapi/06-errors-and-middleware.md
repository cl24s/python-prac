# Exception handlerek, middleware és hibakontextus

## Mit kell tudnod?

- Domainhibát a FastAPI-határon HTTP-válasszá fordítani.
- A bemeneti és kimeneti validáció hibáját megkülönböztetni.
- Kérésenkénti kontextust biztonságosan kezelni.

## Magyarázat

A domain ne dobjon FastAPI HTTPExceptiont csak azért, mert most HTTP-n keresztül hívjuk. Saját kivételt az app exception handlere fordíthat stabil státusszá és hibasémává. A kliens hibás bemenete és a saját kimeneti modell megsértése más felelősség: az utóbbi szerveroldali hiba.

A middleware keresztmetszeti feladatra való, például request ID vagy mérés. A jogosultság erőforrásfüggő részét ne rejtsd egy általános HTTP-middleware-be, amely még nem ismeri a betöltött jobot.

## Kódpélda: request ID és kontrollált domainhiba

```python
from uuid import uuid4
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from fastapi.testclient import TestClient

class JobConflict(Exception):
    pass

app = FastAPI()

@app.middleware('http')
async def request_context(request: Request, call_next):
    request.state.request_id = str(uuid4())
    response = await call_next(request)
    response.headers['X-Request-ID'] = request.state.request_id
    return response

@app.exception_handler(JobConflict)
async def conflict_handler(request: Request, exc: JobConflict):
    return JSONResponse(status_code=409, media_type='application/problem+json',
                        content={'type': 'about:blank', 'title': 'Conflict',
                                 'status': 409, 'detail': 'Job is already closed.',
                                 'request_id': request.state.request_id})

@app.post('/jobs/1/start')
def start_job():
    raise JobConflict('private internal state')

def test_error_context():
    with TestClient(app) as client:
        response = client.post('/jobs/1/start')
        assert response.status_code == 409
        assert response.json()['request_id'] == response.headers['X-Request-ID']
        assert 'private internal state' not in response.text
```

A példa saját azonosítót generál; nem vesz át ellenőrzés nélkül kliens által küldött tetszőleges logkontextust. A kezelt 409-es ágat mutatja. Ismeretlen exceptionnél a middleware és a szerverhiba-handler viszonya eltérhet; a fenti kód nem ígér minden 500-as válaszra azonos fejlécet.

## Senior döntések és tipikus hibák

A RequestValidationError saját handlerrel egységesíthető, de ne küldd vissza vakon a teljes hibás bodyt vagy minden exception input mezőjét: érzékeny adatok lehetnek benne. A naplóban stabil request ID és releváns, szűrt kontextus legyen; ne naplózz minden rétegben ugyanarra a hibára új tracebacket.

A middleware call_next körüli időmérés nem feltétlenül a teljes stream klienshez érkezését méri. Streaming közben bekövetkező hibánál a fejlécek már kimehettek. Nagy bodyk teljes beolvasása naplózáshoz megváltoztatja a memóriaigényt és a streaming működését.

ContextVar-alapú tracingnél a middleware típusa és a taskhatárok befolyásolhatják a kontextusterjedést. Egyszerű request.state elegendő lehet a kérésen belüli átadáshoz; bonyolult infrastruktúrához vizsgáld meg a pure ASGI middleware lehetőségét.

## Interview questions

**Should domain code raise HTTPException?**

Válaszvázlat: Általában saját domainhibát adjon; a HTTP-adapter fordítja, így CLI-ből vagy workerből is használható.

**Why not return the original exception text to the client?**

Válaszvázlat: Belső állapotot, titkot vagy változó implementációs részletet fedhet fel; stabil publikus hibaszerződés kell.

## Önellenőrzés

- [ ] A kezelt és kezeletlen hibás út korlátait külön megnevezem.
- [ ] Nem naplózom automatikusan a teljes request bodyt.

## Kapcsolódó gyakorlat

[F06 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f06)

## Forrás és továbbolvasás

[FastAPI error handling](https://fastapi.tiangolo.com/tutorial/handling-errors/)

[Starlette middleware limitations](https://starlette.dev/middleware/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
