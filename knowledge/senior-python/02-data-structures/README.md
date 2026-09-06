# Adatszerkezetek és algoritmikus gondolkodás

Állapot: kidolgozott első változat. A tananyag elkészülte nem jelenti a tudás elsajátítását; a tanulási állapotot a [ROADMAP](../../../ROADMAP.md) vezeti.

## Tanulási sorrend

| Sorrend | Fejezet | Fókusz |
| --- | --- | --- |
| 1 | [Szekvenciák](01-sequences.md) | List/tuple, szeletelés, törlés, str/bytes. |
| 2 | [Dict, set és hash-elhetőség](02-hashing-and-mappings.md) | Kulcsszerződés, sorrend, hiány és duplikáció. |
| 3 | [Specializált kollekciók](03-specialized-collections.md) | Deque, Counter, defaultdict és heap. |
| 4 | [Idő- és memóriakomplexitás](04-complexity.md) | Big O, átlagos/amortizált költség, indexépítés. |
| 5 | [Rendezés, keresés és csoportosítás](05-sorting-searching-grouping.md) | Stabilitás, bisect, groupby és deduplikáció. |
| 6 | [Nagy adatok és streaming](06-streaming-and-bounded-memory.md) | Egy bejárás, kardinalitás, batch és top-k. |

## Hogyan használd?

1. Olvasd el a magyarázatot, majd futtatás előtt jósolj eredményt a kódhoz.
2. Futtasd az adott teljes Python-kódblokkot, és magyarázd meg az assertionöket.
3. Válaszolj hangosan angolul az interjúkérdésekre; a magyar válaszvázlat utána segít ellenőrizni magad.
4. Oldd meg a [kapcsolódó feladatokat](../../../exercises/senior-python/02-data-structures.md), majd kérj review-t.
5. A roadmap csak megértésellenőrzés és feladatmegoldás alapján kapjon kész állapotot.

## Példák és futtatókörnyezet

- A példák önálló, standard library-alapú Python-kódblokkok. Nincs külső csomagtelepítési igény.
- Ellenőrzött környezet: CPython 3.12.13. A teljes első két témakörben 29 blokk futtatása sikeres volt.
- Egy blokkot egy ideiglenes `.py` fájlba másolva a `python example.py` paranccsal futtathatsz. Az importos példa maga hozza létre az ideiglenes package-et.
- A hibás viselkedést bemutató példák is futtathatók: az elvárt hibákat elkapják és ellenőrzik.
- Az `assert` itt oktatási ellenőrzés. Éles inputvalidációt ne kizárólag assertionre építs, mert optimalizált futtatáskor kikapcsolható.
- A hivatkozott hivatalos dokumentáció `/3/` útvonala az aktuális Python-dokumentációt mutatja; a példák tényleges ellenőrzési verziója a fenti.
- Ahol CPython-specifikus viselkedésről van szó, azt a fejezet jelzi. A részletes OOP, typing és async anyag későbbi témakör.

## A témakör végére

- [ ] Saját példán is megmagyarázom a fő fogalmakat.
- [ ] Hibás kódnál nemcsak a javítást, hanem az okot is megnevezem.
- [ ] A megfelelő feladatok elfogadási feltételeit teljesítettem.
- [ ] Legalább egy megoldási kompromisszumot angolul is elmagyarázok.

[Teljes tudástérkép](../README.md)
