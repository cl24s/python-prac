# Virtuális környezet, csomagolás és reprodukálható telepítés

## Mit kell tudnod?

- Elkülöníteni az importálható csomagot, a disztribúciót és a futtatókörnyezetet.
- Megkülönböztetni a kompatibilitási tartományt a rögzített alkalmazáskörnyezettől.
- Az elkészült artifactot is ellenőrizni a forrásfa helyett.

## Magyarázat

A virtuális környezet külön Python-csomagteret ad; nem konténer és nem biztonsági sandbox. A `python -m pip` ugyanahhoz az interpreterhez kapcsolja a telepítést, amellyel később futtatsz. A környezetet újraépítjük a deklarációból, nem másoljuk Windowsról Linuxra.

A `pyproject.toml` három eltérő kérdésnek ad helyet: mivel épül a csomag (`build-system`), mit ígér a disztribúció (`project`), hogyan működnek a fejlesztői eszközök (`tool`). A telepíthető projekt neve és az import neve eltérhet, például `prac-status` és `prac_status`. A runtime dependency a felhasználónál is kell; a tesztfuttató és linter fejlesztői függőség.

## Konkrét csomagterv

Egy későbbi CLI-nél a `src/prac_status/cli.py` exportálja a `main` függvényt. A README és a pyproject a projekt gyökerében van. Az alábbi önálló teszt egy minimális metaadatmintát olvas; nem hoz létre csomagot.

```python
import tomllib

METADATA = """
[build-system]
requires = ["setuptools>=77.0.3"]
build-backend = "setuptools.build_meta"

[project]
name = "prac-status"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = []

[project.scripts]
prac-status = "prac_status.cli:main"

[tool.setuptools.packages.find]
where = ["src"]
"""

def test_packaging_metadata():
    data = tomllib.loads(METADATA)
    assert data['project']['scripts']['prac-status'] == 'prac_status.cli:main'
    assert data['tool']['setuptools']['packages']['find']['where'] == ['src']
    assert data['build-system']['build-backend'] == 'setuptools.build_meta'
```

A parse sikeressége nem bizonyítja, hogy a hivatkozott modul létezik, vagy a wheel helyes. A build ellenőrzése külön lépés: wheel és sdist építése, a wheel telepítése tiszta környezetbe, majd import és CLI futtatása a checkout könyvtárán kívül. Az editable install elrejtheti a hiányzó csomagadatokat. Az sdistből építhetőséget külön is vizsgáld.

## Senior döntések és tipikus hibák

Egy könyvtár függőségtartománya a támogatott együttműködést írja le. Egy alkalmazás telepítésénél a tranzitív verziókat és artifactokat is rögzíteni kell. A `pip freeze` pillanatfelvétel, nem automatikusan tiszta, többplatformos lock. Egy lock értelmezése eszközfüggő; a Python-verziót, platformot és feloldóeszközt is rögzítsd.

A pip hash-checking módjában minden szükséges függőségnek pontos verzió és elfogadott hash kell. A hash a kiválasztott bájtok azonosságát igazolja, nem a csomag ártalmatlanságát. A build isolation saját buildfüggőségeket telepíthet: a runtime lock önmagában nem rögzíti ezeket. A reprodukálható környezet és a bitazonos build két külön cél.

Egy frissítési folyamat nélkül a tökéletesen rögzített környezet is elavul. Frissítéskor tesztelj, építs új artifactot és rögzítsd annak azonosítóját; ne az éles konténerbe telepíts utólag.

## Interview questions

**Why can editable installs hide packaging bugs?**

Válaszvázlat: A forrásfa elérhető marad, így egy hiányzó wheel-modul vagy adatfájl nem feltétlenül bukik ki. Tiszta artifact-telepítés kell.

**Does a lock file make a build reproducible?**

Válaszvázlat: A feloldás egy részét rögzíti. Python, platform, buildeszköz, buildfüggőségek és artifactforrások is számítanak.

## Önellenőrzés

- [ ] El tudom magyarázni a build és runtime függőségeket.
- [ ] Meg tudom nevezni, mit nem bizonyít a TOML-teszt.

## Kapcsolódó gyakorlat

[OP01 – önálló feladat](../../../exercises/senior-python/13-packaging-security-and-operations.md#op01)

## Forrás és továbbolvasás

[PyPA: pyproject.toml](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/)

[pip: secure installs](https://pip.pypa.io/en/stable/topics/secure-installs/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
