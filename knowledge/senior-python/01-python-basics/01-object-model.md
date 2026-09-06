# Objektumok, referenciák és mutability

## Mit kell tudnod?

- Megkülönböztetni az objektumot, a rá mutató nevet és az objektum értékét.
- Előre megmondani, hogy egy módosítás látható lesz-e a hívónál.
- Indokolni az `is`, `==` és `is None` használatát.

## Magyarázat

Az `a = value` egy nevet köt egy objektumhoz. A `b = a` nem másolja le az adatot: két név ugyanazt az objektumot éri el. A **mutáció** a meglévő objektum állapotát módosítja; az **újrakötés** egy nevet másik objektumhoz rendel. Ez a különbség fontosabb, mint az a pontatlan mondat, hogy „a Python referenciaként ad át mindent”.

Függvényhíváskor a paraméter helyi névként a kapott objektumhoz kötődik. A hívott függvény mutálhat egy közös listát, de a paraméter újrakötése nem köti újra a hívó változóját. Egy ilyen mellékhatás lehet szándékos, de legyen része a függvény szerződésének.

## Kódpélda: mutáció és újrakötés

```python
original = ['api']
alias = original

def update(services):
    services.append('worker')
    services = ['scheduler']
    return services

replacement = update(alias)
assert original == ['api', 'worker']
assert alias is original
assert replacement == ['scheduler']
assert replacement is not original
```

Az `append` a közös listát módosítja. A következő sor csak a helyi `services` nevet köti új listához. A visszatérési érték átvehető, de nem cseréli le automatikusan az `original`-t.

## Értékegyenlőség és azonosság

Az `==` az adott típus egyenlőségi szabályát használja; az `is` ugyanazt az objektumot keresi. Listák esetén két külön objektum lehet érték szerint egyenlő. A `None` egyedi sentinel, ezért `is None` a helyes szokás. Számok vagy stringek összehasonlítását ne építsd objektum-újrafelhasználásra.

```python
left = ['api']
right = ['api']
assert left == right
assert left is not right

config = (['api'], 3)
config[0].append('worker')
assert config == (['api', 'worker'], 3)

value = 0
assert not value
assert value is not None
```

A tuple saját elemekhez tartozó referenciái nem cserélhetők ki; a benne lévő lista ettől még mutable. Az immutable konténer nem jelent automatikusan mély immutabilitást.

## Mikor használnád?

Egy HTTP-kéréshez kapott konfiguráció vagy payload normalizálásakor először döntsd el, módosíthatod-e a bemenetet. Ha ugyanazt az objektumot naplózás, cache vagy más feldolgozó is használja, a mellékhatás máshol jelentkezhet.

## Tipikus hibák és senior szempontok

- **Hiba:** `if not timeout` alapján hiányzó értéket feltételezel. A nulla és a `None` eltérő jelentést kaphat. **Javítás:** a hiányt külön vizsgáld.
- **Hiba:** a `+=` műveletet mindig újrakötésnek tekinted. Listánál helyben bővíthet, immutable értéknél új eredményt köt a névhez.
- Dokumentáld az adat tulajdonosát: ki módosíthatja, kinek kell másolat?
- A `del name` névkötést töröl; nem jelent minden hivatkozástól független objektummegsemmisítést.

## Interview questions

**How does Python pass arguments to functions?**

Válaszvázlat: a paraméter a kapott objektumhoz kötődik; mutáció látszódhat a hívónál, helyi újrakötés nem. Mutass külön példát a kettőre.

**Can an immutable object contain mutable state?**

Válaszvázlat: tuple tartalmazhat listát; a konténer közvetlen referenciái rögzítettek, a lista tartalma módosulhat.

## Önellenőrzés

- [ ] Futás nélkül megmagyarázom mindkét példa állításait.
- [ ] Nem használok `is`-t stringek értékének összehasonlítására.
- [ ] Egy API-függvénynél megmondom, módosítja-e a bemenetet.


## Kapcsolódó gyakorlat

[Önálló feladat: B01](../../../exercises/senior-python/01-python-basics.md#b01)

## Forrás és továbbolvasás

[Python data model](https://docs.python.org/3/reference/datamodel.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
