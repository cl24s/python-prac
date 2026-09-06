# Generikus típusok, variancia és kollekciós interfészek

## Mit kell tudnod?

- Kapcsolatot kifejezni a bemenet és a visszatérési típus között.
- Megérteni, miért nem helyettesíthető minden mutable konténer egy tágabb típussal.
- A fogyasztó igényéhez választani `Iterable`, `Sequence` vagy `Mapping` interfészt.

## Magyarázat

A generikus függvény vagy típus paraméterezett típussal írja le a kapcsolatokat. A `first` visszatérési értéke például az iterable elemének típusa. Ha mindent `object`-re cserélsz, a kapcsolat elveszik; ha `Any`-ra, az ellenőrzés gyengül.

A TypeVar nem tetszőleges runtime változó, hanem az adott használat során konzisztens típuskapcsolatot kifejező paraméter. A példák a régebbi, széles körben olvasható `TypeVar` szintaxist használják; Python 3.12-től a `def first[T](...)` forma is elérhető.

## Kódpélda: típusmegőrző művelet

```python
from collections.abc import Iterable
from typing import TypeVar, assert_type

T = TypeVar('T')

def first(items: Iterable[T]) -> T:
    for item in items:
        return item
    raise ValueError('empty input')

name = first(['api', 'worker'])
count = first([2, 3])
assert_type(name, str)
assert_type(count, int)
assert name.upper() == 'API'
assert count + 1 == 3
```

Az `assert_type` a checker számára ellenőrzi az elvárt típust; runtime nem üzleti validáció. Az üres bemenet szerződését külön határoztuk meg: itt exception, nem `T | None`.

## Miért invariáns a list?

Ha a `list[Dog]` átadható lenne `list[Animal]` helyett egy módosító függvénynek, az utóbbi Cat-et tehetne bele. A hívó ezután Dog-listának gondolná a Cat-et tartalmazó listát. Ezért a mutable list invariáns.

A következő negatív példa futtatva pontosan a veszélyt mutatja; a checker `arg-type` hibával elutasítja.

```python
# typecheck: expect-error [arg-type]
class Animal:
    pass

class Dog(Animal):
    pass

class Cat(Animal):
    pass

def add_cat(animals: list[Animal]) -> None:
    animals.append(Cat())

dogs: list[Dog] = [Dog()]
add_cat(dogs)
assert isinstance(dogs[-1], Cat)
```

Csak olvasó `Sequence[Animal]` paraméterhez egy Dog-lista megfelelő lehet. Ez nem fagyasztja meg az eredeti listát: az adott függvénynek ad szűkebb, olvasó típusfelületet. A mögöttes objektum más referencián keresztül továbbra is módosulhat.

## Interfészválasztás

| Szükséges művelet | Célszerű bemeneti típus |
| --- | --- |
| Egy bejárás | `Iterable[T]` |
| Indexelés és hossz | `Sequence[T]` |
| Kulcs szerinti olvasás | `Mapping[K, V]` |
| Szándékos listamódosítás | `list[T]` vagy megfelelő mutable interfész |

A kovariancia azt engedi, hogy egy kimenetként használt szűkebb típus helyettesítsen tágabbat. Kontravariancia fogyasztóknál jelenhet meg: egy minden Animalt kezelő callable Dogot is képes kezelni. Saját generikus típusnál az olvasási és írási szerep alapján dönts, ne automatikusan állíts kovarianciát.

## Tipikus hibák és senior szempontok

- **Hiba:** a generikus függvény valójában csak stringen működik. **Javítás:** konkrét típus, bound vagy viselkedést leíró Protocol.
- **Hiba:** a `Sequence` immutable snapshotnak számít. **Javítás:** az interfész és az ownership két külön szerződés.
- A bound megengedett felső típust jelöl; constraint meghatározott alternatívák közti kapcsolatot. Csak akkor használd, ha a valódi művelet igényli.

## Interview questions

**Why is list invariant while Sequence can be covariant?**

Válaszvázlat: a listába írhat a fogyasztó; a szélesebb elem írása megtörné a szűkebb lista ígéretét. Olvasó interfésznél ez a művelet nincs jelen.

**What does a TypeVar preserve that object does not?**

Válaszvázlat: a paraméterek és eredmény közti konkrét típuskapcsolatot.

## Önellenőrzés

- [ ] A Dog/Cat példával megindoklom az invarianciát.
- [ ] Csak a szükséges kollekciós műveleteket kérem a hívótól.
- [ ] Megkülönböztetem az olvasó interfészt a másolattól.


## Kapcsolódó gyakorlat

[T04 – önálló feladat](../../../exercises/senior-python/04-types-and-interfaces.md#t04)

## Forrás és továbbolvasás

[Generics specification](https://typing.python.org/en/latest/spec/generics.html) · [collections.abc](https://docs.python.org/3/library/collections.abc.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
