# Saját exceptionök, chaining és hibakontextus

## Mit kell tudnod?

- Értelmes hibahierarchiát tervezni fogyasztói döntésekhez.
- Megőrizni az eredeti okot `raise ... from ...` segítségével.
- Különválasztani a géppel használható adatot az emberi hibaüzenettől.

## Magyarázat

Saját exception akkor hasznos, ha a hívó másként akar reagálni rá, vagy a belső technikai hibát stabil alkalmazásszerződésre kell fordítani. Nem szükséges minden függvényhez külön hibatípus. Egy InvalidConfig és egy ConfigUnavailable eltérő döntést indokolhat: hibás tartalom versus nem elérhető forrás.

A diagnosztikai üzenet olvashatóságot ad, de a hívó ne szövegrészletekből következtesse a hibatípust. Használjon osztályt vagy strukturált mezőt. A hibában tárolt payload se legyen korlátlan méretű vagy érzékeny adatot tartalmazó objektumgráf.

## Kódpélda: fordítás és eredeti ok

```python
class ConfigError(Exception):
    pass

class InvalidConfig(ConfigError):
    def __init__(self, field: str) -> None:
        self.field = field
        super().__init__(f'invalid configuration field: {field}')

def parse_workers(text: str) -> int:
    try:
        workers = int(text)
    except ValueError as exc:
        raise InvalidConfig('workers') from exc
    if workers < 1:
        raise InvalidConfig('workers')
    return workers

try:
    parse_workers('many')
except InvalidConfig as exc:
    assert exc.field == 'workers'
    assert isinstance(exc.__cause__, ValueError)
    exc.add_note('configuration source: local test fixture')
    assert exc.__notes__ == ['configuration source: local test fixture']
else:
    raise AssertionError('Expected InvalidConfig')
assert parse_workers('3') == 3
```

A ValueError a `__cause__` alatt marad, miközben a publikus hiba InvalidConfig. Az `add_note` Python 3.11-től diagnosztikai szöveget adhat a meglévő exceptionhöz, új osztály nélkül. Ez nem stabil gépi API, és nem automatikus adatmaszkolás.

## Chaining-változatok

- `raise`: a jelenleg kezelt exception továbbdobása.
- `raise DomainError(...) from exc`: kifejezett oksági kapcsolat.
- Új exception dobása except belsejében `from` nélkül: implicit kontextus kapcsolódik hozzá.
- `raise DomainError(...) from None`: a szokásos traceback-megjelenítésben elrejti az implicit kontextust; nem töröl minden belső hivatkozást és nem biztonsági szűrő.

A `raise exc` hozzáadhat egy új továbbdobási helyet a tracebackhez; változatlan továbbadásra általában az üres raise tisztább.

## Mikor használnád és tipikus hibák

Vendor SDK ConnectionError-ját stabil alkalmazáshibára fordíthatod az adapterben. Ne minden Exceptiont alakíts automatikusan „átmeneti hibává”: így például TypeError is retry-t kaphatna egy kódhiba miatt.

**Hiba:** az eredeti ok eltűnik egy `raise RuntimeError('failed')` mögött. **Javítás:** szűk kivételkezelés és explicit chaining. **Hiba:** a hibaüzenetbe teljes request vagy hitelesítési adat kerül. **Javítás:** minimális biztonságos azonosító és diagnosztikai mezők.

## Senior szempontok

A hibafordítás ugyanúgy kompatibilitási felület, mint a visszatérési típus. Egy downstream timeout után az eredmény lehet bizonytalan: az operáció még sikerülhetett távol. A kivétel osztálya mellé a művelet szemantikáját is dokumentálni kell.

## Interview questions

**What is the purpose of explicit exception chaining?**

Válaszvázlat: stabil publikus hibát ad, miközben megőrzi a technikai okot a diagnosztikához.

**Does from None remove sensitive data from an exception?**

Válaszvázlat: nem; traceback-megjelenítési viselkedést befolyásol, nem általános adattörlés.

## Önellenőrzés

- [ ] A hívó üzenetparse-olás nélkül tud dönteni.
- [ ] A fordított hiba eredeti oka ellenőrizhető.
- [ ] Csak biztonságos, szükséges kontextust tárolok.


## Kapcsolódó gyakorlat

[E02 – önálló feladat](../../../exercises/senior-python/05-error-handling.md#e02)

## Forrás és továbbolvasás

[Built-in exceptions](https://docs.python.org/3.12/library/exceptions.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
