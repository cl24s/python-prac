# HTTP és backend alapok

Állapot: kidolgozott első változat. Az elsajátítást külön a [ROADMAP](../../../ROADMAP.md) követi.

## Tanulási sorrend

1. [HTTP-metódusok, státuszkódok és idempotencia](01-http-semantics.md)
2. [Headerek, reprezentációk és feltételes kérések](02-headers-and-caching.md)
3. [Request-életciklus, middleware és hibaválaszok](03-request-lifecycle-and-errors.md)
4. [WSGI, ASGI, webszerver és reverse proxy](04-wsgi-asgi-and-proxies.md)
5. [Authentication, authorization és erőforrás-hozzáférés](05-authentication-and-authorization.md)
6. [Lapozás, API-verziózás és kompatibilitás](06-pagination-and-compatibility.md)
7. [HTTP-kliens, connection pool és timeoutkeret](07-outbound-clients-and-timeouts.md)

## Hogyan dolgozz vele?

1. Olvasd el a magyarázatot, és indokold meg a tervezési döntést.
2. Futtasd az egész példablokkot a lent leírt módon.
3. Válaszolj angolul az interjúkérdésekre, majd ellenőrizd a magyar válaszvázlattal.
4. Oldd meg a [kapcsolódó feladatsort](../../../exercises/senior-python/08-http-and-backend.md); kész megoldások nincsenek mellékelve.
5. Review után frissítsük az elsajátítás állapotát.

## Futtatás

A Python-blokkok önálló scriptek: másold egy külön `example.py` fájlba, majd futtasd `python example.py` paranccsal. A 4. és 7. fejezet HTTPX-et használ; a többi példa standard library-alapú. A metódusokról és headerekről szóló első két fejezet tervezési anyag, nem futtatható HTTP-szerver.

Az ellenőrzött verziókat és futtatási eredményeket az [ellenőrzési jegyzet](../VALIDATION.md) rögzíti. Külön gyakorlókörnyezetben ezekkel a parancsokkal indulhatsz:

```sh
python -m venv .venv
# Aktiválás Windows PowerShellben: .venv\Scripts\Activate.ps1
# Aktiválás Linux/WSL alatt: source .venv/bin/activate
python -m pip install pytest==9.1.1 httpx==0.28.1
```

A példák Linuxon, CPython 3.12-vel ellenőrzöttek. A HTTP-példák in-process vagy mock transportot használnak; valódi DNS-, TLS-, proxy-, hálózati timeout- és terhelésellenőrzés nem történt. Az assertionök oktatási ellenőrzések, éles inputvalidációt nem helyettesítenek.

Kapcsolódás: az [endpoint checker](../../../projects/02-endpoint-checker/README.md) a kliensoldalt, a [job processing API](../../../projects/03-job-processing-api/README.md) a szerveroldalt gyakoroltatja. A következő kidolgozandó témakör a **9. FastAPI**.

[Teljes tudástérkép](../README.md)
