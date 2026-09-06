# Memória, elérhetőség és objektuméletciklus

## Mit kell tudnod?

- Különválasztani a Python nyelvi modelljét a CPython megvalósításától.
- Megérteni, miért nem old meg egy `gc.collect()` minden memória-problémát.
- Felismerni a megtartott referenciákat és a külső erőforrások eltérő életciklusát.

## Magyarázat

Egy objektum addig szükséges a program számára, amíg elérhető. Referenciát nemcsak változó tarthat rá, hanem konténer, closure, generator frame vagy exception traceback is. A név törlése ezek közül csak egy kapcsolatot szüntet meg.

CPythonban a referenciaszámlálás és a ciklikus garbage collector együtt vesz részt a memóriakezelésben. Az azonnali felszabadításra azonban ne építs nyelvi garanciaként: más implementáció és más build eltérhet. A GC elérhetetlenné vált ciklusokkal is foglalkozik, de nem törölheti az alkalmazás által még elérhető adatot csak azért, mert az túl sok.

## Kódpélda: a probléma egy élő referencia

```python
from dataclasses import dataclass
import gc
import weakref

@dataclass
class Report:
    rows: list

report = Report([1, 2, 3])
reference = weakref.ref(report)
cache = {'latest': report}
del report
gc.collect()
assert reference() is cache['latest']

cache.clear()
gc.collect()
assert reference() is None
```

A weak reference nem tartja életben az objektumot; a cache viszont igen. Az első gyűjtés helyesen nem töröl semmit. A példa CPythonon ellenőrzött; más runtime-ban a tényleges gyűjtési időzítés eltérhet.

## Ciklus és takarítás

```python
from dataclasses import dataclass
import gc
import weakref

@dataclass
class Node:
    next: object = None

node = Node()
node.next = node
reference = weakref.ref(node)
del node
gc.collect()
assert reference() is None
```

A saját magára mutató kapcsolat nem teszi hasznossá az objektumot. A ciklus a program többi részéből már elérhetetlen. A kézi gyűjtést itt szemléltetésre használjuk, nem kérésenkénti rutin optimalizációként.

## Mikor használnád ezt a tudást?

Egy worker órák alatt egyre több memóriát foglal. Először vizsgáld, korlátos-e a cache, a queue, a felhalmozott eredménylista és a naplópuffer. A `tracemalloc` snapshotokkal a Python-allokációk változása kereshető; a processz RSS ennél tágabb, natív allokációkat és allocator-viselkedést is tükrözhet.

## Tipikus hibák és senior szempontok

- **Hiba:** minden kérés után `gc.collect()`. **Javítás:** az adatot megtartó tulajdonost és a növekedés forrását keresd; a gyűjtés késleltetést is okoz.
- **Hiba:** `sys.getsizeof(container)` alapján teljes mély memóriaigényt állítasz. **Javítás:** a közvetlen méret nem számolja össze automatikusan a beágyazott objektumokat.
- Felszabadított objektum után az RSS nem feltétlenül csökken azonnal; az allocator megtarthat memóriaterületeket.
- Fájllezárásra `with`, nem `__del__` a megfelelő alap. A finalizer időzítése és hibakezelése nem helyettesít explicit életciklust.
- Weak reference nem általános cache-pótlék: az adat eltűnhet, és nem minden típus támogatja.

## Interview questions

**Why does forcing garbage collection not fix an unbounded cache?**

Válaszvázlat: a cache-ből még elérhetők az objektumok; törlési/lejárati vagy kapacitáskorlát kell.

**Why can process memory remain high after objects are released?**

Válaszvázlat: objektuméletciklus és operációs rendszernek visszaadott memória eltérő réteg; allocator és natív könyvtárak is számítanak.

## Önellenőrzés

- [ ] Különbséget teszek élő referencia és elérhetetlen ciklus között.
- [ ] Nem ígérek azonnali felszabadítást minden Python-runtime-ra.
- [ ] Tudok három olyan komponenst mondani, amely tartósan megtarthat adatot.


## Kapcsolódó gyakorlat

[Önálló feladat: B06](../../../exercises/senior-python/01-python-basics.md#b06)

## Forrás és továbbolvasás

[gc](https://docs.python.org/3/library/gc.html) · [tracemalloc](https://docs.python.org/3/library/tracemalloc.html) · [weakref](https://docs.python.org/3/library/weakref.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
