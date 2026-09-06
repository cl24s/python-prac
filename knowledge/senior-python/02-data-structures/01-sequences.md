# Listák, tuple-ök és szekvenciák választása

## Mit kell tudnod?

- Műveletek és jelentés alapján választani `list` és `tuple` között.
- Felismerni a szeletelés, beszúrás és helyben módosítás költségét.
- Megkülönböztetni a szöveget a byte-sorozattól.

## Magyarázat

A lista rendezett, módosítható szekvencia. Jó alap változó hosszúságú eredménygyűjtéshez és index szerinti hozzáféréshez. A tuple rendezett, közvetlenül nem módosítható szekvencia; kényelmes rövid, összetartozó értékcsoporthoz. Sok mezős, nyilvános adatmodellnél a pozíciók megjegyzése helyett névvel rendelkező típus érthetőbb.

A CPython-lista dinamikusan méretezett referenciatömb. A végére írás tipikusan olcsó, az elejére beszúráskor viszont el kell mozdítani az elemeket. A tuple nem „mindig jobb lista”: más módosítási szerződést fejez ki. Egy listát tartalmazó tuple például továbbra sem alkalmas dictkulcsnak.

## Kódpélda: szeletelés és módosítás

```python
jobs = ['a', 'b', 'c', 'd']
window = jobs[1:3]
window.append('x')
assert jobs == ['a', 'b', 'c', 'd']
assert window == ['b', 'c', 'x']
assert jobs[-1] == 'd'

result = jobs.append('e')
assert result is None

endpoint = ('api.local', 443)
host, port = endpoint
assert host == 'api.local' and port == 443
assert ('api',) != 'api'
```

A listaszelet új külső listát ad, sekély másolással. A `list.sort()` és az `append()` helyben dolgozik, és `None`-t ad vissza: ne írd `jobs = jobs.append(...)` formában.

## Tipikus hiba: iterálás közbeni törlés

```python
bad = [1, 2, 2, 3]
for value in bad:
    if value == 2:
        bad.remove(value)
assert bad == [1, 2, 3]

original = [1, 2, 2, 3]
filtered = [value for value in original if value != 2]
assert filtered == [1, 3]
assert original == [1, 2, 2, 3]
```

A törléstől az elemek eltolódnak, miközben a bejáró halad; egy elem kimarad. Új szűrt lista egyszerűbb, ha a bemenetet meg kell őrizni. Ha helyben módosítás a szerződés, a már kiszámolt eredményt `original[:] = filtered` formában is visszaírhatod.

## Szöveg és bytes

A `str` Unicode kódpontok sorozata; a `bytes` kódolt byte-oké. Hálózati vagy fájlbeli határon explicit encodinggal alakíts közöttük. A karakterhossz nem azonos a byte-hosszal, és a kódpontok száma sem mindig azonos a felhasználó által egy jelnek látott karakterek számával.

```python
text = 'ár'
encoded = text.encode('utf-8')
assert len(text) == 2
assert len(encoded) == 3
assert encoded.decode('utf-8') == text
```

## Senior szempontok

- FIFO-feladatlistához a sorozatos `pop(0)` helyett nézd meg a `deque`-t.
- Nagy, egynemű numerikus adatnál a Python-objektumokból álló lista jelentős overheadet hordozhat; mérés után tömör reprezentáció indokolt lehet.
- Sok string összefűzéséhez a `''.join(parts)` világos, egyszeri eredményépítés; ne támaszkodj ismételt konkatenáció runtime-optimalizációjára.

## Interview questions

**When would you choose a tuple instead of a list?**

Válaszvázlat: rögzített értékcsoport, változtathatatlan közvetlen szerkezet, esetleg hash-elhető összetett kulcs; nem automatikusan mély immutable adat.

**Why can removing elements during list iteration skip items?**

Válaszvázlat: a törlés eltolja az indexeket, a bejáró közben előrelép. Szűréssel vagy külön módosítási menettel javítsd.

## Önellenőrzés

- [ ] Tudom, mely listaműveletek adnak új listát és melyek `None`-t.
- [ ] Minden megfelelő elemet eltávolítok kimaradás nélkül.
- [ ] Külön kezelem a kódpont- és byte-hosszt.


## Kapcsolódó gyakorlat

[Önálló feladat: D01](../../../exercises/senior-python/02-data-structures.md#d01)

## Forrás és továbbolvasás

[Sequence types](https://docs.python.org/3/library/stdtypes.html#sequence-types-list-tuple-range)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
