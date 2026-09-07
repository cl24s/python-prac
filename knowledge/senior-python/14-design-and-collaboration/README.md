# Senior szintű tervezés és együttműködés

Állapot: kidolgozott első változat, 7 fejezet és 7 önálló feladat. Az elsajátítást külön a [ROADMAP](../../../ROADMAP.md) követi.

## Tanulási sorrend

1. [Követelmények, invariánsok és hibaforgatókönyvek](01-requirements-and-failure-scenarios.md)
2. [Modulhatárok, cohesion és függőségi irányok](02-module-boundaries-and-dependencies.md)
3. [Egyszerű megoldás, moduláris monolit és bővíthetőség](03-simplicity-and-evolution.md)
4. [Technikai döntések, ADR és kompromisszumok](04-decisions-and-communication.md)
5. [Biztonságos továbbfejlesztés és fokozatos migráció](05-safe-change-and-migration.md)
6. [Kódmegértés, review és mentorálás](06-review-and-mentoring.md)
7. [Saját éles példák és szakmai tanulságok bemutatása](07-interview-stories-and-learning.md)

## Hogyan dolgozz vele?

1. Olvasd el a magyarázatot, és mondd el a döntés vagy hibamód lényegét saját szavaiddal.
2. Ahol van Python-példa, másold a teljes blokkot külön `test_example.py` fájlba és futtasd pytesttel. A sima `python test_example.py` nem indítja a tesztfüggvényeket.
3. Az esettanulmányoknál nevezd meg a feltételezéseket, és mondj egy változást, amely más döntést indokolna.
4. Válaszolj angolul az interjúkérdésekre; a magyar válaszvázlat ellenőrzésre való.
5. Oldd meg a [kapcsolódó feladatsort](../../../exercises/senior-python/14-design-and-collaboration.md). Kész feladatmegoldás nincs mellékelve.

## Futtatókörnyezet és határok

A Python-blokkok CPython 3.12.13, pytest 9.1.1 és Linux alatt ellenőrzöttek. Runtime logikájuk standard libraryt használ. A kód nélküli fejezetek tervezési eseteket és döntési táblákat tartalmaznak.

```sh
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
# Linux / WSL: source .venv/bin/activate
python -m pip install pytest==9.1.1
python -m pytest -q test_example.py
```

Két Python-blokk 10 tesztesete egy HTTP-től független olvasási use case-t és két üzenetverzió olvasási kompatibilitását vizsgálja. Nincs valódi DB-migráció, többverziós deployment vagy konkurenciateszt. A saját esettanulmányok nem személyes munkatörténetek és nem a projekt végleges specifikációi.

A [job processing API](../../../projects/03-job-processing-api/README.md) később közös terep a modulhatárokhoz, döntésekhez és migrációhoz. Az [OOP](../03-oop/README.md) és [típusok](../04-types-and-interfaces/README.md) a kódszintű alapokat, ez a témakör a rendszer- és csapatszintű döntéseket tárgyalja.

[Az ellenőrzések részletei](../VALIDATION.md) · [Teljes tudástérkép](../README.md)
