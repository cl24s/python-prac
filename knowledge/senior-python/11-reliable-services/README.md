# Megbízható szolgáltatások és háttérfeldolgozás

Állapot: kidolgozott első változat. Az elsajátítást külön a [ROADMAP](../../../ROADMAP.md) követi.

## Tanulási sorrend

1. [Részleges hibák, eredménybizonytalanság és időkeretek](01-failure-models-and-deadlines.md)
2. [Retry-szabály, backoff, jitter és retrykeret](02-retry-backoff-and-jitter.md)
3. [Idempotens fogyasztó és tartós deduplikáció](03-idempotent-consumers.md)
4. [Queue, acknowledgement, retry és dead-letter feldolgozás](04-queues-ack-and-dead-letters.md)
5. [Outbox, publikálási bizonytalanság és helyreállítás](05-outbox-and-recovery.md)
6. [Túlterhelés, bulkhead és circuit breaker](06-overload-and-circuit-breakers.md)
7. [Worker-életciklus, drain és megszakítás](07-worker-lifecycle-and-shutdown.md)

## Hogyan dolgozz vele?

1. Fogalmazd meg a fejezet működési vagy hibamodelljét saját szavaiddal.
2. Másold a teljes Python-blokkot külön `test_example.py` fájlba.
3. Futtasd: `python -m pytest -q test_example.py`. A sima `python test_example.py` nem indítja a tesztfüggvényeket.
4. Válaszolj angolul az interjúkérdésekre, majd ellenőrizd a magyar válaszvázlattal.
5. Oldd meg a [kapcsolódó feladatsort](../../../exercises/senior-python/11-reliable-services.md). Kész megoldás nincs mellékelve; review után frissítsük az elsajátítást.

## Futtatókörnyezet

CPython 3.12.13 és pytest 9.1.1 alatt, Linuxon ellenőrzött példák. A példák futási logikája kizárólag standard libraryt használ; pytest a tesztfuttató.

```sh
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
# Linux / WSL: source .venv/bin/activate
python -m pip install pytest==9.1.1
python -m pytest -q test_example.py
```

Nem kell RabbitMQ, Celery vagy külső szolgáltatás a példákhoz. Az ack/retry/breaker modellek helyi szabályokat, a SQLite-minták helyi tranzakciót és újrakapcsolódást ellenőriznek. Nem igazolnak valódi brokerkonfigurációt, többworkerű race-eket vagy processz-/gépkiesés utáni tartósságot.

A [FastAPI háttérmunka](../09-fastapi/09-background-work.md) és az [adatbázis-tranzakciók](../10-databases/03-transactions-and-connections.md) anyagára építünk; a [job processing API](../../../projects/03-job-processing-api/README.md) lesz a későbbi integrációs gyakorlótér.

Az assertionök oktatási ellenőrzések. A pontos eredmény és a mérési korlátok az [ellenőrzési jegyzetben](../VALIDATION.md) szerepelnek.

[Teljes tudástérkép](../README.md)
