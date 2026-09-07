# Adatbázisok és adatkezelés

Állapot: kidolgozott első változat. Az elsajátítást külön a [ROADMAP](../../../ROADMAP.md) követi.

## Tanulási sorrend

1. [SQL: join, aggregáció, NULL és adatinvariánsok](01-sql-and-constraints.md)
2. [Indexek, lekérdezési tervek és mérés](02-indexes-and-query-plans.md)
3. [Tranzakcióhatár, rollback és kapcsolatélettartam](03-transactions-and-connections.md)
4. [Izoláció, lockolás és elvesző módosítások](04-isolation-and-concurrent-writes.md)
5. [ORM, identity map, lazy betöltés és N+1](05-orm-and-n-plus-one.md)
6. [Sémamigráció, backfill és kompatibilis telepítés](06-migrations-and-compatibility.md)
7. [Cache, invalidálás és konzisztencia](07-cache-and-consistency.md)

## Hogyan dolgozz vele?

1. Mondd el saját szavaiddal a fejezet működését és döntési pontjait.
2. Másold a teljes Python-blokkot külön `test_example.py` fájlba.
3. Futtasd: `python -m pytest -q test_example.py`; a sima `python test_example.py` nem futtatja a tesztfüggvényeket.
4. Válaszolj angolul az interjúkérdésekre, és ellenőrizd a magyar válaszvázlattal.
5. Oldd meg a [kapcsolódó feladatsort](../../../exercises/senior-python/10-databases.md), majd review után frissítsük a haladást. Kész megoldás nincs mellékelve.

## Ellenőrzött környezet

CPython 3.12.13, Linux. Külön gyakorlókörnyezetet használj; a következő verziók a két témakör együtt ellenőrzött környezetét írják le.

```sh
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
# Linux / WSL: source .venv/bin/activate
python -m pip install fastapi==0.141.1 starlette==1.6.0 pydantic==2.13.5 sqlalchemy==2.0.52 pytest==9.1.1 httpx==0.28.1 httpx2==2.12.0 anyio==4.15.1
```

A FastAPI TestClientet a Starlette biztosítja; a használt változatban `httpx2` szükséges a nem elavult kliensúthoz. A lifespan-fejezet saját upstream példája továbbra is `httpx`-et importál, ezért mindkettő szerepel a környezetben. [Starlette TestClient](https://starlette.dev/testclient/)

A SQL-példák sqlite3-at, az ORM-fejezet SQLAlchemy 2-t használ. PostgreSQL-re vonatkozó részletek külön jelölve szerepelnek; PostgreSQL-szerverrel és Redis-szel nem futott integrációs teszt. Az Alembic-terv nem végrehajtott migráció.

A példák in-process HTTP-t vagy helyi adatbázist használnak, nem ellenőrzik a DNS-t, TLS-t, proxyt, éles pool-kimerülést vagy processzösszeomlást. Az assertionök oktatási ellenőrzések. Az [ellenőrzési jegyzet](../VALIDATION.md) tartalmazza a futtatási eredményt és a függőségből érkező deprecation warningot.

## Projektkapcsolat

A [job processing API](../../../projects/03-job-processing-api/README.md) később együtt használja a FastAPI-, adatbázis- és háttérfeldolgozási tudást. A jelenlegi anyag kis, önálló mintákból áll; nem kész projektmegoldás.

[Teljes tudástérkép](../README.md)
