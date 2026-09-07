# Részleges hibák, retry és eredménybizonytalanság

## Mit kell tudnod?

- Megkülönböztetni a „nem sikerült” és a „nem tudjuk, sikerült-e” esetet.
- Retry-t csak megfelelő műveletre, limitált költségvetéssel alkalmazni.
- Rekordonkénti folytatást explicit eredménnyel dokumentálni.

## Magyarázat

Ha egy távoli szolgáltatás eltárolja a kérést, de a válasz elveszik, a timeout nem bizonyítja, hogy nem történt mellékhatás. Egy újabb küldés duplikálhatja a műveletet. A retry biztonságát ezért nemcsak az exception típusa, hanem az operáció idempotenciája és az azonosító-kezelés is meghatározza.

Három külön politika: fail-fast az első hibánál; best-effort feldolgozás rekordonkénti hibalistával; vagy mindent-egyben atomikus változtatás megfelelő tranzakcióval. Egy for-ciklus try/excepttel nem alakít több távoli műveletet tranzakcióvá.

## Kódpélda: korlátos retry, valódi várakozás nélkül tesztelve

```python
from collections.abc import Callable
from typing import TypeVar

T = TypeVar('T')

class TemporaryReadError(Exception):
    pass

def retry_read(operation: Callable[[], T], pause: Callable[[float], None],
               attempts: int = 3) -> T:
    if isinstance(attempts, bool) or not isinstance(attempts, int):
        raise TypeError('attempts must be an integer')
    if attempts < 1:
        raise ValueError('attempts must be positive')
    for attempt in range(attempts):
        try:
            return operation()
        except TemporaryReadError:
            if attempt + 1 == attempts:
                raise
            pause(min(0.1 * 2 ** attempt, 1.0))
    raise AssertionError('unreachable')

calls = 0
pauses: list[float] = []

def read_version() -> str:
    global calls
    calls += 1
    if calls < 3:
        raise TemporaryReadError('read temporarily unavailable')
    return 'v3'

assert retry_read(read_version, pauses.append) == 'v3'
assert calls == 3
assert pauses == [0.1, 0.2]
```

A példában olvasási műveletet ismétlünk, csak a megnevezett átmeneti hibára. A harmadik próbálkozás után nincs újabb várakozás. A pause injektálása determinisztikussá teszi a tesztet, de éles használatban valódi várakozásra kell kötni. Globális számláló csak a rövid példa mérőeszköze.

## Mi hiányzik egy éles retry-rendszerhez?

Teljes határidő, kísérletenkénti timeout, megszakítható várakozás és több kliensnél jitter. A backoff nem védi meg a szolgáltatást, ha közben korlátlan feladatot fogadunk. Több réteg egymásra épülő retry-ja megsokszorozhatja a próbálkozásokat; egyértelműen jelöld, mely réteg birtokolja a retry-t.

A példát ne használd változtatás nélkül async függvényre: a coroutine létrehozása nem a művelet végrehajtása, a sync sleep pedig blokkolná az event loopot. Ezeket a következő témakör külön kezeli.

## Tipikus hibák és senior szempontok

- **Hiba:** minden Exception retry-t kap. **Javítás:** szűk, dokumentált hibakör és megfelelő operációs szerződés.
- **Hiba:** a folyamat végén csak az utolsó siker látszik, az elutasított rekordok eltűnnek. **Javítás:** számlálók vagy tételes eredmények, az input sorrendjéhez köthetően.
- Kompenzáló művelet nem feltétlenül pontos rollback és maga is hibázhat. Ne ígérj atomikusságot pusztán „visszacsináló” API-val.
- A timeout utáni eredménybizonytalanság legyen megfigyelhető; a felhasználónak ne állíts biztos sikert vagy biztos sikertelenséget bizonyíték nélkül.

## Interview questions

**Why can retrying after a timeout duplicate a write?**

Válaszvázlat: a szerver végrehajthatta a műveletet, csak a válasz veszett el; idempotencia és eredményellenőrzés szükséges lehet.

**What limits should a retry policy have?**

Válaszvázlat: próbálkozásszám, teljes határidő, kísérlet-timeout, backoff/jitter és megszakíthatóság; ownership egy rétegnél.

## Önellenőrzés

- [ ] A retry műveletszemantikája dokumentált.
- [ ] A kimerült retry az eredeti utolsó hibát továbbítja.
- [ ] A részleges eredmény és elutasítás látható marad.


## Kapcsolódó gyakorlat

[E04 – önálló feladat](../../../exercises/senior-python/05-error-handling.md#e04)

## Forrás és továbbolvasás

[Exception propagation](https://docs.python.org/3.12/tutorial/errors.html#handling-exceptions) · [Timeout semantics](https://docs.python.org/3.12/library/asyncio-task.html#timeouts)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
