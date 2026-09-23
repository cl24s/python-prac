# Tesztelés és hibakeresés az első naptól

## Mit kell tudnod?

Különbséget tenni a várt és a tényleges működés között, megtalálni a legkisebb hibás példát, majd teszttel megőrizni a javítást. Ez nem csak a későbbi nagy projektek feladata.

## Saját elvárás, nem a kód visszamondása

A teszt elvárt eredményét a feladat szerződéséből határozd meg, ne ugyanazzal az algoritmussal számold ki, amelyet tesztelsz. Kezdetben legyen normál eset, üres bemenet és releváns határérték; csak a szerződés által megengedett inputokat és előírt hibákat kérd számon.

```python
def clamp(value: int, low: int, high: int) -> int:
    if low > high:
        raise ValueError("invalid interval")
    return min(max(value, low), high)

assert clamp(5, 0, 10) == 5
assert clamp(-1, 0, 10) == 0
assert clamp(20, 0, 10) == 10
```

Az `assert` itt oktatási ellenőrzés. Ne erre építs éles bemenetvalidációt, mert optimalizált Python-futtatáskor kikapcsolható.

## Áttérés pytestre

A következő önálló blokk ideiglenes `test_example.py` fájlban futtatható. A saját gyakorlatban az implementáció és a teszt később külön fájl legyen.

```python
import pytest


def clamp(value: int, low: int, high: int) -> int:
    if low > high:
        raise ValueError("invalid interval")
    return min(max(value, low), high)


@pytest.mark.parametrize("value, expected", [(-1, 0), (0, 0), (5, 5), (10, 10), (11, 10)])
def test_clamp(value: int, expected: int) -> None:
    assert clamp(value, 0, 10) == expected


def test_invalid_interval() -> None:
    with pytest.raises(ValueError, match="invalid interval"):
        clamp(2, 10, 0)
```

Futtatás a választott környezet interpreterével: Windows alatt `.\.venv\Scripts\python.exe -m pytest test_example.py -q`, Linux/WSL alatt `.venv/bin/python -m pytest test_example.py -q`. A parancsot az adott fájlt tartalmazó könyvtárból vagy megfelelő relatív útvonallal add ki.

## Hibakeresési sorrend

1. Olvasd el a traceback végén a kivételtípust és üzenetet, majd keresd meg a saját kódod érintett sorát.
2. Szűkítsd a bemenetet a legkisebb hibát előidéző esetre. Írd le, mit vártál és mi történt.
3. Vizsgáld meg a változók értékét, típusát és a vezérlési utat. Ehhez ideiglenes kiírás vagy `breakpoint()` használható.
4. Egy feltételezést vizsgálj egyszerre. A javítást regressziós teszt és a korábbi tesztek újrafuttatása kövesse.

Debuggerben a `n` a következő sor, az `s` beléphet a hívott függvénybe, a `p expression` kiír egy értéket, a `c` folytat, a `q` kilép. Ne hagyj aktív breakpointot a tesztelt, végleges megoldásban.

## Review és önellenőrzés

Az „átírtam és most jó” helyett magyarázd el az eredeti okot. Például a ciklusba került return túl korán visszatér, a bejárás közbeni törlés pedig elemet hagyhat ki. A kód ne nyeljen el minden kivételt csak azért, hogy zöld legyen a teszt.

A témát később [Q02](../../../exercises/senior-python/07-testing-and-quality.md#q02) és [Q05](../../../exercises/senior-python/07-testing-and-quality.md#q05) mélyíti el. Nem kell rögtön mock-framework, coverage-cél vagy mutation testing.

**Explain:** Which test would fail before your fix, and why?

Kapcsolódó feladat: [L05](../../../exercises/senior-python/00-foundations.md#l05).

[Pytest kezdés](https://docs.pytest.org/en/stable/getting-started.html) · [pdb](https://docs.python.org/3/library/pdb.html) · [Vissza](README.md)
