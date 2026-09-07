# Tesztstratégia és a megfelelő teszthatárok

## Mit kell tudnod?

- Kockázat alapján kiválasztani a teszt szintjét.
- Megkülönböztetni a unit-, integrációs, contract- és end-to-end teszt bizonyító erejét.
- A megfigyelhető viselkedést ellenőrizni belső metódushívások helyett.

## Magyarázat

Először azt mondd ki, milyen hibát akarsz észrevenni. Egy hibás összegzéshez nem kell HTTP-szerver. Egy rossz SQL-constraintet viszont nem fog kimutatni a listával helyettesített adatbázis. A teszt neve helyett a valóban bevont komponensek és az ellenőrzött szerződés számítanak.

| Szint | Saját projektpélda | Amit nem bizonyít |
| --- | --- | --- |
| Unit | A logszint szerinti összegzés | Fájlkódolás és CLI bekötése |
| Integrációs | Parser + valódi ideiglenes fájl | Éles fájlrendszer minden jogosultsága |
| Adapter contract | Ugyanaz a repository-szerződés két adapterrel | Teljes üzleti folyamat |
| End-to-end | Telepített CLI vagy API teljes útvonala | Minden belső ág helyessége |

A tesztpiramis hasznos irány: sok gyors, célzott ellenőrzés és kevesebb drága teljes folyamat. Nem kötelező százalékos elosztás. Egy adatbázis-központú szolgáltatásnál a valódi adatbázisos integrációs teszt különösen értékes lehet.

## Kódpélda: egy szabály, két eltérő teszthatár

Az egész blokkot `test_example.py` fájlba másold; futtatás: `python -m pytest -q test_example.py`.

```python
from pathlib import Path

def count_errors(lines: list[str]) -> int:
    return sum(line.startswith('ERROR ') for line in lines)

def count_file(path: Path) -> int:
    return count_errors(path.read_text(encoding='utf-8').splitlines())

def test_only_error_prefix_counts():
    assert count_errors(['ERROR disk', 'INFO ERROR text', 'WARNING x']) == 1

def test_file_adapter_reads_utf8(tmp_path):
    path = tmp_path / 'events.log'
    path.write_text('ERROR árvíz\nINFO ok\nERROR tárhely\n', encoding='utf-8')
    assert count_file(path) == 2
```

Az első teszt a felismerési szabályt, a második a fájlolvasás bekötését és a sortörések feldolgozását ellenőrzi. Nem szükséges a másodikban a read_text hívásszámát vizsgálni: a lényeg a kapott eredmény. Az integrációs példa szándékosan helyi és gyors; az integráció nem jelent automatikusan hálózatot.

## Senior döntések és tipikus hibák

Egy regresszióhoz a legszűkebb olyan tesztet add hozzá, amely ténylegesen észleli. A túl magas szint lassú hibakeresést okoz; a túl alacsony szint kihagyhatja a hibás bekötést. A puszta „nem dobott hibát” ritkán elég: ellenőrizd az eredményt és a fontos mellékhatásokat is.

A tesztkód olvashatósága diagnosztikai eszköz. Az Arrange–Act–Assert sorrendben legyen egyértelmű a kiinduló állapot, a vizsgált művelet és az elvárt viselkedés. Egy teszt több kapcsolódó assertiont is tartalmazhat; az „egy teszt, egy assert” nem általános cél.

## Interview questions

**How do you choose the right test level?**

Válaszvázlat: A hibakockázatból indulok: melyik valódi határt kell bevonni, és mi a legolcsóbb teszt, amely észleli a regressziót?

**What does a passing unit test not prove?**

Válaszvázlat: Nem igazolja a függőségek valódi viselkedését, a konfigurációt és a teljes rendszer bekötését.

## Önellenőrzés

- [ ] Megmondom, mi marad kívül az adott teszt határán.
- [ ] Eredményt és fontos mellékhatást ellenőrzök.

## Kapcsolódó gyakorlat

[Q01 – önálló feladat](../../../exercises/senior-python/07-testing-and-quality.md#q01)

## Forrás és továbbolvasás

[pytest: anatomy of a test](https://docs.pytest.org/en/stable/explanation/anatomy.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
