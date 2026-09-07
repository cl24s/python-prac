# Tesztelés és kódminőség

Állapot: kidolgozott első változat. Az elsajátítást külön a [ROADMAP](../../../ROADMAP.md) követi.

## Tanulási sorrend

1. [Tesztstratégia és a megfelelő teszthatárok](01-test-boundaries.md)
2. [pytest: fixture-ök, parametrizálás és exceptionök](02-pytest-fixtures-and-parameters.md)
3. [Fake, mock és szerződéstesztek](03-test-doubles-and-contracts.md)
4. [Async kód, idő és determinisztikus tesztek](04-async-time-and-flaky-tests.md)
5. [Refaktorálás és regresszióvédelem](05-refactoring-and-regressions.md)
6. [Code review és indokolt minőségi ellenőrzések](06-review-and-quality-gates.md)

## Hogyan dolgozz vele?

1. Olvasd el a magyarázatot, és indokold meg a tervezési döntést.
2. Futtasd az egész példablokkot a lent leírt módon.
3. Válaszolj angolul az interjúkérdésekre, majd ellenőrizd a magyar válaszvázlattal.
4. Oldd meg a [kapcsolódó feladatsort](../../../exercises/senior-python/07-testing-and-quality.md); kész megoldások nincsenek mellékelve.
5. Review után frissítsük az elsajátítás állapotát.

## Futtatás

Minden Python-blokk önálló pytest-tesztmodul. Másold egy külön `test_example.py` fájlba, majd futtasd: `python -m pytest -q test_example.py`. A puszta `python test_example.py` nem futtatja le a tesztfüggvényeket. Az async példa szinkron teszten belül használ `asyncio.run`-t; nem kell pytest-asyncio plugin.

Az ellenőrzött verziókat és futtatási eredményeket az [ellenőrzési jegyzet](../VALIDATION.md) rögzíti. Külön gyakorlókörnyezetben ezekkel a parancsokkal indulhatsz:

```sh
python -m venv .venv
# Aktiválás Windows PowerShellben: .venv\Scripts\Activate.ps1
# Aktiválás Linux/WSL alatt: source .venv/bin/activate
python -m pip install pytest==9.1.1 httpx==0.28.1
```

A példák Linuxon, CPython 3.12-vel ellenőrzöttek. A HTTP-példák in-process vagy mock transportot használnak; valódi DNS-, TLS-, proxy-, hálózati timeout- és terhelésellenőrzés nem történt. Az assertionök oktatási ellenőrzések, éles inputvalidációt nem helyettesítenek.

Kapcsolódás: a [logelemző CLI](../../../projects/01-log-analyzer/README.md) adja a tesztelési gyakorlatok egyik későbbi terepét.

[Teljes tudástérkép](../README.md)
