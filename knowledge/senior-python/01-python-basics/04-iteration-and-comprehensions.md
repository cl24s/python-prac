# Iterálás, generátorok és comprehensionök

## Mit kell tudnod?

- Megkülönböztetni az iterable, iterator és generator fogalmakat.
- Megérteni az egyszeri bejárást, a lazy végrehajtást és a hibák időzítését.
- Olvasható comprehensiont írni, megfelelő konténert választva.

## Magyarázat

Az iterable olyan objektum, amelyből `iter()` segítségével iteratort kérhetünk. Az iterator a `next()` hívásokkal adja az elemeket, végül `StopIteration` jelzi a végét. A lista új bejáráshoz új iteratort ad; egy iterator jellemzően önmagát adja vissza, és egyszer fogyasztható el.

A `yield`-et tartalmazó függvény hívása generator objektumot hoz létre. A törzs a bejárás során fut, a `yield` felfüggeszti, és a helyi állapot megmarad. A generator egy iterator; nem minden iterator generator. Például a lista bejárója is iterator.

```python
values = [4, 7, 10]
iterator = iter(values)
assert iter(iterator) is iterator
assert next(iterator) == 4
assert list(iterator) == [7, 10]
assert list(iterator) == []
assert list(values) == [4, 7, 10]

def valid_latencies(lines):
    for line in lines:
        text = line.strip()
        if text:
            yield int(text)

stream = valid_latencies(['12', '', 'bad'])
assert next(stream) == 12
try:
    next(stream)
except ValueError:
    pass
else:
    raise AssertionError('Expected validation error during iteration')
```

A hibás sor nem a generator létrehozásakor okoz hibát, hanem akkor, amikor a fogyasztó odáig eljut. Egy generator API hibakezelésének ezért a tényleges iterálás körül is működnie kell.

## Comprehension, unpacking és bejárás

```python
records = [('api', 200), ('worker', 503), ('api', 500)]
failed_services = {name for name, status in records if status >= 500}
assert failed_services == {'api', 'worker'}

statuses = [status for _, status in records]
assert statuses == [200, 503, 500]
assert sum(status >= 500 for _, status in records) == 2

first, *rest = statuses
assert first == 200 and rest == [503, 500]
assert list(enumerate(['api', 'worker'], start=1)) == [(1, 'api'), (2, 'worker')]
```

A list comprehension listát épít, a set comprehension deduplikál, a dict comprehension kulcs–érték párokat hoz létre. A generator expression elemeket ad igény szerint. A `*rest` valódi listát készít: nagy iterátoron nem memóriaingyenes.

## Mikor használnád?

Logok szűrésénél a generator jól illeszkedik a feldolgozási lánchoz. Ha többször kell végigmenni az eredményen, választhatsz tárolt listát vagy újranyitható adatforrást. Az egyszer bejárható bemenetet az API szerződésében nevezd meg.

## Tipikus hibák és senior szempontok

- **Hiba:** naplózáshoz `list(stream)`, majd ugyanazzal a streammel dolgozol. **Javítás:** egyetlen bejárás, vagy tudatos materializálás és a kapott lista továbbadása.
- **Hiba:** mellékhatásokra comprehension: `[send(x) for x in jobs]`. **Javítás:** sima ciklus, eredménylista nélkül.
- `zip` alapból a rövidebb bemenet végén leáll. Ha azonos hosszúság kell, a `strict=True` segít a szerződés ellenőrzésében.
- A lazy feldolgozás nem jelent párhuzamosságot, és az összesítő dict vagy egy nyitott erőforrás ettől még memóriát, kapcsolatot tarthat életben.
- Saját iterable-nél általában `__iter__` kell; saját iteratornál `__iter__` és `__next__`. Egyszerű bejáróhoz egy `yield`-es `__iter__` gyakran elég.

## Interview questions

**What is the difference between an iterable and an iterator?**

Válaszvázlat: az iterable bejárót ad; az iterator állapotot tart és sorban szolgáltat elemeket. Hasonlítsd össze a listát és a belőle kapott iteratort.

**When can a generator use more memory than expected?**

Válaszvázlat: megtartott frame és lokális objektumok, downstream gyűjtés, korlátlan aggregáció vagy pufferelés miatt.

## Önellenőrzés

- [ ] Megmondom, mikor fut le egy generator törzse.
- [ ] Felismerem a kétszer bejárt iterator hibáját.
- [ ] Indoklom, listát, setet vagy generátort használok-e.


## Kapcsolódó gyakorlat

[Önálló feladat: B03](../../../exercises/senior-python/01-python-basics.md#b03)

## Forrás és továbbolvasás

[Iterator types](https://docs.python.org/3/library/stdtypes.html#iterator-types) · [Comprehensions](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
