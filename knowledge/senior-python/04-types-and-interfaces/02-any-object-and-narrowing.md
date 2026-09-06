# Any, object, union és típusszűkítés

## Mit kell tudnod?

- Az ismeretlen értéket biztonságosan kezelni `object` segítségével.
- `None`-t és unionokat elágazással szűkíteni.
- Nem validációként használni a `cast()`-ot.

## Magyarázat

Az `Any` megengedi, hogy a checker bizonyos műveleteket ellenőrzés nélkül elfogadjon; ezért könnyen továbbterjedő ellenőrzési lyuk. Az `object` szintén lehet bármilyen Python-objektum, de csak a minden objektumra érvényes műveleteket engedi addig, amíg nem szűkíted a típusát.

A `str | None` két lehetséges típus unionja. Az `Optional[str]` ugyanezt jelenti, nem azt, hogy a paraméter kihagyható: a kihagyhatóságot a default érték határozza meg. Az `is None` és `isinstance` ágakban a checker szűkebb típussal számolhat.

## Kódpélda: valódi bemeneti ellenőrzés

```python
def require_count(value: object) -> int:
    if isinstance(value, bool) or not isinstance(value, int):
        raise TypeError('count must be an integer, not bool')
    if value < 0:
        raise ValueError('count must not be negative')
    return value

def display_name(name: str | None) -> str:
    if name is None:
        return 'anonymous'
    return name.strip()

assert require_count(0) == 0
assert display_name(None) == 'anonymous'
assert display_name(' api ') == 'api'
try:
    require_count(True)
except TypeError:
    pass
else:
    raise AssertionError('Boolean is not a valid count')
```

Az input ellenőrzése futáskor is megtörténik. Az `int` típus önmagában nem írja le a „nem negatív és nem bool” üzleti feltételt. Az üres string és a `None` külön eset marad.

## Kódpélda: a cast nem alakít át

```python
from typing import cast

raw: object = '12'
claimed = cast(int, raw)
assert claimed is raw
try:
    claimed + 1
except TypeError:
    pass
else:
    raise AssertionError('cast must not convert the runtime value')
```

A checker elfogadja a programozó állítását; az érték string marad. Ha konverzió kell, valódi parse-olás szükséges. Ha már más módon bizonyított a típus, a cast dokumentálhatja ezt, de az állítás helyességéért a kód felel.

## Tipikus hibák és senior szempontok

- **Hiba:** `Any` minden JSON-mezőre, majd tetszőleges metódushívás. **Javítás:** határon validálj, és belül pontos típust adj tovább.
- **Hiba:** unionból `cast`-tal tünteted el a None-t. **Javítás:** ellenőrizd vagy kezeld a hiányt.
- Egyszerű, lokális szűkítés olvashatóbb, mint indokolatlanul összetett típuspredikátum.
- A `TypeGuard` saját ellenőrző függvényhez használható, de hibás predikátummal a checkert félrevezetheted. A `TypeIs` Python 3.13-tól elérhető a standard typingban; a 3.12-es példák nem építenek rá.
- Külső input ellenőrzésére ne csak `assert`-et használj: optimalizált futtatáskor eltűnhet.

## Interview questions

**How is object safer than Any for unknown input?**

Válaszvázlat: az object szűkítést kér a specifikus művelet előtt; az Any megkerülhet sok ellenőrzést.

**Does Optional[str] make a parameter optional to pass?**

Válaszvázlat: nem; a típus a None megengedését jelzi, a default érték a kihagyhatóságot.

## Önellenőrzés

- [ ] Valódi runtime ellenőrzéssel szűkítek ismeretlen inputot.
- [ ] Tudom, miért zöld mégis a cast-os példa statikusan.
- [ ] Megkülönböztetem a None, az üres és a hiányzó értéket.


## Kapcsolódó gyakorlat

[T02 – önálló feladat](../../../exercises/senior-python/04-types-and-interfaces.md#t02)

## Forrás és továbbolvasás

[mypy type narrowing](https://mypy.readthedocs.io/en/stable/type_narrowing.html) · [typing.cast](https://docs.python.org/3/library/typing.html#typing.cast)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
