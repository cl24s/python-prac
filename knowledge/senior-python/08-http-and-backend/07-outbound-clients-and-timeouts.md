# HTTP-kliens, connection pool és timeoutkeret

## Mit kell tudnod?

- A kliens életciklusát az alkalmazás erőforrásaihoz igazítani.
- A connect/read/write/pool timeoutot külön kezelni a teljes határidőtől.
- A transporthibát, HTTP-hibastátuszt és hibás választ külön kategóriaként látni.

## Magyarázat

Tartós Client/AsyncClient használatával újrahasználhatók a kapcsolatok. Ne készíts új klienst minden iterációban: így elvész a pooling előnye. A kliens tulajdonosa zárja le; async alkalmazásban tipikusan a startup/shutdown életciklushoz igazodik. Minden processz-worker saját poolja tovább növeli az összes kapcsolatszámot.

| Timeout | Mire vár? |
| --- | --- |
| Connect | Kapcsolat felépítése |
| Read | Adat érkezése az olvasás során |
| Write | Adat elküldése az írás során |
| Pool | Szabad kapcsolat a klienspoolból |

A read timeout nem feltétlenül teljes válaszidőkorlát: folyamatosan érkező kis darabok mellett a kérés sokáig tarthat. A teljes felhasználási esetnek külön deadline kell, amelybe a sorban állás, az összes downstream hívás és a retry is belefér.

## Kódpélda: kliensadapter kontrollált transporttal

Önálló Python-script, HTTPX szükséges. A handler nem végez hálózati I/O-t; a timeoutot kifejezetten hiba-injektálással állítjuk elő.

```python
import httpx

class UpstreamUnavailable(Exception):
    pass

class InvalidUpstreamResponse(Exception):
    pass

def read_status(client: httpx.Client, job_id: int) -> str:
    try:
        response = client.get(f'/jobs/{job_id}')
        response.raise_for_status()
    except httpx.TimeoutException as exc:
        raise UpstreamUnavailable('upstream timeout') from exc
    except httpx.HTTPStatusError as exc:
        raise UpstreamUnavailable('upstream error status') from exc
    try:
        data = response.json()
    except ValueError as exc:
        raise InvalidUpstreamResponse('invalid JSON') from exc
    if not isinstance(data, dict) or data.get('status') not in ('queued', 'done'):
        raise InvalidUpstreamResponse('unexpected status payload')
    return data['status']

def handler(request):
    if request.url.path == '/jobs/1':
        return httpx.Response(200, json={'status': 'done'})
    if request.url.path == '/jobs/2':
        raise httpx.ReadTimeout('injected read timeout', request=request)
    if request.url.path == '/jobs/3':
        return httpx.Response(503)
    return httpx.Response(200, json={'state': 'done'})

timeout = httpx.Timeout(connect=1.0, read=2.0, write=2.0, pool=0.5)
limits = httpx.Limits(max_connections=10, max_keepalive_connections=5)
with httpx.Client(base_url='https://example.test', timeout=timeout, limits=limits,
                  transport=httpx.MockTransport(handler)) as client:
    assert read_status(client, 1) == 'done'
    for job_id, cause in [(2, httpx.ReadTimeout), (3, httpx.HTTPStatusError)]:
        try:
            read_status(client, job_id)
        except UpstreamUnavailable as exc:
            assert isinstance(exc.__cause__, cause)
        else:
            raise AssertionError('Expected upstream failure')
    try:
        read_status(client, 4)
    except InvalidUpstreamResponse:
        pass
    else:
        raise AssertionError('Expected schema failure')
assert client.is_closed
```

A mock transport mellett a beállított valódi pool és hálózati timeout nem kerül terhelés alá. A teszt az adapter hibafordítását és payload-ellenőrzését igazolja. A HTTP-hibastátuszhoz raise_for_status kell; a 503 válasz nem automatikusan hálózati kivétel.

## Senior döntések és tipikus hibák

Ez a kis adapter minden HTTP-hibastátuszt egy kategóriába fordít. Éles API-nál például a 404 lehet dokumentált „nincs ilyen job”, a 401 konfigurációs/hitelesítési hiba, a 429 túlterhelési jelzés. Ne retryold őket egységesen. A ConnectError és más transporthibák ebben a példában továbbterjednek; a hívó szerződésében ezeket is tudatosan kezeld.

Két downstream hívásnál ne adj mindkettőnek újra teljes öt másodpercet, ha az egész kérés kerete öt másodperc. Monoton órával számolt maradék idővel dolgozz. Az async timeout kooperatív: blokkoló kód és hosszú cleanup túllépheti a névleges határidőt.

Streaming válasznál a fogyasztás vagy explicit close nélkül fogva maradhat a kapcsolat. A pool limit nem ugyanaz, mint a teljes alkalmazás konkurenciakorlátja, különösen HTTP/2 multiplexelés mellett. A retry legyen idempotenciához, korláthoz, backoffhoz és a fennmaradó időkerethez kötve; ennek alapját az [5. témakör](../05-error-handling/04-partial-failures-and-retry.md) adja.

## Interview questions

**Why is a read timeout not a total deadline?**

Válaszvázlat: Az olvasási inaktivitást korlátozza; több adatdarab, hívás és retry együtt jóval tovább tarthat.

**Does MockTransport test connection pool exhaustion?**

Válaszvázlat: Nem; a kontrollált válasz és hiba az adapterlogikát teszteli. A valódi pool-kimerüléshez más integrációs ellenőrzés szükséges.

## Önellenőrzés

- [ ] Külön nevezem meg a hibastátuszt és a transporthibát.
- [ ] A pool, kliens és stream tulajdonosát is kijelölöm.

## Kapcsolódó gyakorlat

[H07 – önálló feladat](../../../exercises/senior-python/08-http-and-backend.md#h07)

## Forrás és továbbolvasás

[HTTPX timeout types](https://www.python-httpx.org/advanced/timeouts/)

[HTTPX clients](https://www.python-httpx.org/advanced/clients/)

[HTTPX mock transports](https://www.python-httpx.org/advanced/transports/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
