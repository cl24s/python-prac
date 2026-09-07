# I/O, pool-várakozás és adatbázis-költség

## Mit kell tudnod?

- Megkülönböztetni a távoli végrehajtást és a kapcsolatért várakozást.
- Hívásszámot és batch-elést mérni, nem csak egy query sebességét.
- A teljes kérés költségét felbontani.

## Magyarázat

Egy 900 ms-os adatbázis-hívás lehet 800 ms pool-várakozás és 100 ms SQL, vagy fordítva. Más javítás kell rájuk. Ha csak a teljes adapterhívást méred, nincs elég információ a döntéshez.

Külön vizsgáld a kapcsolat megszerzését, DNS/TLS felépítést, request küldést, első byte-ot, body olvasást és deszerializációt, ahol az eszköz ezt támogatja. A trace span a hívási kapcsolatot mutatja; a metrika az eloszlást és trendet. Egyik sem helyettesíti a korrekt időmérési határt.

## Kódpélda: ugyanaz az adat négy SELECT helyett eggyel

```python
from contextlib import closing
import sqlite3

def one_by_one(db, ids):
    return [db.execute('SELECT name FROM jobs WHERE id = ?', (job_id,)).fetchone()[0]
            for job_id in ids]

def batched(db, ids):
    if not ids:
        return []
    placeholders = ','.join('?' for _ in ids)
    names = dict(db.execute(f'SELECT id, name FROM jobs WHERE id IN ({placeholders})', ids))
    return [names[job_id] for job_id in ids]

def test_query_count_and_order():
    with closing(sqlite3.connect(':memory:')) as db:
        db.execute('CREATE TABLE jobs(id INTEGER PRIMARY KEY, name TEXT)')
        db.executemany('INSERT INTO jobs VALUES (?, ?)', [(i, f'job-{i}') for i in range(1, 5)])
        calls = []
        db.set_trace_callback(lambda sql: calls.append(sql) if sql.lstrip().upper().startswith('SELECT') else None)
        ids = [4, 1, 3, 1]
        expected = ['job-4', 'job-1', 'job-3', 'job-1']
        assert one_by_one(db, ids) == expected
        assert len(calls) == 4
        calls.clear()
        assert batched(db, ids) == expected
        assert len(calls) == 1
        assert batched(db, []) == []
```

A placeholder szövegét csak a darabszámból készítjük, az értékek továbbra is paraméterezettek. A kimeneti sorrendet és az ismétlődő ID-kat explicit visszaállítjuk; az IN lekérdezés önmagában nem ígéri a bemeneti sorrendet.

## Senior döntések és tipikus hibák

A két függvény ismert, létező ID-ket feltételez. A hiányzó rekord szerződését külön definiálni kell. Nagy ID-listán a paraméterlimit, queryméret és memória miatt korlátos batch szükséges. A kevesebb query nem automatikusan kisebb latency, ha egyetlen óriási query sokkal több adatot olvas.

A teszt a hívásszámot igazolja, nem hálózati gyorsulást. A valódi DB-tervhez a [10. témakör index- és tervolvasása](../10-databases/02-indexes-and-query-plans.md) tartozik. ORM-nél a [betöltési stratégiát](../10-databases/05-orm-and-n-plus-one.md) is ellenőrizd.

Poolméret növelése csak akkor segít, ha a DB-nek van kapacitása; különben az összes query lassulhat. Ne foglalj kapcsolatot CPU-feldolgozás vagy külső API-várakozás alatt, ha nem szükséges tranzakciós garanciához. Először a kapcsolat foglalási idejét és a hívások számát nézd.

## Interview questions

**Why might increasing the connection pool make latency worse?**

Válaszvázlat: Több egyidejű query túlterhelheti a DB-t; a helyi várakozás távoli versengéssé alakul.

**What can batching accidentally change?**

Válaszvázlat: A sorrendet, duplikációkat, hiányzó elemek viselkedését és memóriaigényt; ezeket külön védeni kell.

## Önellenőrzés

- [ ] A pool és a query idejét külön szeretném látni.
- [ ] Batch után is megőrzöm a szerződés szerinti sorrendet.

## Kapcsolódó gyakorlat

[P04 – önálló feladat](../../../exercises/senior-python/12-performance-and-debugging.md#p04)

## Forrás és továbbolvasás

[SQLAlchemy pool behavior](https://docs.sqlalchemy.org/en/20/core/pooling.html)

[sqlite3 trace callback](https://docs.python.org/3.12/library/sqlite3.html#sqlite3.Connection.set_trace_callback)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
