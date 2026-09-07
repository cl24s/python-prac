# Biztonságos továbbfejlesztés és fokozatos migráció

## Mit kell tudnod?

- Régi és új verzió együttélését megtervezni.
- A rollbacket elkülöníteni az adat-visszaállítástól.
- Kis, megfigyelhető és megszakítható lépésekben változtatni.

## Magyarázat

Élő rendszerben a régi és új kód egy ideig együtt futhat; a queue pedig régi üzeneteket őrizhet. A migráció ezért állapotok sorozata, nem egyetlen rename. Minden átmeneti állapotnak működőképesnek kell lennie, és tudni kell, melyik irányba lehet még visszalépni.

Expand–migrate–contract során először kompatibilis bővítést adunk, majd adatot és olvasási/írási útvonalat váltunk, végül eltávolítjuk a régit. A contract csak akkor jöhet, amikor a régi fogyasztók, késleltetett üzenetek és rollbackigény már nem használják a régi formátumot.

## Kódpélda: két üzenetverzió olvasása

```python
import pytest

def read_job_id(payload):
    version = payload.get('version', 1)
    if type(version) is not int:
        raise ValueError('invalid version')
    if version == 1:
        job_id = payload.get('id')
    elif version == 2:
        job_id = payload.get('job_id')
    else:
        raise ValueError('unsupported version')
    if not isinstance(job_id, str) or not job_id:
        raise ValueError('invalid job id')
    return job_id

@pytest.mark.parametrize('payload', [
    {'id': 'j1'}, {'version': 1, 'id': 'j1'}, {'version': 2, 'job_id': 'j1'},
])
def test_supported_versions(payload):
    assert read_job_id(payload) == 'j1'

@pytest.mark.parametrize('payload', [
    {'version': 3, 'job_id': 'j1'}, {'version': 2, 'id': 'j1'},
    {'version': True, 'id': 'j1'}, {'version': 2, 'job_id': ''},
])
def test_bad_messages_are_explicitly_rejected(payload):
    with pytest.raises(ValueError):
        read_job_id(payload)
```

A bemenet mappingként már parse-olt JSON-objektum; nem teljes input-validátor. A teszt az olvasási kompatibilitást bizonyítja, nem a rolloutot. Az új fogyasztó előbb megy ki, miközben a producer még v1-et küld. Csak akkor vált v2 írásra, amikor minden szükséges fogyasztó érti. V2 termelés után régi, csak v1-et értő fogyasztóra rollback nem biztonságos.

## Adatbázis-migrációs eset

Egy mező átnevezésekor először új nullable oszlop jelenik meg. Az új kód az átmeneti szerződés szerint ír; a régi olvasás még működik. A backfill kis, újraindítható batchben halad, checkpointtal és ellenőrzéssel. Közben a live írásokat is kezelni kell, különben az egyszer végigfutott backfill után új eltérések keletkeznek.

A read switch előtt számold az eltéréseket, ellenőrizd a hiányzó adatot és a terhelési hatást. A NOT NULL és régi oszlop törlése külön contract lépés. A lock- és indexépítési hatás adatbázis- és verziófüggő: éles méretre reprezentatív vizsgálat kell.

## Visszaállás és felügyelet

Kódrollback nem törli a már elküldött emailt és nem állítja vissza automatikusan az átírt adatot. Előre rögzítsd a visszafordíthatatlan pontot és a helyreállítási utat. A mentésből visszaállításnak adatvesztési és időkorlátja van; a létező backup nem bizonyított restore.

Canarynál előre válassz hibaarány-, latency- és üzleti konzisztenciajeleket, valamint megállítási feltételt. A feature flag kontrollált útváltást adhat, de az adatkompatibilitást nem teremti meg. A flagnek tulajdonos és eltávolítási határidő kell.

Branch by abstraction esetén előbb stabil határ mögé rendezed a régi megvalósítást, majd azon át vezeted be az újat. Shadow read hasznos összehasonlításra; mellékhatásos műveletet nem szabad vakon kétszer végrehajtani.

## Interview questions

**Why is rolling back code not the same as rolling back a migration?**

Válaszvázlat: Az adatok és külső mellékhatások már változhattak; a régi kód lehet, hogy nem érti az új sémát vagy üzenetet.

**When can you remove support for an old message version?**

Válaszvázlat: Ha nincs régi producer, a backlog/replay út is kezelve van, és a vállalt rollbackablak sem igényli.

## Önellenőrzés

- [ ] Minden köztes verzióállapothoz van kompatibilitási tervem.
- [ ] A backfill folytatható és ellenőrizhető.

## Kapcsolódó gyakorlat

[SD05 – önálló feladat](../../../exercises/senior-python/14-design-and-collaboration.md#sd05)

## Forrás és továbbolvasás

[Martin Fowler: Branch By Abstraction](https://martinfowler.com/bliki/BranchByAbstraction.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
