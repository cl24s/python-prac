# Teljesítmény és hibakeresés

Állapot: kidolgozott első változat. Az elsajátítást külön a [ROADMAP](../../../ROADMAP.md) követi.

## Tanulási sorrend

1. [Mérés, latency és a valódi szűk keresztmetszet](01-measurement-and-latency.md)
2. [CPU-profiling és algoritmikus költség](02-cpu-profiling.md)
3. [Memóriaprofil, élő referenciák és korlátlan tárolás](03-memory-and-retention.md)
4. [I/O, pool-várakozás és adatbázis-költség](04-io-and-database-bottlenecks.md)
5. [Lefagyás, threadstack és async taskok vizsgálata](05-hangs-and-stack-diagnostics.md)
6. [Reprezentatív benchmark és kapacitási kompromisszumok](06-benchmarks-and-capacity.md)

## Hogyan dolgozz vele?

1. Fogalmazd meg a fejezet működési vagy hibamodelljét saját szavaiddal.
2. Másold a teljes Python-blokkot külön `test_example.py` fájlba.
3. Futtasd: `python -m pytest -q test_example.py`. A sima `python test_example.py` nem indítja a tesztfüggvényeket.
4. Válaszolj angolul az interjúkérdésekre, majd ellenőrizd a magyar válaszvázlattal.
5. Oldd meg a [kapcsolódó feladatsort](../../../exercises/senior-python/12-performance-and-debugging.md). Kész megoldás nincs mellékelve; review után frissítsük az elsajátítást.

## Futtatókörnyezet

CPython 3.12.13 és pytest 9.1.1 alatt, Linuxon ellenőrzött példák. A példák futási logikája kizárólag standard libraryt használ; pytest a tesztfuttató.

```sh
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
# Linux / WSL: source .venv/bin/activate
python -m pip install pytest==9.1.1
python -m pytest -q test_example.py
```

A példák valódi cProfile/pstats profilt, tracemalloc-allokációt, SQLite-hívásszámot, faulthandler-dumpot és kis timeit-mérést használnak. A benchmark nyers eredményéhez `python -m pytest -q -s test_example.py` futtatás kell. A profil- és dumpfájlok a pytest ideiglenes könyvtárába kerülnek.

Nincs gépfüggetlen gyorsulási küszöb vagy request/sec ígéret. A memória összehasonlítása a két konkrét allokációs minta relatív peakjére vonatkozik, nem RSS-re. A thread- és taskpéldák feloldható várakozást hoznak létre, nem végleges deadlockot. Windows, natív profiler és éles terhelés nem volt ellenőrizve.

A [logelemző CLI](../../../projects/01-log-analyzer/README.md), [endpoint checker](../../../projects/02-endpoint-checker/README.md) és [job API](../../../projects/03-job-processing-api/README.md) később saját mérhető munkaterhelést adnak; a jelen példák nem kész projektmegoldások.

Az assertionök oktatási ellenőrzések. A pontos eredmény és a mérési korlátok az [ellenőrzési jegyzetben](../VALIDATION.md) szerepelnek.

[Teljes tudástérkép](../README.md)
