# Rendezés, keresés, csoportosítás és deduplikáció

## Mit kell tudnod?

- Stabil és determinisztikus rendezést tervezni.
- Megkülönböztetni a rendezett keresést és a beszúrás költségét.
- Érteni a `groupby` előfeltételeit és a duplikációs szabályokat.

## Rendezés és holtverseny

A `sorted()` új listát készít egy iterable-ből; a `list.sort()` a listát helyben módosítja. Mindkettő stabil: az azonos kulcsú elemek eredeti relatív sorrendje megmarad. Ez nem feltétlenül teljes determinisztikusság: ha a bemeneti sorrend bizonytalan, explicit másodlagos kulcs kell.

```python
records = [
    {'id': 'c', 'errors': 2},
    {'id': 'b', 'errors': 5},
    {'id': 'a', 'errors': 5},
]
by_errors = sorted(records, key=lambda row: -row['errors'])
assert [row['id'] for row in by_errors] == ['b', 'a', 'c']
ranked = sorted(records, key=lambda row: (-row['errors'], row['id']))
assert [row['id'] for row in ranked] == ['a', 'b', 'c']
assert [row['id'] for row in records] == ['c', 'b', 'a']
```

A key egyszer kerül kiszámításra elemenként az adott rendezésben. Drága hálózati lekérdezés még így sem jó rendezési kulcs: az adatbeszerzést és a tiszta rendezést válaszd külön. A hiányzó mező és a `None` helyét explicit szabályozd.

## Bináris keresés

```python
from bisect import bisect_left

latencies = [10, 20, 20, 40]
position = bisect_left(latencies, 20)
assert position == 1
assert latencies[position] == 20

missing = bisect_left(latencies, 30)
assert missing == 3
assert missing == len(latencies) or latencies[missing] != 30
```

A bisect rendezett adaton beszúrási helyet ad, nem tagságot. A végére esés és az elem egyenlősége külön ellenőrzendő. A keresés logaritmikus lehet, de listába beszúrni az eltolás miatt továbbra is O(n). Egyszeri keresés kedvéért rendezni gyakran drágább, mint végigmenni az adaton.

## Csoportosítás és deduplikáció

```python
from itertools import groupby

rows = [('api', 1), ('worker', 2), ('api', 3)]
consecutive = [(key, list(group)) for key, group in groupby(rows, key=lambda x: x[0])]
assert [key for key, _ in consecutive] == ['api', 'worker', 'api']

ordered = sorted(rows, key=lambda x: x[0])
grouped = {key: [value for _, value in group]
           for key, group in groupby(ordered, key=lambda x: x[0])}
assert grouped == {'api': [1, 3], 'worker': [2]}
assert list(dict.fromkeys(['worker', 'api', 'worker'])) == ['worker', 'api']
```

A `groupby` egymás melletti azonos kulcsokat fog össze, nem SQL-szerű globális csoportosítás. Ha teljes csoport kell, rendezz azonos kulccsal, vagy használj dict-alapú gyűjtést. A csoportiterátor osztja az alapadatforrást a külső bejáróval: fogyaszd el a csoportot, mielőtt továbblépsz.

## Senior szempontok és tipikus hibák

- **Hiba:** rendezés nélkül a `groupby` eredményét dictbe írod; egy későbbi azonos kulcs felülírhatja a korábbi csoportot. **Javítás:** tudatos előrendezés vagy dict-aggregáció.
- API-lapozásnál holtversenyt feloldó stabil azonosító is kellhet, különben rekordok ugrálhatnak az oldalak között.
- „Duplikáció eltávolítása” előtt tisztázd: azonos ID vagy teljes érték számít, első vagy utolsó előfordulás marad?
- Nagy adathalmaz teljes rendezése materializálást igényelhet; streaming forrásra önmagában a `sorted()` nem memóriahatékony megoldás.

## Interview questions

**Does stable sorting guarantee deterministic API output?**

Válaszvázlat: csak a bemeneti holtverseny-sorrendet őrzi. Bizonytalan bemenetnél teljes rendezési kulcs kell.

**How does itertools.groupby differ from SQL GROUP BY?**

Válaszvázlat: egymást követő csoportok, közös iterator; globális csoporthoz rendezés vagy aggregáló tároló kell.

## Önellenőrzés

- [ ] Rendezést tervezek explicit holtversennyel.
- [ ] A bisect eredményét nem keverem a sikeres találattal.
- [ ] Elmagyarázom, miért keletkezik három csoport az első példában.


## Kapcsolódó gyakorlat

[Önálló feladat: D05](../../../exercises/senior-python/02-data-structures.md#d05)

## Forrás és továbbolvasás

[Sorting HOWTO](https://docs.python.org/3/howto/sorting.html) · [bisect](https://docs.python.org/3/library/bisect.html) · [itertools.groupby](https://docs.python.org/3/library/itertools.html#itertools.groupby)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
