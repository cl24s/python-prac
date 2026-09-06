# Ellenőrzési jegyzet – OOP és típusok

Dátum: 2026-09-06. Hatókör: a 3. és 4. témakör jelenlegi első változata.

## Környezet

- CPython 3.12.13.
- mypy 2.3.1, `--strict --no-incremental --show-error-codes`.
- Pydantic 2.13.4 az egyetlen Pydantic-példához.

## Eredmények

| Ellenőrzés | Eredmény |
| --- | --- |
| Önálló Python-blokkok futtatása | 21/21 sikeres |
| A 4. témakör statikus ellenőrzése | 13 blokk ellenőrizve |
| Pozitív típusos minták | 11/11 hiba nélkül |
| Szándékosan hibás típusos minták | 2/2 a várt hibakategóriával |

Az értékadási negatív minta `assignment`, a listavarianciás negatív minta `arg-type` hibát ad. Ezek runtime is futnak, hogy a statikus és dinamikus viselkedés különbsége látható legyen.

## Mit jelent ez?

- A példák önálló folyamatban futottak, a saját assertionjeikkel; nincsen közöttük rejtett futási állapot.
- Az OOP-példákra runtime ellenőrzés történt; a mypy-ellenőrzés a 4. témakör kódblokkjaira vonatkozik.
- A feladatoknak kiírása van, megoldásuk nincs; felhasználói feladatteljesítést ez a jegyzet nem igazol.
- A Pydantic-példa a fenti verzióban ellenőrzött. A többi példa nem igényel külső runtime csomagot.
- A típusellenőrzés nem bizonyít minden üzleti szabályt, adatbázis-garanciát vagy konkurens viselkedést.
- Az első két témakör korábbi 29 runtime-példájának ellenőrzése az előző munkamenethez tartozik.

[OOP](03-oop/README.md) · [Típusok és interfészek](04-types-and-interfaces/README.md) · [Tudástérkép](README.md)
