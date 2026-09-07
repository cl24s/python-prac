# Tranzakcióhatár, rollback és kapcsolatélettartam

## Mit kell tudnod?

- Több írást egy üzleti tranzakcióba foglalni.
- A connection, session és tranzakció fogalmát elkülöníteni.
- Hibás művelet után visszaállított állapotot ellenőrizni.

## Magyarázat

Az atomikusság azt biztosítja, hogy a tranzakció együtt kezelt változtatásai sikerülnek vagy visszagördülnek. A konzisztencia a megőrzendő szabályokhoz kötődik, amelyeket az alkalmazás és a constraint együtt tart fenn. Az izoláció a konkurens megfigyelést szabályozza, a tartósság a sikeres commit megőrzésére vonatkozó garancia, a motor és konfiguráció szerint.

A connection a tárolóhoz vezető kapcsolat, a tranzakció ezen belüli munkaegység. A SQLAlchemy Session további ORM-állapotot tart nyilván. Bezárás, rollback és commit külön műveletek; egy poolba visszaadott kapcsolat később más kéréshez kerülhet.

## Kódpélda: terhelés után hibázó jóváírás visszagördül

```python
from contextlib import closing
import sqlite3
import pytest

def transfer(db, source, target, amount):
    if amount <= 0:
        raise ValueError('amount must be positive')
    db.execute('BEGIN')
    try:
        changed = db.execute('UPDATE accounts SET balance = balance - ? '
                             'WHERE id = ? AND balance >= ?', (amount, source, amount)).rowcount
        if changed != 1:
            raise ValueError('insufficient funds or missing source')
        if db.execute('UPDATE accounts SET balance = balance + ? WHERE id = ?',
                      (amount, target)).rowcount != 1:
            raise ValueError('missing target')
        db.execute('COMMIT')
    except BaseException:
        if db.in_transaction:
            db.execute('ROLLBACK')
        raise

def test_atomic_transfer():
    with closing(sqlite3.connect(':memory:', isolation_level=None)) as db:
        db.execute('CREATE TABLE accounts(id INTEGER PRIMARY KEY, balance INTEGER NOT NULL CHECK(balance >= 0))')
        db.executemany('INSERT INTO accounts VALUES (?, ?)', [(1, 100), (2, 20)])
        with pytest.raises(ValueError, match='missing target'):
            transfer(db, 1, 99, 30)
        assert db.execute('SELECT balance FROM accounts ORDER BY id').fetchall() == [(100,), (20,)]
        transfer(db, 1, 2, 30)
        assert db.execute('SELECT balance FROM accounts ORDER BY id').fetchall() == [(70,), (50,)]
```

Az isolation_level=None itt explicit SQL-tranzakciókezelést tesz lehetővé a Python 3.12 sqlite3 legacy alapbeállítása mellett. Nem feltételezünk implicit BEGIN-t. A BaseException csak rollback és azonnali továbbdobás miatt szerepel: nem nyeljük el a megszakítást. A függvény friss tranzakciót vár, meglévő tranzakcióba nem ágyazható automatikusan.

## Senior döntések és tipikus hibák

Pénzértéket ne bináris floatként kezelj; a példa egész alegységeket használ. Más domainben Decimal és megfelelő adatbázistípus lehet szükséges. A két UPDATE közé nem kerülhet külön commit, különben részleges átvezetés maradhat.

Hosszú tranzakció lockot, MVCC-verziót és kapcsolatot tarthat életben. Ne várj benne indokolatlanul felhasználóra vagy külső HTTP-szolgáltatásra. A pool nem pótolja a bezárást; a leak egy idő után pool timeoutként jelenhet meg.

Hálózati hiba commit közben eredménybizonytalanságot okozhat: a kliens nem tudja biztosan, megmaradt-e a tranzakció. A vak teljes ismétlés újabb átvezetést okozhat; ilyenkor műveletazonosító és visszakereshető üzleti eredmény kell.

## Interview questions

**Why is closing a connection not the same as committing?**

Válaszvázlat: A lezárás erőforrásművelet, a commit üzleti tranzakciós döntés. A függő munka rollbackelhet vagy elbukhat.

**Why can retry after a commit timeout be unsafe?**

Válaszvázlat: A commit lehet sikeres a szerveren; műveletazonosító nélkül az ismétlés duplikált hatást okozhat.

## Önellenőrzés

- [ ] A második írás hibája után az első is visszagördül.
- [ ] A pool timeout mögött kapcsolatszivárgást is keresek.

## Kapcsolódó gyakorlat

[DB03 – önálló feladat](../../../exercises/senior-python/10-databases.md#db03)

## Forrás és továbbolvasás

[Python sqlite3 transaction control](https://docs.python.org/3.12/library/sqlite3.html#transaction-control)

[SQLAlchemy connection pooling](https://docs.sqlalchemy.org/en/20/core/pooling.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
