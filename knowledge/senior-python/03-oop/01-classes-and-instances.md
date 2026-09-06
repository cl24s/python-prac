# Osztályok, példányok és attribútumok

## Mit kell tudnod?

- Elkülöníteni a típushoz tartozó közös adatot a példány saját állapotától.
- Megmagyarázni a metóduskötést és a `self` szerepét.
- Eldönteni, hogy indokolt-e egyáltalán osztályt létrehozni.

## Magyarázat

Egy osztály saját típust ír le: az objektum állapotát és az azon értelmes műveleteket kapcsolhatja össze. Az osztály maga is objektum. A példányosítás szokásos útján a `__new__` létrehozza a példányt, az `__init__` inicializálja; az utóbbinak nem szabad tetszőleges értéket visszaadnia. Alkalmazáskódban általában csak az `__init__` szükséges.

Az osztálytörzsben megadott adat osztályattribútum. A `self.name = ...` rendszerint példányattribútumot hoz létre. Közönséges attribútumoknál a példányon létrehozott név elfedheti az osztály ugyanilyen nevű attribútumát. Property-k és más descriptorok módosítják a feloldás részleteit; ezekhez a következő fejezet kapcsolódik.

## Kódpélda: megosztott lista és helyes példányállapot

```python
class BadRegistry:
    services: list[str] = []

left = BadRegistry()
right = BadRegistry()
left.services.append('api')
assert right.services == ['api']

class Registry:
    default_region = 'eu'

    def __init__(self) -> None:
        self.services: list[str] = []

    def add(self, name: str) -> None:
        self.services.append(name)

first = Registry()
second = Registry()
first.add('worker')
assert first.services == ['worker']
assert second.services == []
first.default_region = 'us'
assert second.default_region == 'eu'
assert Registry.default_region == 'eu'
assert first.add.__self__ is first
Registry.add(second, 'scheduler')
assert second.services == ['scheduler']
```

A `left.services.append()` nem hoz létre példányattribútumot: megtalálja a közös listát és módosítja. Ezzel szemben a `first.default_region = ...` közönséges attribútumnál példányszintű értéket állít be. A `first.add` kötött metódus: a `self` automatikusan átadódik. A `self` elnevezés konvenció, nem kulcsszó.

## Mikor használnád?

Ha több művelet ugyanazt az állapotot használja és annak szabályait védi, az osztály összetartja a felelősséget. Például egy kapacitáskorlátos buffernek értelmes saját `add` és `drain` művelete. Egy állapotmentes `normalize_name(text)` átalakítót viszont egyszerű függvényként könnyebb olvasni és tesztelni.

## Tipikus hibák és senior szempontok

- **Hiba:** kérésenként különnek gondolt mutable osztályattribútum. **Javítás:** új objektum létrehozása az inicializálóban.
- **Hiba:** minden stateless helper köré `Manager` osztály. **Javítás:** nevezz meg valódi állapotot vagy életciklust; enélkül modul/funkció gyakran elég.
- A konstruktor lehetőleg állapotot állítson be és validáljon; hálózati hívás vagy workerindítás külön életciklus-művelet legyen.
- A `__slots__` csökkentheti a példányok tárigényét és korlátozza az új attribútumokat, de nem biztonsági védelem és nem deep immutability. Mérés nélkül ne ez legyen az első optimalizáció.

## Interview questions

**Why can two instances unexpectedly share a list?**

Válaszvázlat: a lista az osztályon jött létre; a metódus ugyanazt a mutable objektumot éri el. A per-instance lista az `__init__`-ben készüljön.

**When is a function preferable to a class?**

Válaszvázlat: állapotmentes átalakításnál, ahol nincs közös invariáns vagy életciklus. Az osztály nem önmagában minőségi mutató.

## Önellenőrzés

- [ ] Megmagyarázom a példányattribútum elfedését és a közös objektum mutációját.
- [ ] Tudom, honnan kapja a kötött metódus a `self`-et.
- [ ] Egy osztály létrehozását felelősséggel indoklom.


## Kapcsolódó gyakorlat

[O01 – önálló feladat](../../../exercises/senior-python/03-oop.md#o01)

## Forrás és továbbolvasás

[Python classes](https://docs.python.org/3/tutorial/classes.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
