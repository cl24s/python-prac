# Objektumtervezési review és biztonságos refaktorálás

## Mit kell tudnod?

- Felelősségek és változtatási okok alapján elemezni egy osztályt.
- Refaktorálás előtt rögzíteni a megőrzendő viselkedést.
- Külön kezelni az interfész szignatúráját és a viselkedési szerződést.

## Kiinduló probléma

Képzelj el egy `JobManager` osztályt, amely JSON-t parse-ol, SQL-kapcsolatot nyit, jogosultságot ellenőriz, időt kér, állapotot módosít és emailt küld. A név nem árulja el a határokat, a tesztnek minden infrastruktúrát indítania kell. A gond nem önmagában az osztály hossza, hanem az eltérő felelősségek és rejtett függőségek összekapcsolása.

Refaktorálás előtt írd le a szerződést: mely input érvényes, milyen hiba keletkezik, mi történik duplikált beküldésnél, mikor jön létre tartós adat, mi marad hátra külső szolgáltatás hibájánál. Ezek nélkül a „szebb” kód észrevétlenül mást csinálhat.

## Kódpélda: adapter viselkedési szerződése

```python
from typing import Protocol

class Names(Protocol):
    def add(self, name: str) -> None: ...
    def contains(self, name: str) -> bool: ...

class MemoryNames:
    def __init__(self) -> None:
        self._names: set[str] = set()

    def add(self, name: str) -> None:
        if name in self._names:
            raise ValueError('duplicate name')
        self._names.add(name)

    def contains(self, name: str) -> bool:
        return name in self._names

def verify_contract(store: Names) -> None:
    assert not store.contains('api')
    store.add('api')
    assert store.contains('api')
    try:
        store.add('api')
    except ValueError:
        pass
    else:
        raise AssertionError('Duplicate must be rejected')

verify_contract(MemoryNames())
```

A Protocol a műveletek típusát írja le, a contract check a duplikációs szemantikát is rögzíti. Egy SQL-adapter ugyanezt a szerződést friss, izolált tesztadatbázissal teljesítheti; azt ez a memóriabeli példa nem bizonyítja.

## Refaktorálási sorrend

1. Rögzítsd a fontos megfigyelhető viselkedést célzott tesztekkel.
2. Tedd explicitté az időforrást, tárolót és külső kliensfüggőséget.
3. Emeld ki a tiszta átalakításokat függvénybe.
4. Az üzleti állapotátmeneteket helyezd a szabályt birtokló objektumhoz.
5. Az összeszerelést hagyd az alkalmazás belépési pontján.
6. Ugyanazokat az elfogadási feltételeket ellenőrizd minden lépés után.

## Tipikus hibák és senior szempontok

- **Hiba:** nagy átírás közben a hibakód és a retry viselkedése is változik. **Javítás:** a refaktorálást és a funkcióváltozást külön lépésekre bontsd.
- **Hiba:** fake tároló csendben felülír, az éles unique constraint hibázik. **Javítás:** közös viselkedési szerződés és megfelelő integrációs teszt.
- Egy „előbb ellenőrzöm, aztán beszúrom” megoldás nem garantál konkurens egyediséget. Tartós tárolónál atomikus adatbázis-korlát is szükséges lehet.
- Ne minden belső metódust mockolj. A teszt a jelentős eredményt és mellékhatást vizsgálja, különben a refaktorálás pusztán a belső elrendezés miatt töri el.

## Interview questions

**What would you test before splitting a large service class?**

Válaszvázlat: sikeres és hibás input, duplikáció, mellékhatás, részleges hiba; a megőrzendő külső szerződés számít.

**Why can a fake pass unit tests while the real adapter fails?**

Válaszvázlat: eltérő konzisztencia, hibakezelés, tranzakció vagy identity-szemantika; contract és integrációs teszt együtt kell.

## Önellenőrzés

- [ ] Meg tudom nevezni, mit nem szabad megváltoztatni refaktoráláskor.
- [ ] Külön látom a típus- és viselkedési szerződést.
- [ ] A fake és a valódi adapter eltérési kockázatát is vizsgálom.


## Kapcsolódó gyakorlat

[O06 – önálló feladat](../../../exercises/senior-python/03-oop.md#o06)

## Forrás és továbbolvasás

[unittest: test cases](https://docs.python.org/3/library/unittest.html#test-cases) · [Protocols](https://typing.python.org/en/latest/spec/protocol.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
