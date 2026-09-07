# Izoláció, lockolás és elvesző módosítások

## Mit kell tudnod?

- Az olvasás–módosítás–írás race conditionjét felismerni.
- Optimista és pesszimista konkurenciakezelést összehasonlítani.
- Motoronként értelmezni az izolációs szinteket.

## Magyarázat

Két kérés elolvassa a v1 állapotot, mindkettő helyben módosít, majd feltétel nélkül ír. A második felülírhatja az első eredményét. A tranzakció léte önmagában nem mondja meg, hogy ezt észleli-e a rendszer; az izoláció és a konkrét SQL számít.

Optimista védelemnél a módosítás WHERE feltétele az ismert verziót is tartalmazza. Nulla módosított sor konfliktust jelent. Pesszimista megoldásnál a sor lockolása előzi meg a döntést; PostgreSQL-ben erre SELECT ... FOR UPDATE használható. A lock várakozást és deadlockot okozhat, ezért a sorrend és az időkeret fontos.

## Kódpélda: két régi olvasás, csak egy sikeres írás

A teszt kontrolláltan a két kérés által már elolvasott azonos verzióból indul. Nem thread-verseny és nem PostgreSQL lockteszt.

```python
from contextlib import closing
import sqlite3

class Conflict(Exception):
    pass

def rename(db, job_id, version, name):
    changed = db.execute('UPDATE jobs SET name = ?, version = version + 1 '
                         'WHERE id = ? AND version = ?', (name, job_id, version)).rowcount
    if changed != 1:
        raise Conflict('stale or missing job')

def test_stale_version_cannot_overwrite():
    with closing(sqlite3.connect(':memory:', isolation_level=None)) as db:
        db.execute('CREATE TABLE jobs(id INTEGER PRIMARY KEY, name TEXT, version INTEGER NOT NULL)')
        db.execute("INSERT INTO jobs VALUES (1, 'original', 1)")
        first_version = second_version = 1
        rename(db, 1, first_version, 'first change')
        try:
            rename(db, 1, second_version, 'second change')
        except Conflict:
            pass
        else:
            raise AssertionError('Expected optimistic conflict')
        assert db.execute('SELECT name, version FROM jobs').fetchone() == ('first change', 2)
```

A verzióellenőrzés és írás egy SQL-utasítás. Külön SELECT-es ellenőrzés, majd feltétel nélküli UPDATE újra megnyitná a versenyablakot. A hiányzó és régi verziójú job itt közös hiba; ha a publikus szerződés különbséget igényel, azt tudatosan kell feloldani.

## PostgreSQL és SQLite: eltérő működés

| Motor / szint | Fontos tulajdonság |
| --- | --- |
| PostgreSQL Read Committed | Jellemzően utasításonként új snapshot; egy tranzakció két olvasása eltérhet |
| PostgreSQL Repeatable Read | Tranzakciós snapshot; egyes konkurens műveleteknél abort szükséges |
| PostgreSQL Serializable | Soros végrehajtással összeegyeztethető eredmény vagy serialization failure |
| SQLite | Egyidejű írás korlátozott; a journal/WAL mód befolyásolja az olvasó–író viszonyt |

PostgreSQL-ben a Read Uncommitted Read Committedként viselkedik. Azonos elnevezésből ne következtess azonos lockmechanizmusra más adatbázisban. SQLite-on nincs PostgreSQL-szerű soronkénti FOR UPDATE tesztünk.

## Senior döntések és tipikus hibák

Serialization failure vagy deadlock után általában az egész érintett tranzakciót kell újraértékelni, korlátozott retryjal. A tranzakción kívül már megtörtént e-mail vagy HTTP-hívás nem gördül vissza. Retry előtt a mellékhatásokat és a művelet idempotenciáját is vizsgáld.

A konzisztens lock-sorrend csökkentheti a deadlock esélyét, de timeout és hibakezelés továbbra is kell. A „mindent Serializable-ra állítok” nem ingyenes korrekt működés: abortokat kell kezelni és a teljes munkafolyamat szerződését megőrizni.

## Interview questions

**Does using a transaction prevent all lost updates?**

Válaszvázlat: Nem; a konkrét izoláció és SQL számít. Verziófeltétel vagy megfelelő lock és újraértékelés kellhet.

**What should be retried after a serialization failure?**

Válaszvázlat: A teljes üzleti tranzakció új olvasásokkal, korlátozottan; külső mellékhatások ismétlését külön kell védeni.

## Önellenőrzés

- [ ] A verziófeltételt ugyanabba az UPDATE-be teszem.
- [ ] A SQLite-tesztet nem nevezem PostgreSQL-izolációs bizonyítéknak.

## Kapcsolódó gyakorlat

[DB04 – önálló feladat](../../../exercises/senior-python/10-databases.md#db04)

## Forrás és továbbolvasás

[PostgreSQL isolation](https://www.postgresql.org/docs/18/transaction-iso.html)

[PostgreSQL explicit locking](https://www.postgresql.org/docs/18/explicit-locking.html)

[SQLite isolation](https://www.sqlite.org/isolation.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
