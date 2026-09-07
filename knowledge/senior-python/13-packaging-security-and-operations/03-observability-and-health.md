# Strukturált logok, metrikák, tracing és health checkek

## Mit kell tudnod?

- Külön kérdésekhez használni logot, metrikát és trace-t.
- Korlátozni a metric label-kardinalitást és a naplózott adatot.
- Elkülöníteni a startup, readiness és liveness jelentését.

## Magyarázat

A log egy esemény részleteit adja; a metrika sok esemény összesített viselkedését; a trace egy művelet útját és részidőit. Egy magas hibaarány jelzi, hogy vizsgálódni kell; egy trace megmutathatja a lassú függőséget; a korrelált log segít értelmezni a hibát. A trace mintavételezett lehet, ezért nem helyettesíti az összes kérésből képzett szolgáltatási metrikát.

Egy HTTP-metrika dimenziója lehet metódus, útvonalsablon és státuszosztály. A nyers URL, user ID vagy request ID korlátlan idősorszámot okozhat. A részletes korreláció logba vagy trace-be való, megfelelő adatkezeléssel. A trace ID sem hitelesítési bizonyíték; a bejövő kontextus nem megbízható jogosultsági adat.

## Kódpélda: explicit eseményséma

```python
import json
import pytest

def completion_event(*, route, status, duration_ms, request_id):
    if route not in {'/jobs', '/jobs/{job_id}'}:
        raise ValueError('unknown route template')
    if type(status) is not int or not 100 <= status <= 599:
        raise ValueError('invalid status')
    if type(duration_ms) is not int or duration_ms < 0:
        raise ValueError('invalid duration')
    if not isinstance(request_id, str) or not 1 <= len(request_id) <= 64:
        raise ValueError('invalid request id')
    return json.dumps({
        'event': 'http.completed', 'route': route, 'status': status,
        'duration_ms': duration_ms, 'request_id': request_id,
    }, ensure_ascii=True)

def test_event_is_one_line_and_structured():
    line = completion_event(route='/jobs/{job_id}', status=200,
                            duration_ms=7, request_id='r1\nforged')
    assert '\n' not in line
    event = json.loads(line)
    assert event['duration_ms'] == 7
    assert set(event) == {'event', 'route', 'status', 'duration_ms', 'request_id'}

def test_raw_path_is_rejected():
    with pytest.raises(ValueError, match='route'):
        completion_event(route='/jobs/private-id', status=200,
                         duration_ms=1, request_id='r1')
```

Az explicit mezők miatt a hívó nem adhat át tetszőleges request bodyt vagy Authorization headert. A JSON serializer escape-eli a sortörést; a későbbi logviewer megjelenítési biztonsága külön felelősség. A minta nem teljes logging-konfiguráció: éles eseményhez időbélyeg, szint, service és verzió is kell. A request ID itt csak bounded korrelációs adat, nem metric label.

## Health check döntési tábla

| Jel | Kérdés | Hibára adott reakció |
| --- | --- | --- |
| Startup | Befejeződött az indulás? | Az indulási türelmi idő után újraindítás lehet |
| Readiness | Kaphat most új forgalmat? | Kivétel a forgalomból |
| Liveness | Segítene a folyamat újraindítása? | Újraindítás |

Külső DB kiesése általában nem javul attól, hogy minden podot újraindítasz. A readiness függőségvizsgálata csak akkor indokolt, ha a szolgáltatás anélkül valóban nem szolgálhat ki; egy opcionális ajánlórendszer hibája ne vegye ki a rendelési API-t. A probe maga legyen olcsó és időkorlátos. A felülete ne szivárogtasson belső címet vagy credentialt.

Egy dashboardhoz előre írd le: kérésráta, hibaarány, latency-eloszlás, erőforrás-telítettség, queue age. A „sok ERROR log” önmagában gyenge riasztási feltétel; a felhasználói hatáshoz és beavatkozási lehetőséghez kösd.

## Interview questions

**Why should request IDs not be metric labels?**

Válaszvázlat: Majdnem minden kérés új idősorhoz vezetne. A metrikához bounded dimenzió, a részletekhez log és trace kell.

**Should a database outage fail liveness?**

Válaszvázlat: Általában nem: az újraindítás nem javítja a DB-t és restartvihart okozhat. A kiszolgálhatóság readiness-kérdés.

## Önellenőrzés

- [ ] Egy lassú kérésnél tudom, melyik jelet keresném.
- [ ] A health végpont nem végez drága üzleti műveletet.

## Kapcsolódó gyakorlat

[OP03 – önálló feladat](../../../exercises/senior-python/13-packaging-security-and-operations.md#op03)

## Forrás és továbbolvasás

[OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/)

[Kubernetes probes](https://kubernetes.io/docs/concepts/workloads/pods/probes/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
