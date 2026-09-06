# Típusok és interfészek

Állapot: kidolgozott első változat; az elsajátítást külön a [ROADMAP](../../../ROADMAP.md) követi.

A típusos részhez az első két témakör és az OOP alapjai szükségesek. A statikus szerződés mellett mindig vizsgáld a runtime viselkedést is.

## Tanulási sorrend

| Sorrend | Fejezet |
| --- | --- |
| 1 | [Type hint-ek és statikus ellenőrzés](01-type-hints-and-checking.md) |
| 2 | [Any, object, union és típusszűkítés](02-any-object-and-narrowing.md) |
| 3 | [Protocol, ABC és strukturális típusosság](03-protocols-and-abcs.md) |
| 4 | [Generikus típusok, variancia és kollekciós interfészek](04-generics-and-variance.md) |
| 5 | [TypedDict, dataclass és runtime validáció](05-data-models-and-validation.md) |
| 6 | [Callable, ParamSpec, Self és típusos API-tervezés](06-signatures-and-type-design.md) |

## Használat

1. Olvasd el a magyarázatot, majd futtatás előtt mondd meg a példa várt eredményét.
2. Minden teljes Python-blokk önálló példa, a szükséges importokkal együtt.
3. Az angol interjúkérdésre saját választ adj, majd hasonlítsd össze a magyar válaszvázlattal.
4. Oldd meg az [önálló feladatsort](../../../exercises/senior-python/04-types-and-interfaces.md); kész megoldás nincs mellékelve.
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
