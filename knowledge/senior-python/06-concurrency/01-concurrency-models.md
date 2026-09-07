# Concurrency, parallelism és a végrehajtási modell

## Mit kell tudnod?

- Megkülönböztetni az egymást átfedő feladatokat a tényleges párhuzamos CPU-végrehajtástól.
- A workload és a használt könyvtárak alapján választani async, thread és process között.
- Pontosan behatárolni a GIL szerepét és a runtime-verziót.

## Magyarázat

Concurrency esetén több feladat életciklusa átfed: egyik I/O-ra vár, miközben a másik halad. Parallelismnál ténylegesen egyszerre történik végrehajtás több végrehajtó egységen. Asyncio egy event loopjában a taskok kooperatívan váltanak; ez önmagában nem teszi többmagossá a CPU-számítást.

| Helyzet | Lehetséges választás | Költség vagy korlát |
| --- | --- | --- |
| Sok várakozás, async klienskönyvtár | asyncio | Blokkoló hívás az egész loopot feltarthatja |
| Meglévő sync I/O-kliens | ThreadPoolExecutor | Threadsafety, szál- és kapcsolatszám |
| Jelentős tiszta Python CPU-munka | ProcessPoolExecutor | Indítás, másolás/szerializáció, külön memória |
| GIL-t elengedő natív számítás | Thread is lehet jó | Könyvtár- és workloadfüggő mérés kell |

Hagyományos, GIL-es CPythonban egy interpreterben egyszerre jellemzően egy szál hajt végre Python-kódot. I/O közben és bizonyos natív műveleteknél a GIL elengedhető. A GIL nem garantálja az üzleti többműveletes állapotátmenet atomikusságát.

Python 3.13-tól elérhetők opcionális free-threaded CPython buildek. Ez nem jelenti azt, hogy minden Python-telepítés GIL nélkül fut, és egy extension kompatibilitása is számít. A tananyag példái CPython 3.12.13-on, a szokásos GIL-es buildben ellenőrzöttek; a szálbiztonsági szabályokat nem szabad a GIL-re bízni.

## Kódpélda: thread pool mint vezérlési eszköz

```python
from concurrent.futures import ThreadPoolExecutor

def normalize(name: str) -> str:
    return name.strip().lower()

with ThreadPoolExecutor(max_workers=2) as pool:
    results = list(pool.map(normalize, [' API ', 'WORKER']))
assert results == ['api', 'worker']
```

Ez a rövid függvény önmagában nem indokol thread poolt: a példa a használati formát mutatja, nem gyorsulási állítás. A map itt input-sorrendben ad eredményeket, miközben a tényleges befejezési sorrend eltérhet. A context manager kilépése megvárja a pool befejezését.

## Senior szempontok és tipikus hibák

- **Hiba:** „async gyorsabb”. **Javítás:** nevezd meg, mire várunk, hol van a CPU-munka, és támogatja-e a kliens az async használatot.
- A workerszám nem request/másodperc garancia. Adatbázispool, külső limit, latency és erőforrásigény együtt szabja meg a kapacitást.
- Sok rövid processzfeladatnál az IPC költsége meghaladhatja a számítást; batch-elés segíthet.
- CPU- és I/O-szakasz egy feladaton belül is váltakozhat. A határokat profiling alapján érdemes kialakítani.
- A konkurencia ára a bonyolultabb hibakezelés és leállás is; egyszerű szekvenciális megoldás jó kiindulópont lehet.

## Interview questions

**How would you choose between asyncio, threads, and processes?**

Válaszvázlat: workload, könyvtár API, GIL/native viselkedés, megosztott állapot és kommunikációs költség alapján.

**Does the GIL make application state thread-safe?**

Válaszvázlat: nem; check-then-act és több lépéses invariáns külön szinkronizációt igényelhet.

## Önellenőrzés

- [ ] Nem keverem a concurrencyt a többmagos CPU-végrehajtással.
- [ ] Verzióval és builddel együtt beszélek a GIL-ről.
- [ ] Gyorsulást csak megfelelő mérés alapján állítok.


## Kapcsolódó gyakorlat

[C01 – önálló feladat](../../../exercises/senior-python/06-concurrency.md#c01)

## Forrás és továbbolvasás

[concurrent.futures](https://docs.python.org/3.12/library/concurrent.futures.html) · [Free-threading support](https://docs.python.org/3/howto/free-threading-python.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
