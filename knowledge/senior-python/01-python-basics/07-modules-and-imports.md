# Modulok, package-ek és importok

## Mit kell tudnod?

- Érteni, hogy az import kódot is végrehajthat.
- Megkülönböztetni a modulobjektumot és az importált névkötést.
- Felismerni a körkörös importot és a rosszul megválasztott modulhatárokat.

## Magyarázat

Egy `.py` fájl tipikusan modul. Egy hagyományos package könyvtár `__init__.py` fájllal; namespace package-eknél ez nem kötelező. Az importáló rendszer megkeresi és inicializálja a modult, majd a modulobjektumot a `sys.modules` tárolja. Egy interpreterben a szokásos újabb import általában ebből használja a már betöltött modult.

A modul felső szintű utasításai inicializáláskor futnak. Ezért veszélyes ott hálózati kapcsolatot nyitni, workert indítani vagy nagy fájlt feldolgozni: egy ártatlan import is lassú vagy hibás lehet. Az `if __name__ == '__main__':` blokk a belépési pont futását különíti el az importálástól.

## Kódpélda: valódi modulok ideiglenes könyvtárban

A példa maga hozza létre az apró package-et, ezért nem kell mellé külön forrásfájlokat kézzel készíteni. A subprocess-ekben az `assert` hibák is nem nulla kilépési kódot okoznak.

```python
from pathlib import Path
from tempfile import TemporaryDirectory
import subprocess
import sys

with TemporaryDirectory() as directory:
    root = Path(directory)
    package = root / 'lesson_demo'
    package.mkdir()
    (package / '__init__.py').write_text('', encoding='utf-8')
    (package / 'settings.py').write_text('RETRIES = 2\n', encoding='utf-8')
    (package / 'cli.py').write_text(
        "from .settings import RETRIES\n"
        "def main():\n    print(RETRIES)\n"
        "if __name__ == '__main__':\n    main()\n",
        encoding='utf-8',
    )
    result = subprocess.run(
        [sys.executable, '-m', 'lesson_demo.cli'],
        cwd=root, capture_output=True, text=True, check=True,
    )
    assert result.stdout.strip() == '2'
    probe = subprocess.run(
        [sys.executable, '-c',
         'import lesson_demo.settings as settings; '
         'from lesson_demo.settings import RETRIES; '
         'settings.RETRIES = 5; '
         'assert RETRIES == 2; assert settings.RETRIES == 5'],
        cwd=root, capture_output=True, text=True, check=True,
    )
    assert probe.stdout == ''
```

A `from ... import RETRIES` a pillanatnyi objektumhoz köt helyi nevet. A modul attribútumának későbbi újrakötése nem írja át ezt a kötést. Ha mutable objektumot importálnál és azt helyben módosítanád, a közös referencia miatt más lenne az eredmény.

## Körkörös import és javítása

Ha `service` importálja `storage`-ot, az pedig inicializálás közben visszaimportálna egy még létre nem jött nevet a `service`-ből, részben inicializált modult érhet el. Nem minden ciklus hibázik azonnal, de a betöltési sorrendtől függő működés törékeny.

Elsőként a függőségi irányt javítsd: közös adatmodell kerüljön harmadik, alacsonyabb szintű modulba; az alkalmazás összeszerelése legyen külön; konkrét szolgáltatásokat paraméterként adhatsz át. A függvényen belüli import néha indokolt késleltetés, de önmagában nem oldja meg a rossz felelősséghatárokat.

## Tipikus hibák és senior szempontok

- Saját `json.py` vagy `typing.py` fájl elfedhet standard library modult. Átnevezéssel és tiszta indítási környezettel javítsd.
- Package-modult általában a package gyökeréből `python -m package.module` módon indíts, ha relatív importokra támaszkodik.
- Ne `sys.path` kézi módosítgatásával rejtsd el a hibás csomagolást.
- Az importált globális cache egy interpreterben közös; több worker processz között nem közös tár.

## Interview questions

**Why can a circular import fail only in one import order?**

Válaszvázlat: inicializálás közben egy szükséges név még nem létezik; a ciklus időzítése számít. Függőségi határral javítsd.

**Does from module import name track later reassignment?**

Válaszvázlat: nem; saját névkötést hoz létre. Különítsd el ezt a közös mutable objektum mutálásától.

## Önellenőrzés

- [ ] Az importálás nem indít véletlenül üzleti munkát a modulomban.
- [ ] Megmagyarázom a két subprocess eredményét.
- [ ] Tudok architekturális javítást javasolni egy importciklusra.


## Kapcsolódó gyakorlat

[Önálló feladat: B06](../../../exercises/senior-python/01-python-basics.md#b06)

## Forrás és továbbolvasás

[Modules tutorial](https://docs.python.org/3/tutorial/modules.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
