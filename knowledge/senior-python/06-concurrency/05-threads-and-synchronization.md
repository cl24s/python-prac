# Threadek, race condition, lock és deadlock

## Mit kell tudnod?

- A több lépésből álló állapotváltozást közös kritikus szakaszként védeni.
- Megkülönböztetni a Lock, Event, Semaphore és Queue célját.
- A futó thread leállítását együttműködési protokollként kezelni.

## Magyarázat

A race condition akkor jelentkezik, amikor az eredmény a műveletek időzítésétől függ. A „kiolvasom, kiszámolom, visszaírom” három lépése közé másik szál léphet. Ne arra építs, hogy egy adott Python-verzió adott bytecode-sorozata épp nem váltott szálat.

## Kódpélda: determinisztikus lost update és javítás

```python
from concurrent.futures import ThreadPoolExecutor
from threading import Barrier, Lock

counter = 0
barrier = Barrier(2, timeout=5)

def unsafe_increment() -> None:
    global counter
    previous = counter
    barrier.wait()
    counter = previous + 1

with ThreadPoolExecutor(max_workers=2) as pool:
    futures = [pool.submit(unsafe_increment) for _ in range(2)]
    for future in futures:
        future.result()
assert counter == 1

counter = 0
lock = Lock()

def safe_increment() -> None:
    global counter
    for _ in range(1000):
        with lock:
            counter += 1

with ThreadPoolExecutor(max_workers=2) as pool:
    futures = [pool.submit(safe_increment) for _ in range(2)]
    for future in futures:
        future.result()
assert counter == 2000
```

A barrier az első példában mindkét olvasást a visszaírás elé kényszeríti. Így a hiba reprodukálható, nem véletlen stresszteszt. A javítás az egész frissítést védi. A barrier a hibát demonstrálja; ne tedd a lock belsejébe, mert a másik szál nem jutna el hozzá.

## Szinkronizációs eszközök

| Eszköz | Szerep |
| --- | --- |
| Lock | Egy kritikus szakasz kizárólagos használata |
| RLock | Ugyanaz a szál újra beléphet; nem old meg tetszőleges deadlockot |
| Event | Állapotjelzés, például stop-kérés |
| Semaphore | Egyszerre legfeljebb N felhasználó |
| queue.Queue | Threadek közti elemátadás, korlátos kapacitással és várakozással |

Deadlock lehet eltérő lock-sorrendből vagy abból, hogy egy pool worker ugyanabban a telített poolban indított másik future-re vár. Legyen közös lock-sorrend, rövid kritikus szakasz; idegen callbacket és lassú I/O-t lehetőleg ne hívj benne.

## Kódpélda: az async várakozás cancellationje után élő thread

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor
from threading import Event

async def main() -> None:
    started, release, finished = Event(), Event(), Event()

    def blocking_work() -> None:
        started.set()
        try:
            if not release.wait(timeout=5):
                raise TimeoutError('test release was not received')
        finally:
            finished.set()

    with ThreadPoolExecutor(max_workers=1) as pool:
        future = asyncio.get_running_loop().run_in_executor(pool, blocking_work)
        try:
            assert await asyncio.to_thread(started.wait, 5)
            future.cancel()
            try:
                await future
            except asyncio.CancelledError:
                pass
            assert not finished.is_set()
        finally:
            release.set()
            assert await asyncio.to_thread(finished.wait, 5)

if __name__ == '__main__':
    asyncio.run(main())
```

Az időlimitek tesztvédő korlátok, nem alvásra épített sorrendfeltételek. A release jelzés engedi a szálat ténylegesen befejeződni. A context manager végül a poolt is lezárja. Éles sync kliensben saját timeout kell, mert a kód nem feltétlen tud egy Eventet rendszeresen figyelni.

## Tipikus hibák és senior szempontok

- ThreadPoolExecutor `Future.cancel()` csak a még el nem indult munkát tudja törölni; futó függvényt nem szakít félbe.
- Workerhibát `future.result()`-tal vagy más felügyelt úton figyelj meg.
- Daemon thread nem helyettesít graceful shutdownt; a processz végén félkész munka maradhat.
- A threadsafe konténerművelet nem tesz atomikussá több műveletből álló invariánst.

## Interview questions

**Why is locking only the final assignment insufficient?**

Válaszvázlat: a köztes számítás elavult olvasatból indulhat; az egész read-modify-write szakaszt védeni kell.

**How do you stop a running worker thread?**

Válaszvázlat: együttműködő stop-jelzés és I/O-timeout, majd join; nincs általános biztonságos thread-kill.

## Önellenőrzés

- [ ] Reprodukálhatóan bemutatom a lost update okát.
- [ ] A teljes invariánst védem, nem csak egy sort.
- [ ] Future-törlés és tényleges workerleállás külön állapot.


## Kapcsolódó gyakorlat

[C05 – önálló feladat](../../../exercises/senior-python/06-concurrency.md#c05)

## Forrás és továbbolvasás

[threading](https://docs.python.org/3.12/library/threading.html) · [Future.cancel](https://docs.python.org/3.12/library/concurrent.futures.html#concurrent.futures.Future.cancel)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
