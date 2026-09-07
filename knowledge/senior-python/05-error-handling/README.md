# Hibakezelés és erőforrás-kezelés

Állapot: kidolgozott első változat. A tananyag és az elsajátítás állapotát külön a [ROADMAP](../../../ROADMAP.md) követi.

## Tanulási sorrend

| Sorrend | Fejezet |
| --- | --- |
| 1 | [Hibakategóriák és a kezelés helye](01-error-boundaries.md) |
| 2 | [Saját exceptionök, chaining és hibakontextus](02-custom-errors-and-chaining.md) |
| 3 | [Erőforrás-tulajdonlás és megbízható takarítás](03-resource-cleanup.md) |
| 4 | [Részleges hibák, retry és eredménybizonytalanság](04-partial-failures-and-retry.md) |
| 5 | [Diagnosztika és hibás végrehajtási utak ellenőrzése](05-logging-and-error-tests.md) |

## Hogyan dolgozz vele?

1. Olvasd el a magyarázatot, és jósolj eredményt a példához.
2. Futtasd az egész kódblokkot önálló `.py` fájlként.
3. Válaszolj angolul az interjúkérdésekre, majd ellenőrizd a magyar válaszvázlattal.
4. Oldd meg a [kapcsolódó feladatsort](../../../exercises/senior-python/05-error-handling.md). A megoldások nincsenek előre mellékelve.
5. Review után a tényleges megértés és feladatteljesítés alapján frissítsük a roadmapet.

## Futtatás és korlátok

- CPython 3.12.13-on ellenőrzött, standard library-alapú példák; az `add_note` legalább Python 3.11-et igényel.
- Minden teljes kódblokk önálló `.py` fájlként futtatható.
- A fájlpélda ideiglenes könyvtárban dolgozik; az atomikus csere nem jelent összeomlás utáni tartóssági garanciát.
- A retry-példa helyi fake függőséget és befecskendezett várakozást használ; nem végez valódi hálózati kérést.
- Az assertionök oktatási ellenőrzések; éles inputvalidáció explicit hibát adjon.
- A pontos eredmények az [ellenőrzési jegyzetben](../VALIDATION.md) szerepelnek.

## Önellenőrzés

- [ ] A sikeres és hibás végrehajtást is elmagyarázom.
- [ ] Megnevezem a hiba kezelőjét és az erőforrás tulajdonosát.
- [ ] A cleanupot és az elmaradó mellékhatásokat is ellenőrzöm.
- [ ] Megindoklom, mikor biztonságos egy művelet újrapróbálása.

[Teljes tudástérkép](../README.md)
