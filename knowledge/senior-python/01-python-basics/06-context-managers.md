# Context managerek és erőforrás-életciklus

## Mit kell tudnod?

- Megérteni a `with`, `__enter__` és `__exit__` kapcsolatát.
- Erőforrást lezárni normál és hibás végrehajtásnál.
- Tudni, mikor nyelődik el egy exception.

## Magyarázat

A context manager egy blokkhoz köt belépési és kilépési viselkedést. Fájl, lock vagy ideiglenes állapot életciklusa így együtt látszik a használattal. Sikeres belépés után a `__exit__` normál befejezés, `return` és exception esetén is lefut a szokásos Python vezérlés során. Kényszerített processzleállítás vagy géphiba ellen ez nem tartóssági garancia.

A `with manager as value` esetén a `value` a `__enter__` visszatérési értéke. A `__exit__(exc_type, exc_value, traceback)` információt kap a blokkbeli hibáról. Igaz értékkel elnyomja azt; hamis értékkel vagy `None`-nal továbbengedi. Ha már a `__enter__` hibázik, az adott manager `__exit__` metódusa nem fut: a részleges belépés takarítása a belépő kód felelőssége.

## Kódpélda: saját manager

```python
from io import StringIO

class TextBuffer:
    def __enter__(self):
        self.buffer = StringIO()
        return self.buffer

    def __exit__(self, exc_type, exc_value, traceback):
        self.buffer.close()
        return False

manager = TextBuffer()
try:
    with manager as buffer:
        buffer.write('partial result')
        raise ValueError('invalid record')
except ValueError:
    pass
else:
    raise AssertionError('The exception must not be swallowed')
assert manager.buffer.closed
```

A példához nem kell előzetes mély OOP-tudás: az osztály csak megmutatja a protokoll két metódusát. A manager nem formál át egy hibás feldolgozást látszólag sikeressé.

## Egyszerűbb változat contextlib segítségével

```python
from contextlib import contextmanager
from io import StringIO

@contextmanager
def text_buffer():
    buffer = StringIO()
    try:
        yield buffer
    finally:
        buffer.close()

with text_buffer() as output:
    output.write('ready')
    assert output.getvalue() == 'ready'
assert output.closed
```

A `yield` előtti rész a belépés, az átadott érték kerül az `as` változóba, a folytatás a kilépés. Pontosan egyszer kell yieldelni. A blokkban keletkező exception a yield helyén jelenik meg; ha elkapod és nem dobod tovább, elnyelheted. A `finally` itt a takarítás helye.

## Mikor használnád és milyen hibákat keress?

- Fájlmegnyitásnál használd a beépített `with open(...)` formát, explicit encodinggal szöveghez.
- Több, futás közben eldőlő erőforráshoz az `ExitStack` kezeli a már sikeresen megszerzett erőforrások fordított sorrendű lezárását.
- **Hiba:** `__exit__` végén gondolkodás nélkül `True`. **Javítás:** csak kifejezetten kezelt hibát nyomj el.
- **Hiba:** a garbage collectorra bízott kapcsolatlezárás. **Javítás:** explicit életciklus.
- A tranzakciós manager rollback/commit szabálya külön szerződés; a `with` önmagában nem ad adatbázis-tranzakciót.
- Async erőforráshoz más protokoll kell: `async with`, `__aenter__`, `__aexit__`.

## Interview questions

**What happens if __enter__ raises an exception?**

Válaszvázlat: a blokk nem indul el, az adott `__exit__` nem fut; részleges megszerzést a belépési műveletnek kell takarítania.

**How can a context manager accidentally hide a failure?**

Válaszvázlat: truthy `__exit__` eredménnyel, vagy generator-alapú managerben elkapott, tovább nem dobott exceptionnel.

## Önellenőrzés

- [ ] A manageremet normál és hibás úton is ellenőrzöm.
- [ ] Tudom, melyik objektum kerül az `as` változóba.
- [ ] Külön kezelem a takarítást és az üzleti hiba kezelését.


## Kapcsolódó gyakorlat

[Önálló feladat: B05](../../../exercises/senior-python/01-python-basics.md#b05)

## Forrás és továbbolvasás

[contextlib](https://docs.python.org/3/library/contextlib.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
