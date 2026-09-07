# Memóriaprofil, élő referenciák és korlátlan tárolás

## Mit kell tudnod?

- Peak allokációt elkülöníteni a tartósan megmaradó memóriától.
- tracemalloc és processz RSS hatókörét megkülönböztetni.
- Batch, generator és cache memóriaigényét megtervezni.

## Magyarázat

A növekvő memória nem mindig klasszikus leak. Lehet nagyobb bemenet, később ürülő queue, szándékos cache vagy allocator által megtartott memória. Pythonban egy objektum akkor is él, ha nincs rá szükséged, amennyiben egy lista, closure, task vagy cache még referálja.

A tracemalloc a követett Python-allokációkat mutatja; nem feltétlenül látja az összes natív könyvtári foglalást vagy a teljes RSS-t. Két snapshot különbsége megmutathatja, hol nő a megtartott adat. A peak rövid életű allokációs csúcsot is tartalmazhat.

## Kódpélda: teljes lista és stream csúcsallokációja

```python
import tracemalloc

def buffered():
    chunks = [bytearray(4096) for _ in range(256)]
    return sum(len(chunk) for chunk in chunks)

def streamed():
    return sum(len(bytearray(4096)) for _ in range(256))

def measure_peak(operation):
    tracemalloc.start()
    try:
        result = operation()
        current, peak = tracemalloc.get_traced_memory()
        return result, peak
    finally:
        tracemalloc.stop()

def test_stream_avoids_full_buffer():
    result_a, peak_a = measure_peak(buffered)
    result_b, peak_b = measure_peak(streamed)
    assert result_a == result_b == 256 * 4096
    assert peak_a > peak_b
```

Ez kontrollált, megközelítőleg 1 MiB payloadot egyben tartó lista és egyenként feldolgozó változat összehasonlítása. A relatív peak különbségét ellenőrizzük, nem konkrét byte-értéket és nem RSS-csökkenést. A példa a blokkok hosszát összegzi; nem végez fájl-I/O-t.

## Diagnosztikai munkamenet

1. Rögzíts terhelési szakaszokat: indulás, bemelegítés, ismételt munka, nyugalmi idő.
2. Mérj RSS-t és Python-allokációt külön; nézd, melyik nő.
3. Hasonlíts össze snapshotokat azonos fázisban, a jelentős size_diff és count_diff helyekre fókuszálva.
4. Keresd a megtartó tulajdonost: cache-kulcs, globális lista, queue, task vagy callback.
5. Javítás után ugyanazon munkaciklust ismételd; ellenőrizd a helyességet és a memória trendjét.

## Senior döntések és tipikus hibák

A del nem garantál azonnali RSS-csökkenést. A gc.collect nem oldja meg az élő referenciák által megtartott objektumokat. Egy korlátlan lru_cache(maxsize=None) is lehet a „leak” oka, ha a bemeneti kulcsok száma korlátlan.

A generator önmagában sem garancia: list(generator) újra materializál, egy closure nagy objektumot tarthat, a downstream pedig gyűjtheti az összes eredményt. Az egész feldolgozási láncot nézd. Batchméret választásakor memória, tranzakcióhossz és throughput együtt számít.

A queue elemszámkorlátja sem fix byte-korlát, ha egy üzenet 1 KiB és 100 MiB is lehet. Nagy payloadhoz méretlimit, hivatkozásos tárolás vagy külön erőforráskeret kell.

## Interview questions

**Why might RSS stay high after objects are freed?**

Válaszvázlat: Az allocator vagy natív könyvtár megtarthat memóriát; a Python-objektumélet és az OS-nek visszaadott memória külön mérés.

**Does using a generator guarantee bounded memory?**

Válaszvázlat: Nem; a fogyasztó materializálhat, a generator pedig referenciákat tarthat. A teljes láncot kell vizsgálni.

## Önellenőrzés

- [ ] A peak, retained Python memory és RSS külön fogalom.
- [ ] A cache és queue tényleges payloadméretét is mérem.

## Kapcsolódó gyakorlat

[P03 – önálló feladat](../../../exercises/senior-python/12-performance-and-debugging.md#p03)

## Forrás és továbbolvasás

[tracemalloc](https://docs.python.org/3.12/library/tracemalloc.html)

[functools cache behavior](https://docs.python.org/3.12/library/functools.html#functools.lru_cache)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
