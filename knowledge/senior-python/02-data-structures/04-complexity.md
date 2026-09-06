# Idő- és memóriakomplexitás a gyakorlatban

## Mit kell tudnod?

- Megnevezni, mit jelent az `n`, `m` vagy `k` a saját algoritmusodban.
- Különválasztani az átlagos, amortizált és legrosszabb költséget.
- A gyorsabb keresés árát memóriában és előkészítésben is számolni.

## Magyarázat

A Big O azt jellemzi, hogyan nő a költség a bemenet méretével; nem konkrét milliszekundumot ad. Az O(n) algoritmus lehet lassabb egy kisebb adaton, mint egy jól optimalizált másik megoldás. A növekedési rend mégis megmutatja, hol lehet skálázási baj.

Az **átlagos** költség valamilyen feltételezett bemeneti viselkedésre épít. Az **amortizált** költség egy műveletsor összköltségét osztja el: néhány drága listabővítés mellett sok olcsó append fér el. Ez nem ugyanaz, mint hogy minden append konstans idejű.

## Gyakori műveletek

Az alábbiak a szokásos CPython-konténerek algoritmikus modelljét írják le. A hash és összehasonlítás költségét az egyszerű táblázat konstansnak tekinti; hosszú stringeknél vagy saját metódusoknál ezt külön számold.

| Művelet | Jellemző idő | Megjegyzés |
| --- | --- | --- |
| Lista indexelése | O(1) | Közvetlen elemhozzáférés |
| Lista append | amortizált O(1) | Egy átméretezés O(n) is lehet |
| Lista eleji beszúrás / `pop(0)` | O(n) | Referenciák eltolása |
| Lista tagságvizsgálata | O(n) | Legrosszabb esetben teljes bejárás |
| Listaszelet k elemmel | O(k) | O(k) új referencia tárigénye |
| Dict/set keresés | átlagosan O(1) | Kedvezőtlen esetben O(n) |
| Összehasonlításos rendezés | O(n log n) legrosszabb | A Python rendezése meglévő rendezettséget is kihasznál |
| Heap push/pop | O(log n) legrosszabb | Gyökérelem lekérése O(1) |
| `heapify` | O(n) | Nem ugyanaz, mint n külön push |

## Kódpélda: ismételt tagságvizsgálat

```python
def filter_slow(events, allowed):
    return [event for event in events if event in allowed]

def filter_indexed(events, allowed):
    allowed_set = set(allowed)
    return [event for event in events if event in allowed_set]

events = ['api', 'worker', 'api', 'scheduler']
allowed = ['api', 'scheduler']
assert filter_slow(events, allowed) == ['api', 'api', 'scheduler']
assert filter_indexed(events, allowed) == filter_slow(events, allowed)
assert events == ['api', 'worker', 'api', 'scheduler']
```

Ha n esemény és m engedélyezett név van, a listás keresés O(nm) lehet. A setes megoldás várhatóan O(n+m), de O(m) extra indexet épít, és továbbra is O(r) memóriát használ az r elemű kimenethez. Ha az engedélylista újrahasználható, az indexet a kéréseken kívül is létrehozhatod, az érvénytelenítés szabályával együtt.

## Mikor használnád?

Egy API minden visszaadott rekordnál újra átnézi az összes jogosult ID-t. A beágyazott munka előbb gond, mint a rövid változónevek vagy egy comprehension stílusa. Ugyanakkor egyszeri, apró keresésnél az indexépítés nem feltétlenül térül meg.

## Tipikus hibák és senior szempontok

- **Hiba:** „két ciklus, tehát O(n²)”. **Javítás:** egymás után futó két n-es ciklus O(n); beágyazásnál a méretek és határok számítanak.
- **Hiba:** „dict mindig O(1)”. **Javítás:** nevezd meg az átlagos esetet és a kulcsműveletek költségét.
- Az input, output és kiegészítő munkamemória külön tétel legyen.
- Mérésnél hasonlíts azonos szemantikát, reprezentatív méretet, több futást és memóriaigényt. Ne tegyél általános sebességállítást egyetlen apró teszt alapján.

## Interview questions

**What does amortized O(1) mean for list.append?**

Válaszvázlat: a műveletsor egészére jut konstans átlagos költség; egy adott átméretezés drágább lehet.

**How would you optimize repeated membership checks?**

Válaszvázlat: index/set előkészítés, O(nm) helyett várható O(n+m); memória, újrahasználat és invalidálás árának megnevezése.

## Önellenőrzés

- [ ] Külön megadom a két bemeneti méretet a példában.
- [ ] Nem hagyom ki az index felépítését az összköltségből.
- [ ] Elmagyarázom az amortizált és átlagos kifejezések különbségét.


## Kapcsolódó gyakorlat

[Önálló feladat: D04](../../../exercises/senior-python/02-data-structures.md#d04)

## Forrás és továbbolvasás

[CPython list implementation](https://github.com/python/cpython/blob/3.12/Objects/listobject.c) · [heapq](https://docs.python.org/3/library/heapq.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
