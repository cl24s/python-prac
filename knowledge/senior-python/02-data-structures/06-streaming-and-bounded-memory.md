# Nagy adatok és korlátos memóriahasználat

## Mit kell tudnod?

- Egyetlen bejárással feldolgozni a bemenetet.
- A bemeneti sorok száma mellett az egyedi kulcsok számát is figyelni.
- Különválasztani a streaminget, a bounded memory-t és a párhuzamosságot.

## Magyarázat

A streaming azt jelenti, hogy az inputot fokozatosan dolgozod fel. Nem következik belőle konstans memóriahasználat. Egy generatorral olvasott fájl összes egyedi request ID-jét setbe gyűjtve továbbra is a fájl méretével arányos memória kellhet.

Mindig számold meg a megtartott állapotot: aktuális rekord, batch, aggregációs kulcsok, kimeneti puffer és nyitott erőforrás. A soronkénti olvasás is egy teljes sort tárolhat; egy hatalmas sor kezeléséhez további méretkorlát vagy daraboló parser kell.

## Kódpélda: egy menetben végzett aggregáció

A példa CSV-sorokat kap, és szolgáltatásonként átlagos latency-t számol. Ez szemléltető példa; az önálló gyakorlat más formátumot és további szerződést kér.

```python
import csv
from io import StringIO

def summarize(source):
    totals = {}
    invalid = 0
    for row in csv.reader(source):
        if len(row) != 2 or not row[0].strip():
            invalid += 1
            continue
        try:
            latency = int(row[1])
        except ValueError:
            invalid += 1
            continue
        if latency < 0:
            invalid += 1
            continue
        service = row[0].strip()
        count, total = totals.get(service, (0, 0))
        totals[service] = (count + 1, total + latency)
    averages = {key: total / count for key, (count, total) in totals.items()}
    return averages, invalid

source = StringIO('api,10\nworker,30\napi,20\napi,bad\n')
averages, invalid = summarize(source)
assert averages == {'api': 15.0, 'worker': 30.0}
assert invalid == 1
assert summarize(StringIO('')) == ({}, 0)
```

A `StringIO` tesztbemenet már memóriában van; éles használatnál nyitott szöveges fájl is átadható, amelyet a hívó `with` blokkja birtokol. Az algoritmus állapota k egyedi szolgáltatásnál O(k), nem O(1). A példa a CSV parser hibáit továbbengedi; a mezőszintű validáció hibáit számolja.

## Batch és top-k

```python
from itertools import islice
import heapq

def batches(source, size):
    if size <= 0:
        raise ValueError('size must be positive')
    iterator = iter(source)
    while True:
        batch = list(islice(iterator, size))
        if not batch:
            return
        yield batch

assert list(batches(iter(range(5)), 2)) == [[0, 1], [2, 3], [4]]
latencies = (value for value in [10, 90, 20, 70, 30])
assert heapq.nlargest(2, latencies) == [90, 70]
```

B méretű batch a teljes input helyett legfeljebb B elemet ad egyszerre a fogyasztónak. Ha a fogyasztó az összes batch-et listába gyűjti, ez az előny elveszik; a példabeli `list(...)` csak a kicsi teszt kimenetét ellenőrzi. Top-k kérdésnél egy k méretű heap gyakran jobb, mint mindent rendezni; k nagyságát és a konkrét implementáció útját is mérlegeld.

## Tipikus hibák és senior szempontok

- **Hiba:** `readlines()` vagy `sorted(stream)` egy több GB-os inputon. **Javítás:** soronkénti feldolgozás, szükség esetén külső rendezés vagy adatbázis.
- Az exact globális deduplikáció emlékezetet igényel az eddigi kulcsokról. Ha ez túl nagy, lemezalapú index, partícionálás vagy üzletileg elfogadott közelítés kell.
- Az `itertools.tee` tárolhatja a lemaradó fogyasztónak az adatot; nagy eltérésnél nagy puffer lesz.
- Generatorból kilógó fájléletciklus helyett a hívó birtokolja az erőforrást, vagy explicit context manager szerződés legyen.
- Háttérqueue esetén a termelő gyorsabb lehet a fogyasztónál; a korlátos queue és backpressure külön concurrency-téma.

## Interview questions

**Why is a streaming aggregation not necessarily constant-memory?**

Válaszvázlat: a különböző kulcsok és a felhalmozott eredmény állapota nőhet; n input, k egyedi kulcs és B batch külön mennyiségek.

**How would you process a file larger than available memory?**

Válaszvázlat: fokozatos olvasás, korlátos állapot és kimenet; szükség esetén külső rendezés/tárolás; hibapolitika és erőforrás-életciklus rögzítése.

## Önellenőrzés

- [ ] Megadom az aggregáció O(k) állapotigényét.
- [ ] Az empty input és hibás rekord viselkedését is definiálom.
- [ ] Felismerem a pipeline-ban a rejtett materializálást.


## Kapcsolódó gyakorlat

[Önálló feladat: D06](../../../exercises/senior-python/02-data-structures.md#d06)

## Forrás és továbbolvasás

[itertools](https://docs.python.org/3/library/itertools.html) · [csv](https://docs.python.org/3/library/csv.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
