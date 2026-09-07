# Queue, acknowledgement, retry és dead-letter feldolgozás

## Mit kell tudnod?

- Publisher confirm és consumer ack jelentését megkülönböztetni.
- Korlátos újrakézbesítést és poison-message utat tervezni.
- A DLQ-t ellenőrzött visszajátszási folyamattal összekötni.

## Magyarázat

A publisher confirm a producer–broker szakaszról, a consumer ack a broker–fogyasztó szakaszról ad visszajelzést. Egyik sem automatikus end-to-end üzleti sikerigazolás. A fogyasztó akkor ackeljen, amikor a szerződés szerinti tartós hatás megtörtént.

| Sorrend | Crash kockázata |
| --- | --- |
| Ack, majd üzleti commit | Üzenet eltűnhet elvégzetlen munkával |
| Üzleti commit, majd ack | Újrakézbesítés és duplikáció lehetséges |
| Tranzakciós deduplikáció, majd ack | A helyi hatás replay ellen védhető, brokerbeállítás továbbra is számít |

RabbitMQ-ban a delivery tag csatornához kötött technikai azonosító, nem globális üzleti event ID. A prefetch az egy fogyasztó/csatorna körüli outstanding kézbesítések korlátozásában segít a beállítás szerint; nem helyettesíti a teljes rendszer kapacitástervét.

## Kódpélda: explicit feldolgozási döntés

Ez tiszta policyfüggvény, nem brokeradapter. A „retry” késleltetett újrapróbálási út szándéka; a „dead-letter” ellenőrzött hibasor szándéka.

```python
import pytest

def disposition(outcome, delivery_attempt, max_attempts=3):
    if not 1 <= delivery_attempt <= max_attempts:
        raise ValueError('invalid delivery attempt')
    if outcome == 'committed':
        return 'ack'
    if outcome == 'invalid':
        return 'dead-letter'
    if outcome == 'temporary':
        return 'retry' if delivery_attempt < max_attempts else 'dead-letter'
    raise ValueError('unknown outcome')

@pytest.mark.parametrize('outcome,attempt,expected', [
    ('committed', 1, 'ack'), ('committed', 3, 'ack'),
    ('invalid', 1, 'dead-letter'), ('temporary', 1, 'retry'),
    ('temporary', 3, 'dead-letter'),
])
def test_disposition(outcome, attempt, expected):
    assert disposition(outcome, attempt) == expected
```

A delivery_attempt itt tartósan vezetett policyadat, nem a RabbitMQ redelivered booleanból kikövetkeztetett pontos számláló. A konkrét broker/queue típus és kliens lehetőségeit külön kell bekötni. A dead-letter döntés csak megfelelő routinggal vezet valódi hibasorba; konfiguráció nélkül eldobás is történhet.

## DLQ és visszajátszás

A hibasorhoz kell ok, eseményazonosító, sémaverzió, próbálkozásszám és időpont, érzékeny payload kontrollált kezelésével. Legyen felelős, riasztás és replay-feltétel. A „mindent visszaöntök” újabb kiesést és duplikált hatást okozhat.

Poison message lehet hibás séma vagy determinisztikus programhiba. Átmeneti tárolóhiba más kezelés: késleltetett retry és limit. Az azonnali nack/requeue ciklus forró hurokba kerülhet, amely a jó üzeneteket is kiszorítja. A DLX nem univerzális veszteségmentes garancia; a queue típusának és továbbítási módjának hibakezelését is ellenőrizni kell.

## Celery-kapcsolat és senior döntések

Celerynél az acks_late, retry, worker-loss és timeout beállítások együtt számítanak. Az acks_late önmagában nem bizonyít „minden hiba után biztos replay”-t, és nem pótolja az idempotens taskot. A framework állapotát és az üzleti job rekordját ne tekintsd automatikusan azonosnak.

Üzenetsorrend több worker, retry és DLQ mellett megváltozhat. Ha egy aggregátum sorrendje fontos, sorszám, partíció vagy állapotverzió szerinti ellenőrzés kell. A követelmény az, hogy helyes legyen az állapot, nem hogy reménykedjünk az eredeti sorrendben.

## Interview questions

**Does a publisher confirm mean the job completed?**

Válaszvázlat: Nem; a broker átvételi szakaszára vonatkozik. A feldolgozás és üzleti siker külön visszajelzés.

**Why is immediate requeue risky?**

Válaszvázlat: A determinisztikusan hibás üzenet forró hurokká válhat, kapacitást fogyasztva és késleltetve a jó munkát.

## Önellenőrzés

- [ ] Üzleti commit előtt nem küldök sikeres ackot.
- [ ] A DLQ-hoz replay- és felelősségi terv is tartozik.

## Kapcsolódó gyakorlat

[R04 – önálló feladat](../../../exercises/senior-python/11-reliable-services.md#r04)

## Forrás és továbbolvasás

[RabbitMQ confirms and acknowledgements](https://www.rabbitmq.com/docs/confirms)

[RabbitMQ dead-letter exchanges](https://www.rabbitmq.com/docs/dlx)

[Celery task acknowledgements](https://docs.celeryq.dev/en/stable/userguide/tasks.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
