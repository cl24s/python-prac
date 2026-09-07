# Processzek, CPU-munka és spawn

## Mit kell tudnod?

- A processzpool kommunikációs és memóriahatárát megérteni.
- Importálható workert és biztonságos belépési pontot készíteni.
- A worker eredményét és hibáját a szülőben megfigyelni.

## Magyarázat

Külön processzek külön interpretert és normál esetben külön memóriát kapnak. Ez hagyományos CPythonban tiszta Python CPU-munkát is több magra vihet, de a munkát és eredményt át kell adni a processzhatáron. A startup és szerializáció miatt sok apró feladat rosszul térülhet meg.

A start method platform- és verziófüggő. Windows alatt a spawn az alapvető modell; az alábbi példa mindenhol explicit spawn-t kér, így nem épít Linux-forkra. A gyermek új interpreterként importálja a szükséges modult. Emiatt a pool létrehozása az `if __name__ == '__main__'` blokk védelme alatt legyen.

## Kódpélda: CPU-feladat és workerhiba

A teljes blokkot `.py` fájlba másolva futtasd. Ne REPL-ben vagy stdinből indítsd: a spawn-nak importálható főmodul és felső szintű worker kell.

```python
from concurrent.futures import ProcessPoolExecutor
import multiprocessing as mp

def sum_squares(limit: int) -> int:
    if limit < 0:
        raise ValueError('limit must not be negative')
    return sum(number * number for number in range(limit))

def main() -> None:
    context = mp.get_context('spawn')
    with ProcessPoolExecutor(max_workers=2, mp_context=context) as pool:
        results = list(pool.map(sum_squares, [0, 10, 100]))
        failed = pool.submit(sum_squares, -1)
        try:
            failed.result(timeout=10)
        except ValueError as exc:
            assert str(exc) == 'limit must not be negative'
        else:
            raise AssertionError('Worker failure must reach the parent')
    assert results == [0, 285, 328350]

if __name__ == '__main__':
    main()
```

A program csak a helyes eredményt és a hibaátadást bizonyítja. Nem állít gyorsulást; ilyen kis bemenetnél a pool várhatóan több overheadet hoz. A példát Linux alatt explicit spawn-nal ellenőriztük, nem külön Windows-gépen.

## Adat- és erőforráshatárok

A szokásos processzpool argumentumai és eredményei pickle-alapú átadáson mennek át. Felső szintű függvény egyszerűbb; lambda, closure, nyitott klienskapcsolat vagy lock általában nem alkalmas ugyanarra a használatra. Ne untrusted pickle-adatot tölts be.

A szülő globális listájának módosítása a gyermekben nem lesz automatikusan közös állapot. Shared memory vagy multiprocessing eszközök explicit szerződést igényelnek. Élő adatbázis-kapcsolatot ne örökölt erőforrásként ossz meg; worker-specifikus inicializáció indokolt lehet.

## Tipikus hibák és senior szempontok

- **Hiba:** top-level poolindítás guard nélkül. **Javítás:** main függvény és main guard, különösen spawn mellett.
- Nagy objektumok oda-vissza küldése gyorsan elviheti a CPU-nyereséget; kisebb input, batch, fájlazonosító vagy tudatos shared memory lehet jobb.
- A `result(timeout=...)` a várakozást korlátozza, nem állítja le a futó munkát. A pool context kijárata ettől még várhat.
- `shutdown(cancel_futures=True)` a sorban váró munkákat célozza; a már futókat nem teszi visszamenőleg töröltté.
- Worker processz rendellenes megszűnése BrokenProcessPool hibát okozhat; az új pool indítása és a feladat újraküldése nem lehet vak retry mellékhatásos munkánál.

## Interview questions

**Why must a process-pool worker be importable with spawn?**

Válaszvázlat: a gyermek új interpreterből tölti be a definíciót; nem a szülő élő closure-állapotát kapja meg.

**Why can multiprocessing be slower for small tasks?**

Válaszvázlat: processzindítás, szerializáció, IPC és eredménykezelés meghaladhatja a számítás idejét.

## Önellenőrzés

- [ ] A példámat önálló fájlból és spawn-nal futtatom.
- [ ] Megfigyelem a worker exceptionjét a szülőben.
- [ ] A timeoutot nem keverem processzkilövéssel.


## Kapcsolódó gyakorlat

[C06 – önálló feladat](../../../exercises/senior-python/06-concurrency.md#c06)

## Forrás és továbbolvasás

[multiprocessing contexts](https://docs.python.org/3.12/library/multiprocessing.html#contexts-and-start-methods) · [ProcessPoolExecutor](https://docs.python.org/3.12/library/concurrent.futures.html#processpoolexecutor)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
