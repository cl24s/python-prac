# Erőforrás-tulajdonlás és megbízható takarítás

## Mit kell tudnod?

- Részleges erőforrás-megszerzés után is lezárni, amit már birtokolsz.
- Elkerülni, hogy a cleanup elnyomja az eredeti hibát.
- Megkülönböztetni az atomikus láthatóságot a tartósságtól.

## Magyarázat

A `with` és a try/finally nemcsak kényelmi szintaxis: láthatóvá teszi, ki felel a lezárásért. A függvény által kapott erőforrás alapértelmezett tulajdonosa gyakran a hívó; a függvény által megnyitotté maga a függvény. Ettől el lehet térni, de a szerződést rögzíteni kell.

Több dinamikus erőforráshoz az ExitStack a már regisztrált kilépési műveleteket fordított sorrendben futtatja. Ha a második megnyitás hibázik, az elsőt akkor is le kell zárni. Ha a belépés félúton meghiúsul még a regisztráció előtt, a részleges megszerzést az azt végző kód takarítsa.

## Kódpélda: részleges megszerzés

```python
from contextlib import ExitStack, contextmanager

history: list[str] = []

@contextmanager
def resource(name: str):
    history.append(f'open:{name}')
    try:
        yield name
    finally:
        history.append(f'close:{name}')

try:
    with ExitStack() as stack:
        stack.enter_context(resource('input'))
        stack.enter_context(resource('output'))
        raise OSError('write failed')
except OSError:
    pass
else:
    raise AssertionError('The operation must fail')
assert history == ['open:input', 'open:output', 'close:output', 'close:input']
```

A finally-ben lévő `return` elfedheti a korábbi exceptiont vagy visszatérést. Cleanup közben keletkező új exception is felülírhatja a kifelé látható hibát, az előző kontextusba kerülhet. Ne használj vak `except: pass`-t: döntsd el, melyik hiba elsődleges, és hogyan marad diagnosztizálható a másik.

## Kódpélda: kész fájl cseréje

```python
from pathlib import Path
from tempfile import TemporaryDirectory, NamedTemporaryFile
import os

with TemporaryDirectory() as directory:
    target = Path(directory) / 'report.txt'
    target.write_text('old', encoding='utf-8')
    temporary = None
    try:
        with NamedTemporaryFile(mode='w', encoding='utf-8',
                                dir=directory, delete=False) as stream:
            temporary = Path(stream.name)
            stream.write('new')
        os.replace(temporary, target)
    finally:
        if temporary is not None:
            temporary.unlink(missing_ok=True)
    assert target.read_text(encoding='utf-8') == 'new'
```

A cél mellé írt ideiglenes fájl ugyanazon fájlrendszeren marad; lezárás után történik a csere. A csere sikeres esetben atomikus névváltást ad, de ez nem teljes crash-durability garancia: fsync, könyvtárszinkron és fájlrendszer/platform szabályok is számíthatnak. Jogosultságokat és konkurens írókat ez az egyszerű példa nem koordinál.

## Mikor használnád és senior szempontok

Riportgenerálásnál a félkész kimenet publikálása helyett ideiglenes eredményt építesz. A `flush()` Python-puffer kiürítése nem ugyanaz, mint tartós lemezre írás. Adatbázis-tranzakció és fájlrendszeri csere pedig nem egy közös atomikus tranzakció.

Az `async with` lezárása maga is awaitelhet. Cancellation és ismételt megszakítás alatt a cleanupra is kell életciklus-szabály; ezt a concurrency-fejezet tárgyalja. Processzkilövés vagy géphiba ellen a finally önmagában nem véd.

## Interview questions

**How do you clean up if the second resource acquisition fails?**

Válaszvázlat: az első erőforrás már regisztrált cleanupja fusson; ExitStack vagy helyesen egymásba ágyazott context managerek segítenek.

**Is atomic file replacement the same as durable persistence?**

Válaszvázlat: nem; láthatóság és géphibát túlélő tartósság külön garancia.

## Önellenőrzés

- [ ] Nincs return a cleanup finally blokkjában.
- [ ] Megnevezem az erőforrás tulajdonosát.
- [ ] Nem állítok tranzakciós vagy tartóssági garanciát a példán túl.


## Kapcsolódó gyakorlat

[E03 – önálló feladat](../../../exercises/senior-python/05-error-handling.md#e03)

## Forrás és továbbolvasás

[contextlib](https://docs.python.org/3.12/library/contextlib.html) · [os.replace](https://docs.python.org/3.12/library/os.html#os.replace)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
