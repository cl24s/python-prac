# Diagnosztika és hibás végrehajtási utak ellenőrzése

## Mit kell tudnod?

- Hasznos hibakontextust rögzíteni titkok és teljes payloadok nélkül.
- Visszatérési eredmény mellett a mellékhatásokat és azok hiányát is ellenőrizni.
- Elkülöníteni a megfigyelést az exception kezelésétől.

## Magyarázat

A jó hibanapló megmutatja, melyik művelet hibázott, milyen korrelációs azonosítóval és milyen okból. Nem kell minden rétegben ugyanaz a traceback. Általában az a határ naplózzon teljes hibát, ahol a művelet életciklusa lezárul vagy a hibát ténylegesen kezelik. A belső réteg adhat strukturált kontextust és továbbdobhatja az exceptiont.

A `logger.exception()` except blokkon belül a jelenlegi exception információját is rögzíti. A `%s` paraméteres naplózás elkerüli a formázás előzetes munkájának egy részét, de önmagában nem maszkol titkokat. Az exception üzenete és a kapcsolt ok is tartalmazhat érzékeny információt.

## Kódpélda: egyszeri határnapló és továbbdobás

```python
import io
import logging

output = io.StringIO()
logger = logging.getLogger('lesson.error_boundary')
handler = logging.StreamHandler(output)
handler.setFormatter(logging.Formatter('%(levelname)s %(message)s'))
logger.addHandler(handler)
logger.setLevel(logging.ERROR)
logger.propagate = False

failure = OSError('fixture unavailable')
try:
    try:
        raise failure
    except OSError:
        logger.exception('operation=load_config request_id=%s', 'req-17')
        raise
except OSError as caught:
    assert caught is failure
finally:
    logger.removeHandler(handler)
    handler.close()

text = output.getvalue()
assert text.count('operation=load_config') == 1
assert 'request_id=req-17' in text
assert 'OSError: fixture unavailable' in text
```

A logoló kód nem változtatja a hibát sikeres visszatéréssé. A teszt StringIO-ba naplóz; éles alkalmazásban a handlereket egyszer, a konfigurációs rétegben állítsd be, ne minden kérésnél.

## Hibamátrix

| Eset | Mit ellenőrizz? |
| --- | --- |
| Hibás bemenet | Pontos elutasítás, nincs mellékhatás |
| Első függőség hibázik | A következő művelet nem indul |
| Tárolás sikerül, értesítés hibázik | A részleges állapot dokumentált |
| Lezárás közbeni hiba | Az elsődleges és cleanup-hiba diagnosztizálható |
| Retry kimerül | Pontos hívásszám és utolsó exception |
| Megszakítás | A cleanup lefut, a megszakítás nem tűnik el |

A fake a hibát is tudja előállítani, ne csak a happy pathot. A teszt ne kötődjön a traceback minden sorához: a típus, a releváns mező, a chaining és a jelentős mellékhatás stabilabb szerződés.

## Több hiba egyszerre

Python 3.11-től az ExceptionGroup több hibát őrizhet egy strukturált eredményben. Az `except*` részhalmazokra illeszkedik; a nem kezelt hibák továbbterjednek. Ez főként több párhuzamos task közös életciklusánál fontos, ezért a [TaskGroup-fejezet](../06-concurrency/03-task-groups-and-failures.md) futtatható példában tárgyalja.

## Tipikus hibák és senior szempontok

- **Hiba:** `except Exception: return None`, majd a hívó ezt „nincs adat”-ként értelmezi. **Javítás:** külön hibajelzés és hiányzó eredmény.
- **Hiba:** csak a kivétel szövegét ellenőrzöd. **Javítás:** típus és szerződés, például nincs második küldés.
- Metrikacímkébe ne kerüljön korlátlan request ID vagy teljes hibaüzenet; a kardinalitás is erőforrásköltség.
- Loghiba sem teheti megbízhatóvá vagy atomikussá az üzleti műveletet. A megfigyelhetőség és helyesség külön felelősség.

## Interview questions

**What should an error-path test verify besides the exception type?**

Válaszvázlat: állapot, hívásszám, elmaradó mellékhatás, cleanup és eredeti ok megőrzése.

**Why avoid logging and rethrowing at every layer?**

Válaszvázlat: duplikált zaj és félrevezető eseményszám; egyértelmű határ és korrelálható kontextus kell.

## Önellenőrzés

- [ ] A napló nem tartalmaz teljes requestet vagy titkot.
- [ ] A hibaút tesztje a mellékhatás hiányát is bizonyítja.
- [ ] A logging nem nyeli el az eredeti exceptiont.


## Kapcsolódó gyakorlat

[E05 – önálló feladat](../../../exercises/senior-python/05-error-handling.md#e05)

## Forrás és továbbolvasás

[logging](https://docs.python.org/3.12/library/logging.html) · [Exception groups](https://docs.python.org/3.12/library/exceptions.html#exception-groups)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
