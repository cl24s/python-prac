# SQL: join, aggregáció, NULL és adatinvariánsok

## Mit kell tudnod?

- JOIN és aggregáció eredményét kézzel is megmagyarázni.
- Paraméterezett SQL-t használni értékinterpoláció helyett.
- A kritikus invariánst adatbázis-constrainttel is védeni.

## Magyarázat

A relációs modellben az idegen kulcs kapcsolatot, a unique egyediséget, a not null kötelező értéket, a check értékfeltételt véd. Az API validációja gyors és érthető hibát adhat, de más író és konkurens kérés ellen a tárolóban is szükség van a szabályra.

INNER JOIN csak illeszkedő sorokat ad; LEFT JOIN megtartja a bal oldali sort akkor is, ha nincs párja. Ilyenkor a jobb oldali mezők NULL értékűek. COUNT(*) az eredménysorokat, COUNT(job.id) a nem NULL jobazonosítókat számolja. Ez fontos, ha a job nélküli felhasználónak nullát szeretnél.

## Kódpélda: nulla jobbal rendelkező tulajdonos is megmarad

Minden Python-blokk önálló pytest-modul; ez standard library sqlite3-at használ.

```python
from contextlib import closing
import sqlite3

def test_left_join_and_parameter_binding():
    with closing(sqlite3.connect(':memory:')) as db:
        db.execute('PRAGMA foreign_keys = ON')
        db.executescript("""
            CREATE TABLE owners(id INTEGER PRIMARY KEY, name TEXT NOT NULL UNIQUE);
            CREATE TABLE jobs(id INTEGER PRIMARY KEY, owner_id INTEGER NOT NULL
                REFERENCES owners(id), status TEXT NOT NULL CHECK(status IN ('queued', 'done')));
        """)
        db.executemany('INSERT INTO owners VALUES (?, ?)', [(1, 'alice'), (2, 'bob')])
        db.executemany('INSERT INTO jobs VALUES (?, ?, ?)', [(1, 1, 'queued'), (2, 1, 'done')])
        rows = db.execute("""
            SELECT o.name, COUNT(j.id)
            FROM owners AS o LEFT JOIN jobs AS j
              ON j.owner_id = o.id AND j.status = ?
            GROUP BY o.id, o.name ORDER BY o.id
        """, ('queued',)).fetchall()
        assert rows == [('alice', 1), ('bob', 0)]
        hostile = "alice' OR 1=1 --"
        assert db.execute('SELECT id FROM owners WHERE name = ?', (hostile,)).fetchall() == []
```

Ha a j.status feltételt a WHERE-be tennéd, a NULL jobú owner kiesne. A szűrés helye megváltoztatja a LEFT JOIN jelentését. Az egy-a-többhöz join megsokszorozza a sorokat; újabb kapcsolat hozzáadásakor az összegzés véletlenül többször számolhat ugyanazzal a jobbal.

## Senior döntések és tipikus hibák

Értékeket bind paraméterrel adj át. Tábla- vagy oszlopnév, rendezési irány általában nem bindolható értékként: dinamikus rendezéshez ellenőrzött név→SQL mapping kell, nem tetszőleges kliensszöveg.

NULL esetén az = NULL helyett IS NULL vizsgálat szükséges. A COUNT és SUM üres eredményének viselkedését is tudd: a nulla és a NULL nem automatikusan ugyanaz az üzleti jelentés. COALESCE csak tudatos alapértékkel kerüljön be.

A SQLite foreign key ellenőrzését a példa explicit bekapcsolja a kapcsolaton. A különböző adatbázisok típusszigora és NULL/unique részletei eltérhetnek; a tesztmotor viselkedését ne általánosítsd minden szerverre.

## Interview questions

**Why can a WHERE clause change a LEFT JOIN result?**

Válaszvázlat: A jobb oldali NULL sorokat a feltétel kiszűrheti; az ON a kapcsolódó sorok körét, a WHERE a már összeállt eredményt szűri.

**Does parameter binding protect dynamic table names?**

Válaszvázlat: Nem általánosan; az értékek bindolhatók, az azonosítókhoz ellenőrzött allowlist/mapping kell.

## Önellenőrzés

- [ ] A job nélküli owner nullás darabszámmal megmarad.
- [ ] A join miatt megsokszorozott sorokra figyelek.

## Kapcsolódó gyakorlat

[DB01 – önálló feladat](../../../exercises/senior-python/10-databases.md#db01)

## Forrás és továbbolvasás

[SQLite SELECT semantics](https://www.sqlite.org/lang_select.html)

[Python sqlite3 placeholders](https://docs.python.org/3.12/library/sqlite3.html#how-to-use-placeholders-to-bind-values-in-sql-queries)

[PostgreSQL constraints](https://www.postgresql.org/docs/18/ddl-constraints.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
