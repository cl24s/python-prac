# Deque, Counter, defaultdict és heap

## Mit kell tudnod?

- Felismerni, mikor ad jobb modellt egy specializált standard library eszköz.
- Megkülönböztetni a FIFO-sorrendet és a prioritás szerinti sorrendet.
- Kezelni az azonos prioritást és a tároló kapacitását.

## Eszközválasztás

| Eszköz | Tipikus helyzet | Fontos korlát |
| --- | --- | --- |
| `deque` | Sor elejéről kivétel, végére betétel | Középső indexelés nem listasebességű |
| `Counter` | Előfordulások számlálása | Memóriaigény az egyedi kulcsokkal nő |
| `defaultdict` | Csoportok fokozatos felépítése | `d[key]` hiány esetén módosítja a dictet |
| `heapq` | Mindig a legkisebb prioritás kell | A belső lista nem teljesen rendezett |

## Kódpélda: queue és aggregáció

```python
from collections import Counter, defaultdict, deque

queue = deque(['job-a', 'job-b'])
queue.append('job-c')
assert queue.popleft() == 'job-a'
assert list(queue) == ['job-b', 'job-c']

recent = deque(maxlen=2)
recent.extend(['old', 'middle', 'new'])
assert list(recent) == ['middle', 'new']

counts = Counter(['api', 'api', 'worker'])
assert counts['api'] == 2
assert counts['unknown'] == 0
assert 'unknown' not in counts

groups = defaultdict(list)
for team, service in [('core', 'api'), ('core', 'auth')]:
    groups[team].append(service)
assert groups['core'] == ['api', 'auth']
assert groups.get('missing') is None
assert 'missing' not in groups
assert groups['missing'] == []
assert 'missing' in groups
```

A `maxlen` automatikusan eldob régi elemet a másik végéről. Ez jó utolsó N eseményhez, de el nem veszthető feladatqueue-hoz veszélyes. A `defaultdict` factory-je például `list`, nem `list()`: hívható objektum kell.

## Kódpélda: prioritás és determinisztikus holtverseny

```python
import heapq
from itertools import count

sequence = count()
heap = []
heapq.heappush(heap, (2, next(sequence), {'id': 'a'}))
heapq.heappush(heap, (1, next(sequence), {'id': 'b'}))
heapq.heappush(heap, (1, next(sequence), {'id': 'c'}))

order = [heapq.heappop(heap)[2]['id'] for _ in range(3)]
assert order == ['b', 'c', 'a']
```

A tuple-ök lexikografikusan hasonlítódnak össze: először prioritás, majd egyedi sorszám. Enélkül azonos prioritásnál a nem rendezhető dict payload összehasonlítása `TypeError`-hoz vezethetne. A sorszám egyben stabil beküldési sorrendet ad a holtversenyeknek.

## Mikor használnád és mire figyelj?

- **Hiba:** `heap[0]` után a többi elemet is rendezettnek gondolod. **Javítás:** a heap csak a legkisebb gyökérelemet és a heap-invariánst garantálja; rendezett kivételhez ismételt pop kell.
- **Hiba:** a heapben lévő elem prioritását helyben megváltoztatod. **Javítás:** új bejegyzés és régi érvénytelenítés, vagy kontrollált újraépítés.
- Egy deque nem biztosít teljes producer–consumer protokollt. Több lépés atomikusságát, várakozást és backpressure-t külön kell kezelni; erre `queue.Queue` vagy `asyncio.Queue` lehet jobb.
- Korlátlan `Counter` nagy kardinalitású azonosítókkal ugyanúgy elfogyaszthatja a memóriát, mint bármely dict.

## Interview questions

**Why is deque a better FIFO queue than repeatedly calling list.pop(0)?**

Válaszvázlat: a deque végein végzett műveletek elkerülik az összes hátralévő referencia eltolását.

**Why put a sequence number into a priority queue entry?**

Válaszvázlat: determinisztikus holtverseny és a payload összehasonlításának elkerülése.

## Önellenőrzés

- [ ] Tudom, mikor veszíthet elemet egy korlátos deque.
- [ ] Megmagyarázom a `defaultdict.get()` és `[]` különbségét.
- [ ] Azonos prioritású dict payloadokkal is működő heapet tervezek.


## Kapcsolódó gyakorlat

[Önálló feladat: D03](../../../exercises/senior-python/02-data-structures.md#d03)

## Forrás és továbbolvasás

[collections](https://docs.python.org/3/library/collections.html) · [heapq](https://docs.python.org/3/library/heapq.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
