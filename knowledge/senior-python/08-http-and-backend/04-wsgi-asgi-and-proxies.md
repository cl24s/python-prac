# WSGI, ASGI, webszerver és reverse proxy

## Mit kell tudnod?

- Megkülönböztetni a framework, szerver és proxy felelősségét.
- Felismerni a WSGI és ASGI alkalmazásinterfészt.
- Az in-process HTTP-teszt határait pontosan megnevezni.

## Magyarázat

A framework route-okat és alkalmazáslogikát szervez. A webszerver fogadja a kapcsolatokat, feldolgozza a hálózati protokollt és meghívja az alkalmazást. A reverse proxy előtte végezhet TLS-lezárást, útválasztást és terheléselosztást. Attól, hogy stabil szerver alá teszel egy appot, egy benne lévő deadlock vagy blokkoló hívás még nem javul meg.

WSGI esetén az alkalmazás environ és start_response argumentumot kap, és byte-darabok iterálhatóját adja vissza. ASGI esetén a scope írja le a kapcsolat/kérés kontextusát, receive/send pedig aszinkron eseményeket közvetít. ASGI HTTP mellett WebSocketet és lifespan eseményeket is képes modellezni.

## Kódpélda: ugyanaz a válasz két alkalmazásinterfészen

Önálló script, HTTPX szükséges. A transportok memórián belül hívják az appot, nem nyitnak hálózati portot.

```python
import asyncio
import httpx

def wsgi_app(environ, start_response):
    body = b'{"status":"ok"}'
    start_response('200 OK', [('Content-Type', 'application/json'),
                              ('Content-Length', str(len(body)))])
    return [body]

async def asgi_app(scope, receive, send):
    if scope['type'] != 'http':
        raise RuntimeError('This teaching app only supports HTTP')
    await send({'type': 'http.response.start', 'status': 200,
                'headers': [(b'content-type', b'application/json')]})
    await send({'type': 'http.response.body', 'body': b'{"status":"ok"}'})

with httpx.Client(transport=httpx.WSGITransport(app=wsgi_app),
                  base_url='http://example.test') as client:
    response = client.get('/health')
    assert response.status_code == 200
    assert response.json() == {'status': 'ok'}

async def main():
    async with httpx.AsyncClient(transport=httpx.ASGITransport(app=asgi_app),
                                 base_url='http://example.test') as client:
        response = await client.get('/health')
        assert response.status_code == 200
        assert response.json() == {'status': 'ok'}

asyncio.run(main())
```

A tanpéldák minden HTTP-útvonalra ugyanazt adják, nincs router, autentikáció vagy startup erőforrás. A HTTPX ASGITransport nem indít automatikusan lifespan eseményeket. Ha az app startupkor nyit adatbázis- vagy klienspoolt, annak lifecycle-ját a tesztben is külön működtetni kell.

## Worker és blokkolás

Több processz-worker általában külön memóriát és saját poolokat jelent. A processzenkénti cache és lock nem globális. A worker-szám és a poolméret szorzata túllépheti az adatbázis kapacitását.

Az async def függvényben közvetlenül futó blokkoló kód feltarthatja az event loopot. A WSGI/ASGI különbség nem azonos a „lassú/gyors” különbséggel; a terhelés és a függőségek határozzák meg az eredményt. A framework által threadpoolba küldött szinkron endpoint részleteit a FastAPI-fejezetben dolgozzuk ki.

## Proxyhatár és diagnosztika

A Forwarded vagy X-Forwarded-* adatot csak ismert proxy felől szabad megbízhatóként kezelni. Ha az alkalmazás közvetlenül elérhető és tetszőleges kliens állíthat ilyen headert, a forrás-IP és az eredeti séma meghamisítható. A bizalmi határt telepítési konfigurációval is védeni kell.

502 esetén nézd meg, melyik köztes réteg állította elő; 504-nél melyik várakozás járt le. Az in-process teszt nem ellenőrzi a DNS-t, TLS-t, proxyfejléceket vagy worker-újraindítást. A valódi deployhoz ezekre külön integrációs ellenőrzés kell.

## Interview questions

**What is the difference between an ASGI application and an ASGI server?**

Válaszvázlat: Az alkalmazás a scope/receive/send szerződést valósítja meg, a szerver a hálózatot és az alkalmazáshívást kezeli.

**Why can an in-process API test pass while deployment fails?**

Válaszvázlat: Kívül marad a DNS, TLS, proxy, worker-konfiguráció és akár a startup lifecycle is.

## Önellenőrzés

- [ ] Nem tulajdonítok hálózati bizonyító erőt a fenti transporttesztnek.
- [ ] A poolméretet worker-számmal együtt vizsgálom.

## Kapcsolódó gyakorlat

[H04 – önálló feladat](../../../exercises/senior-python/08-http-and-backend.md#h04)

## Forrás és továbbolvasás

[WSGI specification](https://peps.python.org/pep-3333/)

[ASGI specification](https://asgi.readthedocs.io/en/stable/specs/main.html)

[HTTPX transports](https://www.python-httpx.org/advanced/transports/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
