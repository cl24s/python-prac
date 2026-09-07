# Csomagolás, biztonság és üzemeltetés

Állapot: kidolgozott első változat, 7 fejezet és 7 önálló feladat. Az elsajátítást külön a [ROADMAP](../../../ROADMAP.md) követi.

## Tanulási sorrend

1. [Virtuális környezet, csomagolás és reprodukálható telepítés](01-packaging-and-dependencies.md)
2. [Konfiguráció, secret-kezelés és indulási validáció](02-configuration-and-secrets.md)
3. [Strukturált logok, metrikák, tracing és health checkek](03-observability-and-health.md)
4. [CI, ellenőrzési kapuk és artifactok továbbadása](04-ci-and-artifact-promotion.md)
5. [Konténeres futtatás, workerek és erőforráskorlátok](05-containers-and-capacity.md)
6. [SQL injection, command injection és deszerializáció](06-injection-and-deserialization.md)
7. [SSRF, jogosultsági határok és érzékeny adatok](07-ssrf-and-data-exposure.md)

## Hogyan dolgozz vele?

1. Olvasd el a magyarázatot, és mondd el a döntés vagy hibamód lényegét saját szavaiddal.
2. Ahol van Python-példa, másold a teljes blokkot külön `test_example.py` fájlba és futtasd pytesttel. A sima `python test_example.py` nem indítja a tesztfüggvényeket.
3. Az esettanulmányoknál nevezd meg a feltételezéseket, és mondj egy változást, amely más döntést indokolna.
4. Válaszolj angolul az interjúkérdésekre; a magyar válaszvázlat ellenőrzésre való.
5. Oldd meg a [kapcsolódó feladatsort](../../../exercises/senior-python/13-packaging-security-and-operations.md). Kész feladatmegoldás nincs mellékelve.

## Futtatókörnyezet és határok

A Python-blokkok CPython 3.12.13, pytest 9.1.1 és Linux alatt ellenőrzöttek. Runtime logikájuk standard libraryt használ. A kód nélküli fejezetek tervezési eseteket és döntési táblákat tartalmaznak.

```sh
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
# Linux / WSL: source .venv/bin/activate
python -m pip install pytest==9.1.1
python -m pytest -q test_example.py
```

A hat Python-blokk 17 tesztesete TOML-metaadatot, konfigurációt, JSON-eseményt, connection-budgetet, SQLite-paraméterezést, veszélytelen subprocess-argumentumot és publikus hibaválaszt ellenőriz. A TOML parse nem wheel-build, a kapacitásmodell nem load test. Nincs futtatott Docker/Kubernetes/CI, secret manager, telemetry collector vagy valódi SSRF hálózati vizsgálat. A Windowsos subprocess-viselkedés nem volt tesztelve.

A [FastAPI](../09-fastapi/README.md), [megbízhatóság](../11-reliable-services/README.md) és [teljesítmény](../12-performance-and-debugging/README.md) anyagára építünk. A [logelemző CLI](../../../projects/01-log-analyzer/README.md) később csomagolási, az [endpoint checker](../../../projects/02-endpoint-checker/README.md) hálózatbiztonsági gyakorlótér lehet.

[Az ellenőrzések részletei](../VALIDATION.md) · [Teljes tudástérkép](../README.md)
