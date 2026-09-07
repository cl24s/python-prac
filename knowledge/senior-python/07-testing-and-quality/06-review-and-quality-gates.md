# Code review és indokolt minőségi ellenőrzések

## Mit kell tudnod?

- A review-t helyesség, kockázat és karbantarthatóság szerint priorizálni.
- Megkülönböztetni a formázás, lint, típusellenőrzés és teszt szerepét.
- A hibát láthatóvá tevő, reprodukálható ellenőrzést választani.

## Magyarázat

Egy senior review először a változtatás célját és szerződését vizsgálja. Azután a hibás utak, erőforrások, adatvesztés és kompatibilitás következnek. A stílus jelentős részét automatizálhatod; az üzleti döntést nem fogja helyetted meghozni a formatter.

| Ellenőrzés | Példa arra, amit észrevehet | Amit nem helyettesít |
| --- | --- | --- |
| Formatter | Következetlen tördelés | Jó absztrakció és elnevezés |
| Linter | Használatlan import, egyes veszélyes minták | Üzleti helyesség |
| Statikus típusellenőrzés | Nem megfelelő interfész vagy visszatérési típus | Külső input runtime validációja |
| Teszt | Konkrét viselkedési regresszió | Nem vizsgált forgatókönyvek |
| Emberi review | Hibás határ, túltervezés, hiányzó követelmény | Futási bizonyíték |

## Review-eset: a hiba is szerződés

A következő blokk két tesztet tartalmaz, és `test_example.py` fájlként futtatható pytesttel.

```python
import pytest

def load_port(environ):
    raw = environ.get('APP_PORT', '8080')
    port = int(raw)
    if not 1 <= port <= 65535:
        raise ValueError('invalid port')
    return port

def test_default_port(monkeypatch):
    import os
    monkeypatch.delenv('APP_PORT', raising=False)
    assert load_port(os.environ) == 8080

def test_invalid_config_does_not_fall_back():
    with pytest.raises(ValueError):
        load_port({'APP_PORT': 'eighty'})
```

A review érdemi kérdése: hibás konfigurációnál induljon-e az alkalmazás alapértelmezéssel? Itt a dokumentált döntés az elutasítás. Egy széles except, amely 8080-at ad vissza, elfedné az üzemeltetési hibát. A kód rövidsége önmagában nem teszi ezt jó változtatássá.

## Reprodukálható munkafolyamat

1. Olvasd el a problémát és az elfogadási feltételeket.
2. Ellenőrizd a diffet a kritikus sikeres és hibás út mentén.
3. Futtasd a célzott ellenőrzést; szélesíts, ha közös modul vagy szerződés változott.
4. Rögzítsd, mi futott, milyen verzióval és mi maradt kívül.
5. Review-megjegyzésben írd le a következményt és a kívánt viselkedést.

Példa hasznos megjegyzésre: „Ez az except a hibás APP_PORT értéket is alapértelmezetté alakítja. Így a szolgáltatás rossz porton indulhat. Maradjon explicit konfigurációs hiba, és legyen rá regressziós teszt.” Ez konkrétabb, mint a „nem clean” vagy a „használj SOLID-ot”.

## Tipikus hibák és kompromisszumok

A CI-ben kihagyott vagy xfailként jelölt teszt nem sikeres bizonyíték. Ismert hibánál legyen indok, felelős és visszatérési pont; váratlan javulás észlelésére strict xfail használható. Ne állítsd át tartósan a küszöböt csak azért, hogy zöld legyen a build.

Új eszközt konkrét kockázatra vezess be. Egy formatter hasznos lehet, de ebben a tananyag-bővítésben nem alakítjuk át a repó teljes futtatókörnyezetét. A tényleges projekt indulásakor verziózott függőségekkel és közös helyi/CI parancsokkal rögzítjük azt.

## Interview questions

**What should a senior code review prioritize?**

Válaszvázlat: A követelményt és a megfigyelhető viselkedést, majd a hibás utakat, adat- és erőforrás-kezelést, kompatibilitást és karbantarthatóságot.

**Can static typing replace runtime validation?**

Válaszvázlat: Nem: a típusellenőrző a kód modelljét vizsgálja, a külső adatok értékét külön kell ellenőrizni.

## Önellenőrzés

- [ ] Konkrét következménnyel indokolom a review-kérést.
- [ ] Nem nevezek kihagyott ellenőrzést sikeresnek.

## Kapcsolódó gyakorlat

[Q06 – önálló feladat](../../../exercises/senior-python/07-testing-and-quality.md#q06)

## Forrás és továbbolvasás

[pytest monkeypatch](https://docs.pytest.org/en/stable/how-to/monkeypatch.html)

[pytest skip and xfail](https://docs.pytest.org/en/stable/how-to/skipping.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
