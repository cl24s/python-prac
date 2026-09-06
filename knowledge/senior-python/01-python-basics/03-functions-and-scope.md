# Függvények, argumentumok, scope és closure

## Mit kell tudnod?

- Olvasni a positional-only, keyword-only, `*args`, `**kwargs` paramétereket.
- Elkerülni a mutable default és a late binding hibákat.
- Megkülönböztetni a helyi, környező függvénybeli, modul- és built-in neveket.

## Függvényszerződés

A jó szignatúra jelzi, mely argumentumok kötelezők és mely kapcsolókat érdemes név szerint megadni. A `/` előtti paraméterek csak pozícióval, a `*` utániak csak névvel adhatók meg. A `*args` tuple-be gyűjti a maradék pozicionális, a `**kwargs` dictbe a név szerinti argumentumokat. Ezek nem indokolják egy ismert API szerződésének elrejtését.

```python
def build_target(host, /, *, port=443, secure=True):
    scheme = 'https' if secure else 'http'
    return f'{scheme}://{host}:{port}'

options = {'port': 8443}
assert build_target('api.local', **options) == 'https://api.local:8443'

def collect(*items, **metadata):
    return items, metadata

assert collect('api', 'worker', region='eu') == (
    ('api', 'worker'), {'region': 'eu'}
)
```

## Mutable default: a létrehozás időpontja számít

A default érték a `def` végrehajtásakor keletkezik. Nem minden híváskor kapunk új listát. Ez egy hosszú életű szolgáltatásban különböző kérések adatainak összekeveredését okozhatja.

```python
def bad_add(item, items=[]):
    items.append(item)
    return items

first = bad_add('api')
second = bad_add('worker')
assert first is second
assert first == ['api', 'worker']

def add(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

assert add('api') == ['api']
assert add('worker') == ['worker']
```

A javított függvény továbbra is módosítja az explicit átadott listát. Ez szerződésbeli döntés; ha ezt sem akarod, másold le. Ha a `None` érvényes üzleti érték, saját sentinel különítheti el a kihagyott argumentumot.

## Scope és closure

Hagyományos függvényeknél hasznos a LEGB sorrend: Local, Enclosing, Global, Builtins. Egy függvényen belüli értékadás általában helyivé teszi a nevet az egész függvénytörzsben. Korábbi olvasása ekkor `UnboundLocalError`-t eredményezhet. A `global` a modulbeli, a `nonlocal` a legközelebbi környező függvény már létező kötését engedi újrakötni. Osztálytörzsek és modern annotation scope-ok külön szabályait nem érdemes a LEGB rövidítésből levezetni.

```python
def make_counter():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment

counter = make_counter()
assert (counter(), counter()) == (1, 2)

callbacks = [lambda: index for index in range(3)]
assert [callback() for callback in callbacks] == [2, 2, 2]
fixed = [lambda index=index: index for index in range(3)]
assert [callback() for callback in fixed] == [0, 1, 2]
```

A closure a környező kötést éri el; a callback hívásakor annak akkori értékét látja. A default argumentumos változat a callback létrehozásakor tárolja el az értéket. Ez nem a lambda külön hibája: belső `def` esetén is előfordul.

## Mikor használnád és mire figyelj?

Callback-regisztráció, egyszerű függvénygyár és konfigurált feldolgozó készítésekor hasznos a closure. Ha sok módosítható állapotot és műveletet rejt, egy osztály érthetőbb lehet. Modulglobális állapot tesztek között is megmaradhat; a `global` nem konkurenciakezelés.

Tipikus hiba a `timeout = timeout or 30`: a szándékosan megadott nullát is felülírja. Hiányzó paraméterre külön ellenőrzést használj. Ne nevezd el a változódat `list`-nek vagy `id`-nek, ha később a built-in függvényt is hívnád.

## Interview questions

**Why are mutable default arguments dangerous?**

Válaszvázlat: egyszer jön létre a default; több hívás osztja ugyanazt a módosítható objektumot. Új objektumot hozz létre a törzsben.

**What does a closure capture in Python?**

Válaszvázlat: környező kötéseket; a late binding miatt a futáskori érték számít. Magyarázd el a callback-példát és a javítást.

## Önellenőrzés

- [ ] Default hibát javítok a hívói lista viselkedésének tisztázásával.
- [ ] Megkülönböztetem a `global` és `nonlocal` jelentését.
- [ ] Callback-listát készítek late binding hiba nélkül.


## Kapcsolódó gyakorlat

[Önálló feladat: B02](../../../exercises/senior-python/01-python-basics.md#b02)

## Forrás és továbbolvasás

[Function definitions](https://docs.python.org/3/tutorial/controlflow.html#defining-functions) · [Execution model](https://docs.python.org/3/reference/executionmodel.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
