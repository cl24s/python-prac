# Retry-szabály, backoff, jitter és retrykeret

## Mit kell tudnod?

- A retry biztonságát és várható hasznát külön megvizsgálni.
- Korlátos próbálkozást és késleltetést tervezni.
- Elkerülni a több rétegen megsokszorozott újrapróbálást.

## Magyarázat

Retry akkor segít, ha a hiba átmeneti lehet, a művelet biztonságosan ismételhető és maradt idő/kapacitás. A hibás jelszó vagy hibás payload többnyire nem javul attól, hogy újra elküldöd. Túlterhelésnél a retry új terhelést okoz, ezért nem lehet korlátlan.

Exponenciális backoff növeli a próbák közti időt; felső korlát megakadályozza az indokolatlanul hosszú késleltetést. Full jitter esetén a várakozás 0 és az adott backoff felső határa közül választott érték. Ez szétteríti az egyszerre hibázó kliensek következő próbáját; önmagában nem korlátozza az összes retry mennyiségét.

## Kódpélda: késleltetési döntés tesztelhető véletlennel

```python
import pytest

def retry_delay(failed_attempt, max_attempts, remaining, random_fraction,
                base=0.25, cap=2.0):
    if max_attempts < 1 or not 1 <= failed_attempt <= max_attempts:
        raise ValueError('invalid attempt count')
    if base <= 0 or cap <= 0 or not 0 <= random_fraction <= 1:
        raise ValueError('invalid backoff settings')
    if failed_attempt == max_attempts or remaining <= 0:
        return None
    ceiling = min(base, cap)
    for _ in range(failed_attempt - 1):
        ceiling = min(cap, ceiling * 2)
        if ceiling == cap:
            break
    delay = ceiling * random_fraction
    return delay if delay < remaining else None

def test_retry_limits_and_jitter():
    assert retry_delay(1, 4, 10, 0.5) == 0.125
    assert retry_delay(2, 4, 10, 0.5) == 0.25
    assert retry_delay(8, 10, 10, 0.5) == 1.0
    assert retry_delay(4, 4, 10, 0.5) is None
    assert retry_delay(2, 4, 0.1, 0.5) is None
    with pytest.raises(ValueError):
        retry_delay(0, 4, 10, 0.5)
```

A max_attempts az első próbát is tartalmazza. A None azt jelenti, hogy a policy szerint nincs következő próbálkozás. A függvény nem osztályoz exceptiont és nem hajt végre hívást. Éles hívó a várakozás után újra ellenőrizze a fennmaradó időt, és a következő I/O timeoutját is ebből számolja.

## Retrykeret és rétegek

Ha egy API háromszor próbál, a service háromszor és az SDK is háromszor, egy logikai művelet akár 27 távoli kérést okozhat. Válassz egy felelős réteget, és ismerd az SDK/broker saját retryját. Hasznos lehet szolgáltatási szintű retry budget vagy tokenkeret is; egyenként korlátos hívások együtt még túlterhelhetnek.

Retry-After fogadásakor értelmezd a dokumentált időformátumot, és vesd össze a teljes deadline-nal. Ha a szerver által kért várakozás nem fér bele, ne rövidítsd le csendben azért, hogy mégis retryzhass. A cancellation jelzését ne alakítsd át újabb retryvá.

## Senior döntések és tipikus hibák

Az idempotens GET is okozhat drága számítást; az ismételhetőség nem korlátlan terhelési engedély. Írásnál a ReadTimeout után különösen fontos az eredménybizonytalanság. A retry metrikában a logikai művelet és a fizikai próbák száma különüljön el.

A kód numerikus, validált konfigurációt feltételez; nem publikus konfigurációparser. Tesztben injektált véletlen értéket használunk, nem valódi sleepet vagy véletlen sikert. Valódi rendszerben a jitter mellé megfigyelés és túlterhelési korlát is kell.

## Interview questions

**Why can retries amplify an outage?**

Válaszvázlat: További terhelést küldenek a már lassú függőségre; több retryzó réteg szorzódhat.

**What is the difference between retry safety and retry usefulness?**

Válaszvázlat: Az egyik a duplikált hatás kockázata, a másik a hiba várható átmenetisége és a maradék kapacitás/idő.

## Önellenőrzés

- [ ] A próbálkozáslimitbe az első hívást is beleszámolom.
- [ ] Várakozás után újra számolom a deadline-t.

## Kapcsolódó gyakorlat

[R02 – önálló feladat](../../../exercises/senior-python/11-reliable-services.md#r02)

## Forrás és továbbolvasás

[AWS: backoff and jitter](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)

[Retry-After semantics](https://www.rfc-editor.org/rfc/rfc9110.html#name-retry-after)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
