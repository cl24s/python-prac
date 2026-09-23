# Fájlok, formátumok és kis programok

## Mit kell tudnod?

A tiszta feldolgozó logikát megkülönböztetni a fájl megnyitásától és a felhasználói felülettől. Így ugyanazt a logikát később CLI-ből, API-ból vagy tesztből is meghívhatod.

## Pathlib, UTF-8 és erőforrás-kezelés

A `Path` fájlútvonalat kezel; a `with` a megnyitott fájl lezárásáról gondoskodik normál és kivételes kilépéskor is. Az encodingot szövegfájloknál rögzítsd. A következő minta csak saját ideiglenes könyvtárban ír.

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as directory:
    path = Path(directory) / "greeting.txt"
    path.write_text("árvíz\nhello\n", encoding="utf-8")
    with path.open(encoding="utf-8") as source:
        first = next(source)
        assert first == "árvíz\n"
        assert list(source) == ["hello\n"]
    assert source.closed
```

Kis fájl teljes beolvasása lehet tudatos döntés. Nagy fájlnál a soronkénti bejárás segíthet, de a kimeneti lista és az egyedi kulcsokat tároló dict ettől még nőhet. A streaming önmagában nem jelenti azt, hogy minden állapot konstans méretű.

## JSON: formátum és séma külön kérdés

```python
import json

payload = json.loads('{"enabled": true, "limit": 3}')
assert payload == {"enabled": True, "limit": 3}
assert json.loads(json.dumps(payload)) == payload
```

A sikeres JSON-parszolás nem bizonyítja, hogy a kötelező mezők, típusok és határértékek helyesek. Az egész számot kérő szerződésnél a boolt külön kell kezelni, ha az nem megengedett. Fájlból a `json.load`, szövegből a `json.loads` olvas. Hibás formátum, hibás alkalmazási adat és hiányzó fájl eltérő hibahelyzet: ne kezeld mindet „üres eredményként”.

## CSV: ne kézi vessződarabolással

```python
import csv
from io import StringIO

source = StringIO('name,quantity\n"pear, green",2\napple,3\n', newline="")
rows = list(csv.DictReader(source))
assert rows[0]["name"] == "pear, green"
assert rows[0]["quantity"] == "2"
```

A CSV-olvasó alapból string mezőket ad; a számmá alakítás és a validáció külön döntés. Valódi CSV-fájlt `encoding="utf-8", newline=""` beállítással nyiss. Rögzítsd a fejléc, üres mezők, hibás sorok és duplikált azonosítók szabályát. Dátumoknál később az időzóna, pénznél az ábrázolás és kerekítés is a szerződés része lesz.

## Modul és CLI határa

Egy függvény először adatot kapjon és eredményt adjon; ne kérjen önállóan `input()`-ot, ha ugyanazt tesztből vagy más modulból is használni akarod. A parancssori réteg olvassa az argumentumot, hívja a logikát és formázza a kimenetet. Az import ne indítson feldolgozást.

```python
def main() -> None:
    print("Example CLI started")


if __name__ == "__main__":
    main()
```

A `__name__` őrfeltétel segít, hogy a közvetlen futtatás és az import ne ugyanazokat a mellékhatásokat okozza. A `python -m package.module` módot a [modulok fejezet](../01-python-basics/07-modules-and-imports.md) részletezi. Az első projektben jöhet az `argparse`, stdout/stderr és exit code; az első függvényhez még nem kell teljes package-struktúra.

## Önellenőrzés

Tudok mesterséges fájllal tesztelni; külön kezelem a formátumhibát és az I/O-hibát; el tudom mondani, melyik réteg nyitja és zárja az erőforrást. Tudom, hogy a fájl lezárása és a feldolgozás sikeressége nem ugyanaz az állítás.

**Explain:** Which part of this program can be tested without opening a real file?

Kapcsolódó feladat: [L06–L08](../../../exercises/senior-python/00-foundations.md#l06), később [logelemző CLI](../../../projects/01-log-analyzer/README.md).

[Python I/O](https://docs.python.org/3/tutorial/inputoutput.html) · [JSON](https://docs.python.org/3/library/json.html) · [CSV](https://docs.python.org/3/library/csv.html) · [Vissza](README.md)
