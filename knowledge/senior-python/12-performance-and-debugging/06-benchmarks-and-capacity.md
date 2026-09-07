# Reprezentatív benchmark és kapacitási kompromisszumok

## Mit kell tudnod?

- Azonos viselkedésű változatokat összehasonlítható körülmények között mérni.
- Mikrobenchmarkot elkülöníteni a terheléspróbától.
- Throughput, latency és memória kompromisszumát indokolni.

## Magyarázat

Egy gyorsabb függvény nem feltétlenül gyorsítja érzékelhetően az egész API-t. Ha a kérés idejének csak 10%-a az adott rész, annak megduplázott sebessége is legfeljebb körülbelül 5%-ot farag a teljes időből az egyszerű, változatlan többi részt feltételező modellben.

A benchmark előtt legyen viselkedési ekvivalencia. A példabeli listás membership és setes előfeldolgozás egész kulcsokat használ. Az előfeldolgozás idejét beleszámítjuk, mert a mérendő művelet itt egy teljes egyszeri szűrés, nem sok híváson át újrahasznált index.

## Kódpélda: ismételt mérés, sebességküszöb nélkül

```python
import timeit

def filter_list(values, allowed):
    return [value for value in values if value in allowed]

def filter_set(values, allowed):
    index = set(allowed)
    return [value for value in values if value in index]

def test_benchmark_harness():
    values, allowed = list(range(1000)), list(range(0, 1000, 3))
    expected = list(range(0, 1000, 3))
    assert filter_list(values, allowed) == filter_set(values, allowed) == expected
    samples = {}
    for name, function in [('list', filter_list), ('set', filter_set)]:
        timer = timeit.Timer(lambda function=function: function(values, allowed))
        samples[name] = [duration / 5 for duration in timer.repeat(repeat=3, number=5)]
        assert len(samples[name]) == 3
        assert all(value >= 0 for value in samples[name])
    print({'seconds_per_call': samples})
```

A mérést `python -m pytest -q -s test_example.py` paranccsal látod. A teszt nem vár adott gyorsulási arányt, ezért a gép terhelése nem okoz önkényes „túl lassú” bukást. A lokális mérés nem szolgáltatási request/sec ígéret. A timeit alapból kikapcsolja a garbage collectort a mérés idejére; GC-intenzív munkánál tudatosan állítsd a mérési feltételeket.

## Terheléspróba terve

| Rögzítendő adat | Miért számít? |
| --- | --- |
| Verzió, CPU/memória limit, worker/pool | Azonos környezet nélkül nehéz összehasonlítani |
| Payload-eloszlás, cache-hit arány | A „könnyű” kérések torzíthatják az eredményt |
| Érkezési ráta és konkurencia | Nem ugyanaz a két terhelési modell |
| p50/p95/p99, hibák és timeoutok | Throughput önmagában elrejti a romló szolgáltatást |
| Queue-kor, DB/pool-várakozás, RSS | A stabilitás és telítődés jelei |

Zárt terhelésgenerátor a válasz után küldi a következő kérést; lassuláskor maga is ritkábban küldhet, így a valós érkezési nyomást alábecsülheti. Nyílt érkezési modellnél a tervezett rátát külön tartod, és a generátor saját limitjét is méred. A coordinated omission veszélyét ezért a load tool időmérésével együtt kell vizsgálni.

## Kapacitás és senior döntések

Stabil állapotban, azonos rendszerhatáron az átlagos bent lévő munka közelíthető érkezési/átbocsátási ráta × átlagos bent töltött idő szorzatával. Saját példa: 100 befejezett kérés/s és 0,2 s átlagos teljes idő körülbelül 20 in-flight kérést jelent. Ez nem szükséges threadszerver-szám és nem 20 DB-kapcsolat; a határ és várakozások számítanak.

Növekvő queue mellett nincs ilyen stabil becslés. A nagyobb batch javíthatja a throughputot, de rontja a várakozást és emeli a memóriát. Először az elfogadható latencyt és hibaarányt mondd ki, és azon belül keresd a fenntartható kapacitást.

## Tipikus hibák

Egyetlen futásból vagy eltérő bemenetből ne állíts gyorsulást. A bemelegített cache-t ne hasonlítsd hideg rendszerhez jelölés nélkül. CPU-profilerrel együtt mért időt ne keverd a profiler nélküli baseline-nal. A gyorsabb, de más eredményt adó algoritmus nem ugyanannak a szerződésnek az optimalizálása.

## Interview questions

**Why is a microbenchmark not a throughput estimate for an API?**

Válaszvázlat: Nem modellezi a hálózatot, adatbázist, queue-t, konkurenciát és reprezentatív requestkeveréket.

**Can higher throughput be a regression?**

Válaszvázlat: Igen, ha közben nő a tail latency, hiba, memória vagy várakozósor az elfogadható határ fölé.

## Önellenőrzés

- [ ] A setup/preprocessing mérési határát explicit választom.
- [ ] Nem vezetek le thread- vagy poolméretet pusztán átlagos in-flight számból.

## Kapcsolódó gyakorlat

[P06 – önálló feladat](../../../exercises/senior-python/12-performance-and-debugging.md#p06)

## Forrás és továbbolvasás

[timeit](https://docs.python.org/3.12/library/timeit.html)

[Google SRE overload behavior](https://sre.google/sre-book/handling-overload/)

[wrk2 and coordinated omission](https://github.com/giltene/wrk2)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
