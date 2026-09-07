# Indexek, lekérdezési tervek és mérés

## Mit kell tudnod?

- A lekérdezés szűréséhez és rendezéséhez illeszkedő indexet tervezni.
- Megkülönböztetni a terv becslését a tényleges futtatástól.
- Az index írási és tárhely-költségét is számításba venni.

## Magyarázat

Az index nem általános gyorsító minden művelethez. A kulcssorrend, szelektivitás, rendezés és visszaadott adatmennyiség határozza meg a hasznát. Egy tenant_id + status szerint szűrt, id szerint rendezett joblista számára összetett index lehet indokolt; nem szükségképpen három külön index.

Az adatbázis optimalizálója becslésekből választ tervet. Kis táblán egy teljes scan teljesen jó döntés lehet. Az „indexet használ” nem önmagában sikerfeltétel: a latency, beolvasott sorok, I/O és írási költség a fontos.

## Kódpélda: célzott SQLite-index és olvasható terv

```python
from contextlib import closing
import sqlite3

def test_indexed_filter_and_order():
    with closing(sqlite3.connect(':memory:')) as db:
        db.execute('CREATE TABLE jobs(id INTEGER PRIMARY KEY, tenant_id INTEGER, status TEXT)')
        db.executemany('INSERT INTO jobs VALUES (?, ?, ?)',
                       [(i, i % 10, 'queued' if i % 3 else 'done') for i in range(1, 1001)])
        db.execute('CREATE INDEX jobs_lookup ON jobs(tenant_id, status, id)')
        query = 'SELECT id FROM jobs WHERE tenant_id = ? AND status = ? ORDER BY id LIMIT 3'
        assert db.execute(query, (2, 'queued')).fetchall() == [(2,), (22,), (32,)]
        plan = db.execute('EXPLAIN QUERY PLAN ' + query, (2, 'queued')).fetchall()
        detail = ' '.join(row[3] for row in plan)
        assert 'jobs_lookup' in detail
```

Ez az ellenőrzés a helyi SQLite-verzió konkrét tanpéldájára vonatkozik. Nem időalapú benchmark, és nem állítjuk, hogy a PostgreSQL ugyanilyen szöveges tervet ad. A tervformátumot ne másold általános API-szerződésbe.

## Tervolvasás PostgreSQL-ben

Az EXPLAIN becsült tervet ad; az EXPLAIN ANALYZE végre is hajtja az utasítást. Nézd az estimated vs actual sorokat, loops értéket, scan/join típusokat és a sort műveletet. A BUFFERS segít az I/O megértésében. Egy sokszor futó kis belső scan összességében drága lehet.

Író SQL-t ne futtass ANALYZE-zal gondolkodás nélkül éles rendszeren: végrehajtja a mellékhatást. Tesztadat és kontrollált környezet kell. A tranzakciós rollback sem univerzális visszavonás minden külső mellékhatásra.

## Senior döntések és tipikus hibák

Az index karbantartása lassíthatja az INSERT/UPDATE/DELETE műveletet és tárhelyet kér. Több, majdnem azonos index felhalmozása helyett a fontos lekérdezésekből indulj. Függvénybe csomagolt szűrés, eltérő collation vagy típuskonverzió ronthatja a hagyományos index használhatóságát.

A részleges index egy gyakori részhalmazra, a covering index a lekérdezés kiszolgálásához szükséges mezőkre fókuszálhat; támogatásuk és pontos működésük motorfüggő. Nagy offset helyett a 8. témakör keyset lapozása segíthet, megfelelő indexszel együtt.

## Interview questions

**Why might a sequential scan be the right plan?**

Válaszvázlat: Kis tábla vagy a sorok nagy részének visszaadása esetén az index plusz véletlen elérése drágább lehet.

**What is dangerous about EXPLAIN ANALYZE on writes?**

Válaszvázlat: Ténylegesen végrehajtja az utasítást; nem pusztán statikus becslés.

## Önellenőrzés

- [ ] A tervet a reprezentatív adat és a tényleges költség mellett értékelem.
- [ ] Az index írási árát is megnevezem.

## Kapcsolódó gyakorlat

[DB02 – önálló feladat](../../../exercises/senior-python/10-databases.md#db02)

## Forrás és továbbolvasás

[SQLite query plans](https://www.sqlite.org/eqp.html)

[PostgreSQL EXPLAIN](https://www.postgresql.org/docs/18/using-explain.html)

[Multicolumn indexes](https://www.postgresql.org/docs/18/indexes-multicolumn.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
