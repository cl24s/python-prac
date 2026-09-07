# SQL injection, command injection és deszerializáció

## Mit kell tudnod?

- Elkülöníteni az adatot a végrehajtható utasítástól.
- A shell elhagyása mellett az argumentumokat is szabályozni.
- Nem megbízható adathoz biztonságos formátumot és korlátokat választani.

## Magyarázat

Injection akkor jön létre, amikor a külső adat megváltoztatja egy interpreter utasítását. SQL-nél a paraméterezett érték külön jut el az adatbázishoz; shellnél az argumentumlista elkerüli a shell kifejezésnyelvét. A védelem nem általános „special character tisztítás”, hanem megfelelő API és engedélyezett műveletkészlet.

## Kódpélda: adat marad az input

```python
import sqlite3
import subprocess
import sys

def test_sql_values_do_not_change_query():
    connection = sqlite3.connect(':memory:')
    try:
        connection.execute('CREATE TABLE users (name TEXT PRIMARY KEY)')
        connection.execute('INSERT INTO users VALUES (?)', ('alice',))
        supplied = "alice' OR 1=1 --"
        rows = connection.execute('SELECT name FROM users WHERE name = ?', (supplied,)).fetchall()
        assert rows == []
        assert connection.execute('SELECT count(*) FROM users').fetchone()[0] == 1
    finally:
        connection.close()

def test_shell_characters_are_literal_arguments():
    supplied = 'hello; echo unexpected'
    result = subprocess.run(
        [sys.executable, '-c', 'import sys; print(sys.argv[1])', supplied],
        shell=False, check=True, capture_output=True, text=True, timeout=5,
    )
    assert result.stdout.strip() == supplied
```

A gyermek Python fix, saját kódot futtat; a külső szöveg csak `argv` elem. Ne kerüljön felhasználói adat a `-c` kód helyére. A példa nem indít shellt, nem futtatja a bemeneti echo szót külön parancsként.

## Az API-határ után is vannak szabályok

SQL-paraméterrel értéket adsz át, nem táblanevet vagy rendezési irányt. Dinamikus rendezésnél a külső `sort` értéket szerveroldali, fix oszlopkifejezéshez rendeld. A query csak engedélyezett azonosítókból álljon. Az ORM raw SQL lehetősége ugyanúgy veszélyes, ha string-interpolációval használod.

A `shell=False` nem akadályozza meg, hogy egy célprogram a `--output` vagy más opciót értelmezze. Válassz fix executable-t, korlátozott opciókat és validált erőforrást; ahol támogatott, a `--` elválasztó megállíthatja az opciófeldolgozást. A subprocess kimenetére, idejére és jogosultságára is kell korlát. Windows batch fájloknak külön shellviselkedése lehet; ezt a Linuxon ellenőrzött példa nem vizsgálja.

## Deszerializáció

A pickle nem biztonságos ismeretlen eredetű inputhoz: betöltése kódvégrehajtást okozhat. Cache, queue és feltöltött fájl sem válik megbízhatóvá a tárolás helyétől. JSON megfelelőbb adatcseréhez, de séma, méret, típusok és üzleti validáció akkor is kell. Túl nagy vagy mély adat erőforrást meríthet ki.

Ne használj `eval`-t konfiguráció vagy felhasználói adat olvasására. YAML-nál a biztonságos loader választása külön szükséges; az általános objektumkonstrukció kerülendő. Formátumváltáskor a régi fogyasztók és hibás üzenetek útját is kezeld.

## Interview questions

**Can parameterized SQL handle a user-selected column name?**

Válaszvázlat: Az értékparaméter nem SQL-azonosító. Külső választásból fix engedélyezett oszlopra kell leképezni.

**Is shell=False enough to make any subprocess safe?**

Válaszvázlat: Nem. A célprogram opcióit, a fájlokat, a jogosultságot és az erőforrásigényt továbbra is korlátozni kell.

## Önellenőrzés

- [ ] A példákban meg tudom mutatni az adat és utasítás határát.
- [ ] Nem töltök be nem megbízható pickle-t.

## Kapcsolódó gyakorlat

[OP06 – önálló feladat](../../../exercises/senior-python/13-packaging-security-and-operations.md#op06)

## Forrás és továbbolvasás

[OWASP SQL injection prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

[Python subprocess](https://docs.python.org/3.12/library/subprocess.html)

[Python pickle](https://docs.python.org/3.12/library/pickle.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
