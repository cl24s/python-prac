# Idempotens fogyasztó és tartós deduplikáció

## Mit kell tudnod?

- Deduplikációs kulcsot üzleti hatókörrel tervezni.
- Az eredményt és a feldolgozási jelölőt atomikusan rögzíteni.
- A külső mellékhatásokat külön kezelni.

## Magyarázat

At-least-once kézbesítésnél ugyanaz az esemény többször is megérkezhet. A fogyasztó célja lehet, hogy ugyanazt a logikai eseményt csak egyszer alkalmazza a saját tartós állapotára. Ez nem feltétlenül azonos az egész rendszer „exactly once” működésével.

A deduplikációs kulcs tartalmazza a megfelelő hatókört: például tenant és event ID. Tárold a tartalom ellenőrzéséhez szükséges adatot is; ugyanaz az ID más payloadhoz ne legyen csendesen sikeres duplikáció. A jelölő és az üzleti változás egy tranzakcióba kerüljön.

## Kódpélda: számlálóváltozás és marker együtt

```python
from contextlib import closing
import sqlite3
import pytest

class EventConflict(Exception):
    pass

def apply_event(db, tenant, event_id, delta, fail=False):
    db.execute('BEGIN IMMEDIATE')
    try:
        old = db.execute('SELECT delta FROM processed WHERE tenant = ? AND event_id = ?',
                         (tenant, event_id)).fetchone()
        if old is not None:
            if old[0] != delta:
                raise EventConflict('event ID reused with different content')
            applied = False
        else:
            changed = db.execute('UPDATE counters SET value = value + ? WHERE tenant = ?',
                                 (delta, tenant)).rowcount
            if changed != 1:
                raise ValueError('unknown tenant')
            if fail:
                raise RuntimeError('injected failure before marker')
            db.execute('INSERT INTO processed VALUES (?, ?, ?)', (tenant, event_id, delta))
            applied = True
        db.execute('COMMIT')
        return applied
    except BaseException:
        if db.in_transaction:
            db.execute('ROLLBACK')
        raise

def test_persistent_deduplication(tmp_path):
    path = tmp_path / 'consumer.db'
    with closing(sqlite3.connect(path, isolation_level=None)) as db:
        db.executescript("""
            CREATE TABLE counters(tenant TEXT PRIMARY KEY, value INTEGER NOT NULL);
            CREATE TABLE processed(tenant TEXT, event_id TEXT, delta INTEGER NOT NULL,
                                   PRIMARY KEY(tenant, event_id));
            INSERT INTO counters VALUES ('a', 0);
        """)
        with pytest.raises(RuntimeError):
            apply_event(db, 'a', 'evt-1', 5, fail=True)
        assert db.execute('SELECT value FROM counters').fetchone() == (0,)
        assert apply_event(db, 'a', 'evt-1', 5)
    with closing(sqlite3.connect(path, isolation_level=None)) as db:
        assert not apply_event(db, 'a', 'evt-1', 5)
        with pytest.raises(EventConflict):
            apply_event(db, 'a', 'evt-1', 9)
        assert db.execute('SELECT value FROM counters').fetchone() == (5,)
```

A teszt bezárt és újranyitott SQLite-fájlon is ellenőrzi a markert. Az injektált exception rollbacket vizsgál, nem operációsrendszer-összeomlást vagy áramkimaradást. A BEGIN IMMEDIATE itt SQLite-írótranzakciót foglal; nem PostgreSQL-minta minden lockolási részletre.

## Senior döntések és tipikus hibák

A marker előzetes, külön commitja után elmaradhat az üzleti hatás; a hatás utáni külön marker előtt újrakézbesítés duplikálhat. A tranzakció oldja meg a két helyi változás együttkezelését, nem egy processzmemóriás set.

Az e-mail, HTTP-fizetés vagy más távoli hívás nem válik atomikussá attól, hogy helyben DB-tranzakciót nyitsz köré. Külső szolgáltatásnál idempotency key, visszakereshető művelet vagy kompenzáció szükséges. A marker megőrzési ideje fedje le a lehetséges replay időszakát; a korán törölt marker újraengedheti a régi eseményt.

A példában a payload egész delta, ezért közvetlenül tároljuk. Összetett eseménynél rögzített kanonizálás vagy verziózott tartalomazonosítás kell. Az event ID nem helyettesíti a tenant jogosultság ellenőrzését.

## Interview questions

**Why must the deduplication marker and business update share a transaction?**

Válaszvázlat: Bármelyik külön commitja után létrejöhet hiányzó vagy duplikált hatás egy crash/replay során.

**Does this transaction make sending an email exactly once?**

Válaszvázlat: Nem; a távoli mellékhatás kívül van a tranzakción. Külön idempotencia vagy más üzleti protokoll kell.

## Önellenőrzés

- [ ] A duplikációt új DB-kapcsolatból is felismerem.
- [ ] Azonos ID eltérő tartalommal konfliktus.

## Kapcsolódó gyakorlat

[R03 – önálló feladat](../../../exercises/senior-python/11-reliable-services.md#r03)

## Forrás és továbbolvasás

[SQLite transactions](https://www.sqlite.org/lang_transaction.html)

[RabbitMQ reliability and redelivery](https://www.rabbitmq.com/docs/reliability)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
