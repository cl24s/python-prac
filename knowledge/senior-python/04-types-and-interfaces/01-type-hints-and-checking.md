# Type hint-ek és statikus ellenőrzés

## Mit kell tudnod?

- Megkülönböztetni a futtatást és a statikus típuselemzést.
- Olvasni a függvények és konténerek típusjelöléseit.
- Fokozatosan bevezetni a típusosságot egy meglévő kódbázisban.

## Magyarázat

Az annotation a fejlesztőnek és eszközöknek szóló szerződés. A Python alapból nem ellenőrzi a függvényhívás argumentumait és visszatérési értékét ezek alapján. A statikus ellenőrző a kód és a típusleírások összevetésével találhat hibát futtatás nélkül; a futó program tesztje mást bizonyít.

A `list[str]` stringek listája, a `dict[str, int]` stringkulcsokhoz int értéket rendel. A `tuple[str, int]` két különböző pozíciójú mező; a `tuple[str, ...]` tetszőleges hosszú stringtuple. A `-> None` azt jelzi, hogy nincs érdemi visszatérési érték. A típusellenőrző gyakran következtet a helyi típusokra; nem kell minden sort redundánsan annotálni.

## Kódpélda: ellenőrizhető szerződés

```python
from collections.abc import Iterable

def total_errors(counts: Iterable[int]) -> int:
    return sum(counts)

def describe(service: str, errors: int = 0) -> tuple[str, int]:
    return service.upper(), errors

assert total_errors([2, 3]) == 5
assert describe('api') == ('API', 0)
assert total_errors(iter([1, 4])) == 5
```

A függvény csak bejárást igényel, ezért nem kötelezi a hívót listára. A szignatúra már tervezési döntés: több adatforrást támogat, miközben a visszatérési típus egyértelmű.

## Szándékos statikus hiba

A következő blokk futás közben nem dob típushibát, de a típusellenőrzőnek el kell utasítania az értékadást. Ez külön jelölt negatív példa, nem követendő megoldás.

```python
# typecheck: expect-error [assignment]
service: str = 42
assert isinstance(service, int)
```

Elvárt mypy-hibakategória: `assignment`. A pontos üzenetszöveg verziónként eltérhet. A tananyag ellenőrzése a hibakategóriát figyeli; nem a teljes szöveget másolja be elvárásként.

## Használat és bevezetési sorrend

Egy kimásolt pozitív példán `python -m mypy --strict example.py` futtatható a saját környezetbe telepített mypy-val. A futtatás ettől külön `python example.py`. A strict kapcsoló az ellenőrző szabálycsomagja, nem a Python interpreter módja.

Meglévő rendszerben kezdd a publikus függvényhatárokkal és adatmodellekkel. Adj pontos típust a külső függőségek wrapperjeinek, majd csökkentsd a továbbterjedő `Any`-t. CI-ban először követhető részhalmazt ellenőrizz, de az újonnan típusozott kód ne legyen végig ignore-okkal kikapcsolva.

## Tipikus hibák és senior szempontok

- **Hiba:** a zöld mypy miatt nem validálsz JSON-t. **Javítás:** a külső inputhoz runtime ellenőrzés kell.
- **Hiba:** annotáció nélküli függvénytest is teljesen ellenőrzöttnek számít. **Javítás:** nézd meg az eszköz beállításait; annotáld a határokat és a visszatérést.
- A stubok pontossága befolyásolja az eredményt. Hiányos third-party típusinformáció esetén keskeny adaptert készíts.
- A típusellenőrzés nem bizonyít idempotenciát, jogosultságot vagy helyes üzleti állapotátmenetet.

## Interview questions

**What does a type checker guarantee that runtime tests do not, and vice versa?**

Válaszvázlat: a checker típuskapcsolatokat vizsgál több lehetséges út mentén; a teszt konkrét viselkedést futtat. Egyik sem helyettesíti a másikat.

**How would you introduce typing into a large untyped codebase?**

Válaszvázlat: határok és modellek, külső Any kontrollja, fokozatos szigorítás, ellenőrzés CI-ban.

## Önellenőrzés

- [ ] Megmagyarázom, miért fut le a negatív példa.
- [ ] Olvasom a tuple és konténerek típusszintaxisát.
- [ ] Külön nevezem meg a statikus és futásidejű ellenőrzést.


## Kapcsolódó gyakorlat

[T01 – önálló feladat](../../../exercises/senior-python/04-types-and-interfaces.md#t01)

## Forrás és továbbolvasás

[mypy getting started](https://mypy.readthedocs.io/en/stable/getting_started.html) · [typing](https://docs.python.org/3/library/typing.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
