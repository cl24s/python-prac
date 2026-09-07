# Mérés, latency és a valódi szűk keresztmetszet

## Mit kell tudnod?

- Optimalizálás előtt megfigyelhető problémát megfogalmazni.
- Wall time, CPU-idő, throughput és hibaarány különbségét ismerni.
- Percentilist helyes populáción és módszerrel értelmezni.

## Magyarázat

A „lassú” nem diagnózis. Például: a joblista p95 válaszideje tízszeres lett, csak cache-miss esetén, változatlan forgalomnál. Ebből célzottan vizsgálhatod a DB-lekérdezést, pool-várakozást vagy a külső hívást. A teljes újraírás előtt legyen reprodukálható megfigyelés és hipotézis.

Wall time a felhasználó által megélt eltelt idő; CPU-idő a processz aktív CPU-használata, amely a párhuzamos threadek idejét is összesítheti. Alacsony CPU és hosszú latency gyakran várakozásra utal, de egyetlen szám nem bizonyítja az okot.

| Jel | Következő kérdés |
| --- | --- |
| Magas CPU | Mely függvény, milyen bemenet, milyen processz? |
| Alacsony CPU, magas latency | Hálózat, lock, DB vagy pool várakozik? |
| Növekvő memória | Élő referenciák, cache, batch, natív allokáció? |
| Növekvő queue-kor | Beérkezés meghaladja a feldolgozást? |
| Jó átlag, rossz p99 | Kevés nagyon lassú eset vagy elrejtett timeout? |

## Kódpélda: nearest-rank percentilis explicit definícióval

```python
import math
import pytest

def percentile(values, p):
    if not values or not 0 < p <= 100:
        raise ValueError('nonempty sample and percentile in (0, 100] required')
    ordered = sorted(values)
    return ordered[math.ceil(p / 100 * len(ordered)) - 1]

def test_tail_is_not_the_average():
    latencies_ms = [10] * 95 + [1000] * 5
    assert sum(latencies_ms) / len(latencies_ms) == 59.5
    assert percentile(latencies_ms, 95) == 10
    assert percentile(latencies_ms, 99) == 1000
    with pytest.raises(ValueError):
        percentile([], 99)
```

Ez szintetikus adatsor, nem a projekt mért teljesítménye. A percentilismódszer explicit nearest-rank; más könyvtár interpolálhat, kis mintán eltérő értékkel. A függvény előzetesen ellenőrzött, véges numerikus latencyadatokat feltételez.

## Senior döntések és tipikus hibák

Ne átlagold egyszerűen több worker p99 értékét. Az aggregált percentilishez a minták eloszlása vagy összevonható histogram kell. A timeoutokat és hibákat ne dobd ki azért, hogy jobb legyen a sikeres kérések latencyje; külön hibaarány és sikeres-vs-hibás populáció szükséges.

A mérési ablak, endpoint, payloadméret, cache-állapot, konkurencia és verzió legyen rögzítve. A 100%-os összesített CPU félrevezető lehet eltérő CPU-limit mellett, egyetlen telített thread pedig elveszhet a sokmagos átlagban.

Mérj előtte, változtass egy indokolt dolgot, mérj utána azonos feltételekkel. Különítsd el a hideg indulást a bemelegedett állapottól. A sikerfeltétel üzleti legyen: gyorsabb válasz a helyesség és elfogadható erőforrásigény mellett.

## Interview questions

**Why can an average hide a production incident?**

Válaszvázlat: Néhány nagyon lassú kérés elveszhet az átlagban, a hibás/timeoutra futott eseteket pedig ki is zárhatta a mérés.

**Can you average p99 values across workers?**

Válaszvázlat: Nem általánosan; a teljes eloszlás vagy összevonható histogram alapján kell aggregálni.

## Önellenőrzés

- [ ] A mintapopulációt és mérési ablakot rögzítem.
- [ ] A szintetikus számot nem nevezem éles benchmarknak.

## Kapcsolódó gyakorlat

[P01 – önálló feladat](../../../exercises/senior-python/12-performance-and-debugging.md#p01)

## Forrás és továbbolvasás

[Python timing clocks](https://docs.python.org/3.12/library/time.html)

[Prometheus histograms and quantiles](https://prometheus.io/docs/practices/histograms/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
