# OOP és objektumtervezés

Állapot: kidolgozott első változat; az elsajátítást külön a [ROADMAP](../../../ROADMAP.md) követi.

Az OOP-rész a felelősségeket, invariánsokat és együttműködést emeli ki. A Protocolt használó példákhoz a 4. témakör ad részletes típuselméleti hátteret.

## Tanulási sorrend

| Sorrend | Fejezet |
| --- | --- |
| 1 | [Osztályok, példányok és attribútumok](01-classes-and-instances.md) |
| 2 | [Metódusok, property-k és invariánsok](02-methods-and-invariants.md) |
| 3 | [Öröklődés, helyettesíthetőség és super()](03-inheritance-and-mro.md) |
| 4 | [Composition, polimorfizmus és dependency injection](04-composition-and-di.md) |
| 5 | [Dataclass, értékobjektum és egyenlőség](05-dataclasses-and-value-objects.md) |
| 6 | [SOLID és gyakori patternök Pythonban](06-solid-and-patterns.md) |
| 7 | [Objektumtervezési review és biztonságos refaktorálás](07-refactoring-and-contracts.md) |

## Használat

1. Olvasd el a magyarázatot, majd futtatás előtt mondd meg a példa várt eredményét.
2. Minden teljes Python-blokk önálló példa, a szükséges importokkal együtt.
3. Az angol interjúkérdésre saját választ adj, majd hasonlítsd össze a magyar válaszvázlattal.
4. Oldd meg az [önálló feladatsort](../../../exercises/senior-python/03-oop.md); kész megoldás nincs mellékelve.
5. Review után a roadmapben az elsajátítást a tényleges teljesítmény alapján frissítsük.

## Futtatás és ellenőrzés

- A blokkok kimásolt fájlból `python example.py` paranccsal futtathatók; az assertionök a példák viselkedését ellenőrzik.
- A `# typecheck: expect-error [...]` jelölés szándékosan hibás statikus mintát jelez. Ezek runtime demonstrációk is; a checkernek a jelzett hibakategóriát kell adnia.
- A többi típusos blokkra `python -m mypy --strict example.py` használható. A mypy külön fejlesztői eszköz, nem a Python része.
- Egyetlen blokk Pydantic 2.x-et igényel; ezt a saját fejezete jelzi. A többi példa standard library-alapú.
- Az ellenőrzés pontos verzióit és eredményét az [ellenőrzési jegyzet](../VALIDATION.md) tartalmazza.
- A dokumentáció `/3/` linkje az aktuális hivatalos Python-leírást mutatja; a példák tesztelt verziója a jegyzetben szerepel.
- Az assertion nem éles inputvalidátor. A külső input- és üzletiszabály-ellenőrzést explicit kód végezze.

## Önellenőrzés

- [ ] Az alapfogalmakat saját példán megmagyarázom.
- [ ] A hibás megoldás okát és javítását is megnevezem.
- [ ] A kapcsolódó feladatok szerződéseit teljesítem.
- [ ] A döntések költségét és korlátait angolul is elmagyarázom.

[Teljes tudástérkép](../README.md)
