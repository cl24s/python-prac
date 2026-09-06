# Callable, ParamSpec, Self és típusos API-tervezés

## Mit kell tudnod?

- Megőrizni egy wrapper bemeneti és kimeneti típuskapcsolatát.
- A visszatérési típust a valódi publikus szerződéshez igazítani.
- A szükséges egyszerűséget választani típusbravúr helyett.

## Magyarázat

A `Callable[[str], int]` egy stringet fogadó és intet adó hívható objektum. A `Callable[..., Any]` ezzel szemben elveszít fontos információt. Egy általános decoratornál a paraméterek számát, neveit és típusait nem feltétlenül ismerjük előre; a `ParamSpec` ezt a paraméterlistát őrzi, a TypeVar a visszatérési típust kapcsolja össze.

## Kódpélda: típusmegőrző wrapper

```python
from collections.abc import Callable
from functools import wraps
from typing import ParamSpec, TypeVar, assert_type

P = ParamSpec('P')
R = TypeVar('R')

def forward(function: Callable[P, R]) -> Callable[P, R]:
    @wraps(function)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        return function(*args, **kwargs)
    return wrapper

@forward
def format_count(name: str, *, count: int) -> str:
    return f'{name}:{count}'

result = format_count('api', count=3)
assert_type(result, str)
assert result == 'api:3'
```

A wrapper nem teszi tetszőlegessé a paramétereket a checker számára. A keyword-only `count` is megmarad. A `wraps` a runtime metaadatokat segíti, a ParamSpec a statikus kapcsolatot: eltérő feladatuk van. A wrapper itt sync; a coroutine végrehajtásának mérése vagy hibakezelése külön async-tervezést igényelne.

## Kódpélda: a Self a tényleges típust követi

```python
from typing import Self, assert_type

class Label:
    def __init__(self, text: str) -> None:
        self.text = text

    @classmethod
    def parse(cls, text: str) -> Self:
        return cls(text.strip())

class ServiceLabel(Label):
    pass

label = ServiceLabel.parse(' api ')
assert_type(label, ServiceLabel)
assert isinstance(label, ServiceLabel)
assert label.text == 'api'
```

A Self azt ígéri, hogy az eredmény a megfelelő aktuális típus. Ha a metódus mindig `Label(...)`-t adna, az alosztály felől tett ígéret hibás lenne. Alosztályban megváltoztatott, inkompatibilis konstruktor ugyanígy megtörheti a factory működését.

## API-tervezési döntések

- Bemenetnél a szükséges képességet kérd (`Iterable`, kis Protocol); eredménynél a ténylegesen garantált típust add meg.
- A `Literal` néhány megengedett értéket tud leírni. Az `overload` külön bemeneti alakokhoz külön eredményt rendelhet, de runtime csak egy implementáció fut; nem metódusdispatch.
- Ha a sok union és overload nehezen érthető, lehet, hogy két külön elnevezett művelet tisztább API lenne.
- A `Final` és `ClassVar` az eszköznek is jelzi a szándékot, de nem általános runtime írásvédelem.
- Ne terheld a domainlogikát csak azért metaclassokkal vagy dinamikus attribútumokkal, mert Pythonban megtehető; a típusellenőrzés és refaktorálás költségét is mérlegeld.

## Tipikus hibák és senior szempontok

**Hiba:** a wrapper minden típusinformációt `Any`-ra cserél, és a hibás hívók átcsúsznak. **Javítás:** ParamSpec és visszatérési TypeVar, vagy egyszerű konkrét szignatúra.

**Hiba:** a zöld checker miatt nincs viselkedési teszt. **Javítás:** a szerződés sorrend-, mellékhatás-, hiba- és időbeli részét külön vizsgáld. A legjobb publikus típus az, amely valódi ígéretet fejez ki és a csapat számára olvasható.

## Interview questions

**What information does ParamSpec preserve in a decorator?**

Válaszvázlat: a teljes paraméterszerződés kapcsolatát; a visszatéréshez külön R típust használunk. A wraps ettől eltérő runtime metaadatot őriz.

**When is Self more accurate than naming the base class?**

Válaszvázlat: fluent API vagy alternatív konstruktor, amely ténylegesen az aktuális alosztály típusát őrzi.

## Önellenőrzés

- [ ] A wrapper típusa nem lazítja fel indokolatlanul a hívást.
- [ ] A Self-ígéretet runtime is betartom.
- [ ] Tudok egyszerűbb API-t javasolni túl bonyolult típusleírás helyett.


## Kapcsolódó gyakorlat

[T06 – önálló feladat](../../../exercises/senior-python/04-types-and-interfaces.md#t06)

## Forrás és továbbolvasás

[ParamSpec and Self](https://docs.python.org/3/library/typing.html) · [Generics specification](https://typing.python.org/en/latest/spec/generics.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
