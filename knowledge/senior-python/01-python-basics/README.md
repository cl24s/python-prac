# Python működése és nyelvi alapok

Állapot: kidolgozott első változat. A tananyag elkészülte nem jelenti a tudás elsajátítását; a tanulási állapotot a [ROADMAP](../../../ROADMAP.md) vezeti.

## Tanulási sorrend

| Sorrend | Fejezet | Fókusz |
| --- | --- | --- |
| 1 | [Objektumok, referenciák és mutability](01-object-model.md) | Azonos objektum, egyenlő érték, mutáció és újrakötés. |
| 2 | [Másolás és adat-tulajdonlás](02-copying.md) | Shallow/deep copy, nested adatok, célzott leválasztás. |
| 3 | [Függvények, scope és closure](03-functions-and-scope.md) | Paraméterek, defaultok, LEGB, late binding. |
| 4 | [Iterálás és comprehensionök](04-iteration-and-comprehensions.md) | Iterable/iterator/generator, lazy feldolgozás, unpacking. |
| 5 | [Decoratorok](05-decorators.md) | Wrapper, metaadatok, sorrend és exceptionmegőrzés. |
| 6 | [Context managerek](06-context-managers.md) | Belépés, kilépés, takarítás és hibaút. |
| 7 | [Modulok és importok](07-modules-and-imports.md) | Package, import-mellékhatás, körkörös függőség. |
| 8 | [Memória és objektuméletciklus](08-memory-and-lifetime.md) | Élő referenciák, GC, ciklusok és erőforrások. |

## Hogyan használd?

1. Olvasd el a magyarázatot, majd futtatás előtt jósolj eredményt a kódhoz.
2. Futtasd az adott teljes Python-kódblokkot, és magyarázd meg az assertionöket.
3. Válaszolj hangosan angolul az interjúkérdésekre; a magyar válaszvázlat utána segít ellenőrizni magad.
4. Oldd meg a [kapcsolódó feladatokat](../../../exercises/senior-python/01-python-basics.md), majd kérj review-t.
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
