# CPU-profiling és algoritmikus költség

## Mit kell tudnod?

- cProfile eredményt hívási költségként olvasni.
- Saját időt és kumulatív időt megkülönböztetni.
- Algoritmikus javítást a profiler alapján megindokolni.

## Magyarázat

A profiler azt segít megtalálni, hol telik az idő vagy hol halmozódnak a hívások. A cProfile determinisztikus profiler: függvényhívásokhoz kötött eseményeket rögzít, így mérési overheadet is okoz. A profil futtatása nem ugyanaz, mint a reprezentatív benchmark.

A tottime az adott függvény saját ideje a hívott függvények nélkül; a cumtime a belső hívásokat is tartalmazza. Egy orchestrator nagy cumtime-mal és kis saját idővel lehet teljesen rendben. Ne add össze vakon az egymásba ágyazott kumulatív időket: többször számolnád ugyanazt.

## Kódpélda: profil mentése és újraolvasása

```python
import cProfile
import pstats
from io import StringIO

def transform(value):
    return sum(i * i for i in range(value))

def workload():
    return sum(transform(100) for _ in range(20))

def test_profile_records_calls(tmp_path):
    profiler = cProfile.Profile()
    result = profiler.runcall(workload)
    assert result == 20 * 328350
    path = tmp_path / 'cpu.prof'
    profiler.dump_stats(str(path))
    output = StringIO()
    stats = pstats.Stats(str(path), stream=output)
    stats.sort_stats('cumulative').print_stats(10)
    entries = [entry for key, entry in stats.stats.items() if key[2] == 'transform']
    assert len(entries) == 1
    primitive_calls, total_calls, own_time, cumulative_time, callers = entries[0]
    assert primitive_calls == total_calls == 20
    assert cumulative_time >= own_time >= 0
    assert 'transform' in output.getvalue()
```

A teszt valós profilfájlt hoz létre és olvas vissza. Nem állít gyorsulási arányt vagy hardverfüggetlen futásidőt. A függvényhívások száma stabil elvárás ezen a mintán, a másodpercben mért idő nem az.

## Hipotézis és javítás

Ha egy drága transzformáció ugyanarra a bemenetre ismétlődik, lehet indokolt egyszer kiszámolni és újrahasználni. De előbb ellenőrizd, hogy a függvény tiszta-e: időfüggő, jogosultságfüggő vagy mellékhatásos műveletet nem cache-elhetsz pusztán hívásszám alapján.

Gyakori hiba az O(n²) keresés belső ciklusban. Megfelelő hash-map vagy egyszeri előfeldolgozás többet érhet, mint a lokális változók mikrooptimalizálása. A jobb algoritmus azonban más memóriaigényt vagy bemeneti korlátot hozhat; a szerződést is újra ellenőrizd.

## Senior döntések és tipikus hibák

A profil azon a bemeneten érvényes, amelyen futott. Egy kis, cache-be férő minta más szűk keresztmetszetet mutathat, mint a nagy éles adat. Többprocesszes alkalmazásnál egy worker profilja nem a teljes deployment profilja.

Sampling profiler kisebb beavatkozással adhat stackmintákat, de a rövid eseményeket kihagyhatja. Natív extension, GIL-várakozás és async ütemezés értelmezéséhez az eszköz határait ismerni kell. Ne tedd a profiler overheadjét a felhasználó elvárt latencyjének részévé.

## Interview questions

**What is the difference between tottime and cumtime?**

Válaszvázlat: A saját idő nem tartalmazza a belső hívásokat, a kumulatív igen; ezért a nagy cumtime nem feltétlenül helyi számítás.

**Why is profiling not benchmarking?**

Válaszvázlat: A profiler beavatkozik a futásba, célja a költség helyének keresése; a gyorsulást külön, reprezentatív méréssel kell igazolni.

## Önellenőrzés

- [ ] A hívásszámot és a bemenetet is vizsgálom.
- [ ] A profilból nem következtetek automatikusan cache-elhetőségre.

## Kapcsolódó gyakorlat

[P02 – önálló feladat](../../../exercises/senior-python/12-performance-and-debugging.md#p02)

## Forrás és továbbolvasás

[Python profilers and pstats](https://docs.python.org/3.12/library/profile.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
