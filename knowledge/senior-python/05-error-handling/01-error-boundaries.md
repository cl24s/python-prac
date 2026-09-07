# Hibakategóriák és a kezelés helye

## Mit kell tudnod?

- Megkülönböztetni a hibás bemenetet, üzleti elutasítást, infrastruktúrahibát és programozási hibát.
- Keskeny try-blokkot használni, amely nem rejti el más művelet hibáját.
- Tudni, melyik rétegnek van elég információja a helyreállításhoz.

## Magyarázat

Az exception önmagában nem jelenti, hogy a teljes alkalmazásnak le kell állnia. Azt jelzi, hogy az adott művelet nem tudta a szokásos visszatérési szerződését teljesíteni. A hívó akkor kezelje, ha érdemi döntése van: hibás sor kihagyása, felhasználói visszajelzés, korlátozott újrapróbálás vagy a munkamenet lezárása.

| Kategória | Példa | Tipikus döntés |
| --- | --- | --- |
| Hibás bemenet | Szám helyén szöveg | Pontos visszajelzés, nincs vak retry |
| Üzleti elutasítás | Már lezárt job indítása | Elutasítás a domain szerződése szerint |
| Infrastruktúrahiba | Nem nyitható meg a fájl | Megszakítás vagy tudatos fallback |
| Programozási hiba | Rossz attribútum, hibás invariáns | Látható hiba és javítás |

A kategória nem mindig olvasható ki egy built-in exception nevéből. Egy ValueError lehet inputhiba, de lehet a saját algoritmusod hibája is. A művelet határa és a dokumentált szerződés számít.

## Kódpélda: a parser hibája nem a callback hibája

```python
from collections.abc import Callable

def consume(text: str, save: Callable[[int], None]) -> bool:
    try:
        value = int(text)
    except ValueError:
        return False
    else:
        save(value)
        return True

saved: list[int] = []
assert consume('12', saved.append)
assert not consume('bad', saved.append)
assert saved == [12]

def broken_save(value: int) -> None:
    raise ValueError('storage contract failed')

try:
    consume('12', broken_save)
except ValueError as exc:
    assert str(exc) == 'storage contract failed'
else:
    raise AssertionError('Callback failure must propagate')
```

Ha a save is a try-blokkban lenne, a ValueError elnyelődne, mintha a bemenet lett volna hibás. Az else normál parse után fut, de a saját hibája nem kerül ugyanennek a try-nak a handlerébe. A False itt dokumentált „nem parse-olható rekord” eredmény, nem általános „valami rossz történt”.

## Hierarchia és tipikus hibák

Saját alkalmazáshibát rendszerint Exceptionből származtass. A BaseException szélesebb: többek között KeyboardInterrupt, SystemExit és az asyncio CancelledError is közvetlenül ezen az ágon van. A csupasz `except:` ezekbe is belenyúlhat. Normál rekordfeldolgozó ne nyelje el a folyamatleállítás vagy taskmegszakítás jelzését.

A szűkebb except ág legyen előbb; az első illeszkedő handler fut. Ha csak továbbdobás kell, az except blokkon belüli üres `raise` megőrzi a folyamatban lévő hibát. A „catch, log, raise” minden rétegben ugyanazt a hibát sokszor naplózhatja.

## Senior szempontok

Egy API vagy CLI belépési pontján lehet indokolt széles Exception-kezelés a folyamat végeredményének rögzítésére. Ez ne jelentsen sikeres választ ismeretlen hiba után. A belső könyvtár inkább adjon értelmes hibát, a határréteg döntse el a HTTP-státuszt vagy exit code-ot.

## Interview questions

**Where should an exception be handled?**

Válaszvázlat: ott, ahol a rétegnek van érdemi helyreállítási vagy fordítási döntése; nem automatikusan minden hívásnál.

**Why keep a try block small?**

Válaszvázlat: elkerülhető, hogy más művelet azonos típusú hibáját rossz kategóriaként kezeljük. Magyarázd el a save példáját.

## Önellenőrzés

- [ ] Nem rejtem el a callback hibáját inputhibaként.
- [ ] Megkülönböztetem az Exception és BaseException hatókörét.
- [ ] A fallback és az elutasítás része a szerződésemnek.


## Kapcsolódó gyakorlat

[E01 – önálló feladat](../../../exercises/senior-python/05-error-handling.md#e01)

## Forrás és továbbolvasás

[Errors and exceptions](https://docs.python.org/3.12/tutorial/errors.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
