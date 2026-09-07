# Refaktorálás és regresszióvédelem

## Mit kell tudnod?

- Viselkedésmegőrző változtatást elkülöníteni a hibajavítástól.
- Karakterizációs teszttel rögzíteni az ismert működést.
- Független elvárt értéket és hasznos invariánst választani.

## Magyarázat

Refaktoráláskor a belső szerkezet változik, a megfigyelhető szerződés megmarad. Ez nemcsak a visszatérési érték: a kivételtípus, a rendezés, a mellékhatások és esetenként a lusta kiértékelés is része lehet. Egy generátor listává alakítása például változtathat a memóriaigényen és azon, mikor dobódik hiba.

Legacy kódnál előbb térképezd fel a használt viselkedést. A karakterizációs teszt rögzíti, mit csinál most; ettől még az nem feltétlenül helyes üzleti szabály. A tudatos hibajavítás kapjon új elvárást és külön magyarázatot. Kis lépésekben könnyebb megmondani, melyik módosítás hozott regressziót.

## Kódpélda: régi és új implementáció ismert eredménnyel

```python
import pytest

def old_unique(values):
    result = []
    for value in values:
        if value not in result:
            result.append(value)
    return result

def new_unique(values):
    return list(dict.fromkeys(values))

@pytest.mark.parametrize('implementation', [old_unique, new_unique])
@pytest.mark.parametrize('values, expected', [([], []), ([2, 1, 2], [2, 1]),
                                              ([3, 3, 3], [3])])
def test_unique_integer_contract(implementation, values, expected):
    before = values.copy()
    assert implementation(values) == expected
    assert values == before

def test_appending_existing_integer_keeps_result():
    values = [3, 1, 3, 2]
    assert new_unique(values + [1]) == new_unique(values)
```

A szerződés itt kifejezetten egész számok listája. A régi verzió nem hash-elhető értékekkel is működhet, az új nem; tetszőleges objektumokra ezért nem volna automatikusan helyes refaktorálás. Az elvárt [2, 1] rögzíti a sorrendet, nem csak a két implementáció egyezését nézzük. Két hibás algoritmus is adhat azonos rossz eredményt.

## Tulajdonságok és lefedettség

Az invariáns több bemenetre értelmes állítás: nincs duplikáció, az első előfordulás sorrendje megmarad, a bemenet nem módosul. A property-based tesztelés sok generált bemeneten keres ellenpéldát, és gyakran egyszerűsíti a bukó esetet. A fenti rögzített példa még nem generatív teszt; nem állítunk Hypothesis-ellenőrzést.

A sorlefedettség azt mutatja, hol futott kód; az áglefedettség több döntési útvonalat láthatóvá tesz. Egyik sem bizonyítja, hogy jó assertiont írtál. Gondolatban módosítsd a feltételt vagy az eredményt: melyik teszt bukna? Ez célzottan segít gyenge tesztet találni, teljes mutation testing infrastruktúra nélkül is.

## Interview questions

**How do you refactor unfamiliar code safely?**

Válaszvázlat: Feltárom a megfigyelhető szerződést, karakterizációs tesztet írok a fontos utakhoz, majd kis lépésekben változtatok.

**Why can differential tests miss a bug?**

Válaszvázlat: A két implementáció közös hibát tartalmazhat; független példák és invariánsok is kellenek.

## Önellenőrzés

- [ ] Megvizsgálom a bemeneti tartomány változását is.
- [ ] Elkülönítem a viselkedésrögzítést az üzleti helyességtől.

## Kapcsolódó gyakorlat

[Q05 – önálló feladat](../../../exercises/senior-python/07-testing-and-quality.md#q05)

## Forrás és továbbolvasás

[Python dictionaries](https://docs.python.org/3.12/library/stdtypes.html#dict)

[Coverage measurement](https://coverage.readthedocs.io/en/latest/branch.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
