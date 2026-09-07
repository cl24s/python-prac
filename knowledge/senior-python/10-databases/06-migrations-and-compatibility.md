# Sémamigráció, backfill és kompatibilis telepítés

## Mit kell tudnod?

- Modellváltozást elkülöníteni a migráció végrehajtásától.
- Expand–backfill–contract sorrendben tervezni.
- Régi és új alkalmazásverzió együttfutására felkészülni.

## Magyarázat

Az ORM-modell átírása nem változtatja meg automatikusan az éles sémát. A create_all az induló táblák létrehozására alkalmas; nem a meglévő adatok és a kompatibilis rolling deploy migrációs terve.

Expand: új mező/tábla hozzáadása úgy, hogy a régi alkalmazás még működjön. Backfill: a régi adatok fokozatos pótlása, újraindítható módon. Contract: a régi mező vagy támogatás eltávolítása csak akkor, amikor már nincs régi olvasó/író és a szükséges adat elkészült.

## Kódpélda: nullable mező bővítése és ismételhető backfill

A példa a bővítést és backfillt mutatja, nem teljes Alembic-migráció és nem rolling deployteszt.

```python
from contextlib import closing
import sqlite3

def backfill(db):
    db.execute("UPDATE jobs SET priority = 'normal' WHERE priority IS NULL")

def test_expand_and_repeatable_backfill():
    with closing(sqlite3.connect(':memory:', isolation_level=None)) as db:
        db.execute('CREATE TABLE jobs(id INTEGER PRIMARY KEY, name TEXT NOT NULL)')
        db.execute("INSERT INTO jobs VALUES (1, 'legacy')")
        db.execute('ALTER TABLE jobs ADD COLUMN priority TEXT')
        assert db.execute('SELECT id, name FROM jobs').fetchall() == [(1, 'legacy')]
        backfill(db)
        # An old writer is still active after the first backfill.
        db.execute("INSERT INTO jobs(id, name) VALUES (2, 'late legacy')")
        assert db.execute('SELECT priority FROM jobs WHERE id = 2').fetchone() == (None,)
        backfill(db)
        backfill(db)
        assert db.execute('SELECT priority FROM jobs ORDER BY id').fetchall() == [('normal',), ('normal',)]
```

Az egyszer lefutott backfill nem elég, ha a régi író továbbra is NULL-t hoz létre. Előbb kompatibilis írást vagy megfelelő adatbázis-alapértéket kell biztosítani; utána ellenőrizhető a NOT NULL feltétel. Az alkalmazáskódbeli alapérték nem azonos a szerveroldali defaulttal.

## Alembic és review

Az autogenerate sémakülönbségekből migrációjelöltet készít. A mezőátnevezést például eldobás és hozzáadásként is értelmezheti; a generált fájlt review-zni kell. Nem találja ki az adatátalakítás üzleti szabályát vagy a többverziós deployment sorrendjét.

A migrációs tervben legyen előfeltétel, várt lock/időigény, backfill batch-méret, haladásjelző és ellenőrző query. Nagy táblán a teljes egyutas UPDATE hosszú lockot és sok naplót generálhat. PostgreSQL concurrent indexépítésének külön tranzakciós szabályai vannak; motorfüggő eljárást ne másolj SQLite-ról.

## Senior döntések és tipikus hibák

Egy destructive migráció visszagörgetése nem hozza vissza automatikusan az elveszett adatot. A rollback terv lehet alkalmazásvisszaállítás kompatibilis sémán, forward fix vagy backup-visszaállítás; ezek különböző költségű döntések.

Ne indítsd ugyanazt a migrációt minden worker startupjából. A release folyamat egy koordinált lépése végezze. A contract fázis csak mért és ellenőrzött feltételek után jöjjön, ne ugyanabban a pillanatban, amikor az új kód először elindul.

## Interview questions

**Why is a generated migration not deployment-ready?**

Válaszvázlat: Nem ismeri a régi alkalmazások együttfutását, az adatátalakítás szabályát és az éles lock/időigényt.

**Why can a completed backfill become incomplete again?**

Válaszvázlat: A még futó régi író újabb hiányos rekordot hozhat létre; előbb az írási utat kell kompatibilissé tenni.

## Önellenőrzés

- [ ] A régi író késői beszúrására is gondolok.
- [ ] A destructive rollback adatvesztését nem hagyom figyelmen kívül.

## Kapcsolódó gyakorlat

[DB06 – önálló feladat](../../../exercises/senior-python/10-databases.md#db06)

## Forrás és továbbolvasás

[Alembic autogenerate](https://alembic.sqlalchemy.org/en/latest/autogenerate.html)

[PostgreSQL ALTER TABLE](https://www.postgresql.org/docs/18/sql-altertable.html)

[PostgreSQL CREATE INDEX](https://www.postgresql.org/docs/18/sql-createindex.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
