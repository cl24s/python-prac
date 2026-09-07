# FastAPI

Állapot: kidolgozott első változat. Az elsajátítást külön a [ROADMAP](../../../ROADMAP.md) követi.

## Tanulási sorrend

1. [Routing, HTTP-modellek és OpenAPI](01-routing-and-openapi.md)
2. [Pydantic v2: validáció, szerializáció és részleges frissítés](02-pydantic-validation.md)
3. [Depends, függőségláncok és erőforrás-életciklus](03-dependencies-and-cleanup.md)
4. [async def, def és blokkoló függőségek](04-async-and-blocking.md)
5. [Lifespan, alkalmazásállapot és klienspoolok](05-lifespan-and-clients.md)
6. [Exception handlerek, middleware és hibakontextus](06-errors-and-middleware.md)
7. [Hitelesítési dependency és objektumszintű authorization](07-security-and-policies.md)
8. [FastAPI és SQLAlchemy: session, tranzakció és HTTP-válasz](08-database-boundaries.md)
9. [BackgroundTasks és tartós háttérfeldolgozás](09-background-work.md)
10. [FastAPI-tesztek, override és integrációs határok](10-api-testing.md)
11. [Éles futtatás, kapacitás, megfigyelés és graceful shutdown](11-deployment-and-shutdown.md)

## Hogyan dolgozz vele?

1. Mondd el saját szavaiddal a fejezet működését és döntési pontjait.
2. Másold a teljes Python-blokkot külön `test_example.py` fájlba.
3. Futtasd: `python -m pytest -q test_example.py`; a sima `python test_example.py` nem futtatja a tesztfüggvényeket.
4. Válaszolj angolul az interjúkérdésekre, és ellenőrizd a magyar válaszvázlattal.
5. Oldd meg a [kapcsolódó feladatsort](../../../exercises/senior-python/09-fastapi.md), majd review után frissítsük a haladást. Kész megoldás nincs mellékelve.

## Ellenőrzött környezet

CPython 3.12.13, Linux. Külön gyakorlókörnyezetet használj; a következő verziók a két témakör együtt ellenőrzött környezetét írják le.

```sh
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
# Linux / WSL: source .venv/bin/activate
python -m pip install fastapi==0.141.1 starlette==1.6.0 pydantic==2.13.5 sqlalchemy==2.0.52 pytest==9.1.1 httpx==0.28.1 httpx2==2.12.0 anyio==4.15.1
```

A FastAPI TestClientet a Starlette biztosítja; a használt változatban `httpx2` szükséges a nem elavult kliensúthoz. A lifespan-fejezet saját upstream példája továbbra is `httpx`-et importál, ezért mindkettő szerepel a környezetben. [Starlette TestClient](https://starlette.dev/testclient/)

A 11. fejezet deployment-terv, nincs benne futtatható kód. A többi FastAPI-fejezet egy-egy önálló tesztmodul. A bearer tokenek demonstrációs tesztadatok; éles JWT-verifikáció nincs implementálva.

A példák in-process HTTP-t vagy helyi adatbázist használnak, nem ellenőrzik a DNS-t, TLS-t, proxyt, éles pool-kimerülést vagy processzösszeomlást. Az assertionök oktatási ellenőrzések. Az [ellenőrzési jegyzet](../VALIDATION.md) tartalmazza a futtatási eredményt és a függőségből érkező deprecation warningot.

## Projektkapcsolat

A [job processing API](../../../projects/03-job-processing-api/README.md) később együtt használja a FastAPI-, adatbázis- és háttérfeldolgozási tudást. A jelenlegi anyag kis, önálló mintákból áll; nem kész projektmegoldás.

[Teljes tudástérkép](../README.md)
