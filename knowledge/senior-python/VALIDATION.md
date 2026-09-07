# Ellenőrzési jegyzetek

## OOP és típusok – 2026-09-06

Dátum: 2026-09-06. Hatókör: a 3. és 4. témakör jelenlegi első változata.

### Környezet

- CPython 3.12.13.
- mypy 2.3.1, `--strict --no-incremental --show-error-codes`.
- Pydantic 2.13.4 az egyetlen Pydantic-példához.

### Eredmények

| Ellenőrzés | Eredmény |
| --- | --- |
| Önálló Python-blokkok futtatása | 21/21 sikeres |
| A 4. témakör statikus ellenőrzése | 13 blokk ellenőrizve |
| Pozitív típusos minták | 11/11 hiba nélkül |
| Szándékosan hibás típusos minták | 2/2 a várt hibakategóriával |

Az értékadási negatív minta `assignment`, a listavarianciás negatív minta `arg-type` hibát ad. Ezek runtime is futnak, hogy a statikus és dinamikus viselkedés különbsége látható legyen.

### Mit jelent ez?

- A példák önálló folyamatban futottak, a saját assertionjeikkel; nincsen közöttük rejtett futási állapot.
- Az OOP-példákra runtime ellenőrzés történt; a mypy-ellenőrzés a 4. témakör kódblokkjaira vonatkozik.
- A feladatoknak kiírása van, megoldásuk nincs; felhasználói feladatteljesítést ez a jegyzet nem igazol.
- A Pydantic-példa a fenti verzióban ellenőrzött. A többi példa nem igényel külső runtime csomagot.
- A típusellenőrzés nem bizonyít minden üzleti szabályt, adatbázis-garanciát vagy konkurens viselkedést.
- Az első két témakör korábbi 29 runtime-példájának ellenőrzése az előző munkamenethez tartozik.

[OOP](03-oop/README.md) · [Típusok és interfészek](04-types-and-interfaces/README.md) · [Tudástérkép](README.md)

## Hibakezelés és concurrency – 2026-09-07

Hatókör: az 5. és 6. témakör 12 fejezete.

### Környezet és eredmények

- CPython 3.12.13, Linux, standard library; új külső függőség nélkül.
- **16/16 Python-kódblokk sikeresen lefutott**, minden blokk külön ideiglenes `.py` fájlban és önálló folyamatban, saját assertionjeivel.
- `PYTHONASYNCIODEBUG=1` és `-W error` aktív volt; a futtató blokkonként 20 másodperces felső korlátot alkalmazott.
- Az async példák ellenőrzik a taskok együttfutását, a TaskGroup hibaterjedését, a cancellation utáni cleanupot és a queue sikeres kiürítését.
- A threadpéldák determinisztikusan előidézett elvesző módosítást, lockkal védett számlálót és a futó thread asyncio future-től független életciklusát ellenőrzik.
- A processzpoolos példa explicit `spawn` móddal, fájlból, main guarddal futott; sikeres eredményt és a worker kivételének továbbítását is ellenőrzi.

### Határok

- Ez a futtatás az új 16 blokkra vonatkozott; a korábbi 50 példát ebben a munkamenetben nem futtattuk újra.
- Az új témakörökön külön statikus típusellenőrzés nem történt.
- A tesztek helyi, kontrollált hibaforgatókönyveket vizsgálnak; nem bizonyítanak éles szolgáltatási, tartóssági vagy teljesítménygaranciát.
- Windows és free-threaded CPython nem volt tesztelve. A Linuxon végzett explicit spawn-ellenőrzés nem helyettesít platformonkénti ellenőrzést.
- A 12 új önálló feladat csak kiírás; megoldás és felhasználói teljesítés nincs hozzá rögzítve.

[Hibakezelés](05-error-handling/README.md) · [Concurrency](06-concurrency/README.md) · [Tudástérkép](README.md)
