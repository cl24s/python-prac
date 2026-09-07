# BackgroundTasks és tartós háttérfeldolgozás

## Mit kell tudnod?

- Az in-process háttérfeladat garanciáit pontosan megnevezni.
- Külön workerre tartozó munkát felismerni.
- Az adatbázis-mentés és üzenetküldés közti hibát megtervezni.

## Magyarázat

A BackgroundTasks a válaszhoz kapcsolt, ugyanabban az alkalmazásfolyamatban futó munkát ütemez. Hasznos lehet kis, elveszíthető kiegészítő művelethez. Nem tartós queue, nincs automatikus újraindítás utáni visszajátszás vagy pontosan egyszeri feldolgozás.

Egy többperces CPU-job, kritikus számlaküldés vagy garantáltan végrehajtandó üzleti munka külön worker és tartós állapot felé mutat. A 202-es státusz ne ígérjen többet, mint amit a rendszer ténylegesen elfogadott és tárolt.

## Kódpélda: kis, nem kritikus értesítés

```python
from fastapi import BackgroundTasks, FastAPI
from fastapi.testclient import TestClient

def test_background_callback():
    notifications = []
    app = FastAPI()

    def record_notification(job_id: int):
        notifications.append(job_id)

    @app.post('/jobs/7/notify', status_code=202)
    def notify(tasks: BackgroundTasks):
        tasks.add_task(record_notification, 7)
        return {'accepted': True}

    with TestClient(app) as client:
        response = client.post('/jobs/7/notify')
        assert response.status_code == 202
        assert response.json() == {'accepted': True}
        assert notifications == [7]
```

A TestClient a kérés feldolgozásakor megvárja ezt a háttércallbacket is, ezért a lista már módosult a hívás visszatérésekor. Ez nem azt jelenti, hogy az éles hálózati kliens csak a munka után kap választ. A teszt nem bizonyít tartósságot vagy processzösszeomlás utáni helyreállítást.

## Saját tervezési példa: job mentése és queue-ba küldése

Ha előbb commitolsz és utána publikálsz üzenetet, a kettő között összeomolhat a folyamat: van job, nincs üzenet. Ha előbb publikálsz, a worker a még nem létező vagy visszagörgetett jobot olvashatja.

Az outbox minta a jobot és a „publikálandó esemény” rekordot ugyanabba a DB-tranzakcióba írja. Külön relay küldi ki az eseményt. A relay is megismételhet publikálást, ezért idempotens fogyasztás vagy deduplikáció kell; az outbox nem automatikus exactly-once megoldás.

## Senior döntések és tipikus hibák

Ne adj át request-sessiont, nyitott streamet vagy kérésélettartamú ORM-objektumot. Azonosító és minimális adat menjen a workerhez, amely saját erőforrást szerez. A callback hibája a már kiküldött sikeres válasz státuszát nem tudja utólag átírni.

A sync background callback threadpool-kapacitást használhat, az async callback blokkoló hívással az event loopot terhelheti. A „background” nem kapacitás- vagy izolációs garancia. A részletes retry/ack/DLQ terv a 11. témakör következő lépése.

## Interview questions

**When is BackgroundTasks insufficient?**

Válaszvázlat: Tartósságot, újrapróbálást, külön skálázást vagy erős erőforrás-izolációt igénylő munkánál.

**Does the outbox pattern guarantee exactly-once delivery?**

Válaszvázlat: Nem; a publikálás ismétlődhet, ezért idempotens feldolgozás és eseményazonosító szükséges.

## Önellenőrzés

- [ ] A callback helyi tesztjét nem nevezem tartóssági ellenőrzésnek.
- [ ] A job mentése és publikálása közti hibát meg tudom nevezni.

## Kapcsolódó gyakorlat

[F09 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f09)

## Forrás és továbbolvasás

[FastAPI background tasks](https://fastapi.tiangolo.com/tutorial/background-tasks/)

[Starlette background execution](https://starlette.dev/background/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
