# Konténeres futtatás, workerek és erőforráskorlátok

## Mit kell tudnod?

- A folyamat, worker és replika összesített költségét becsülni.
- Terhelés és függőségi kapacitás alapján skálázni.
- A buildet, indulást és leállítást külön ellenőrizni.

## Magyarázat

A konténer az alkalmazás futási környezetét csomagolja. A folyamatok továbbra is memóriát, CPU-t, fájlleírót és hálózati kapcsolatot fogyasztanak. A több worker nem ingyenes: a Python heap, connection pool és alkalmazáscache jellemzően folyamatonként jön létre.

A worker számát ne pusztán a host CPU-számából képezd. A konténer CPU-kvótája, a munka I/O- vagy CPU-jellege és a külső függőségek limitje határozza meg a hasznos párhuzamosságot. Több async task egyetlen folyamaton belül is túlterhelheti a DB-t.

## Kódpélda: connection budget rolling deploy alatt

```python
import pytest

def max_connections(*, replicas, surge, workers, pool_size, overflow):
    values = (replicas, surge, workers, pool_size, overflow)
    if any(type(value) is not int or value < 0 for value in values):
        raise ValueError('nonnegative integers required')
    if replicas == 0 or workers == 0 or pool_size == 0:
        raise ValueError('positive replicas, workers and pool size required')
    return (replicas + surge) * workers * (pool_size + overflow)

def test_rollout_budget():
    normal = max_connections(replicas=3, surge=0, workers=2, pool_size=5, overflow=2)
    rollout = max_connections(replicas=3, surge=1, workers=2, pool_size=5, overflow=2)
    assert normal == 42
    assert rollout == 56
    assert rollout + 12 <= 80  # Other clients and operational reserve.

def test_invalid_budget():
    with pytest.raises(ValueError):
        max_connections(replicas=3, surge=1, workers=2, pool_size=0, overflow=2)
```

Ez felső becslés egy fix poolmodellben, nem a pillanatnyilag megnyitott kapcsolatok száma. Feltételezi, hogy minden workernek egy ilyen poolja van. Több engine, külön worker deployment, migrációs job, terminating pod és autoscaling további kapcsolatokat adhat. A valós maximumot ezekkel együtt kell számolni; a példában a 12 csak deklarált saját tartalék.

## Futtatási terv

Build során különítsd el a buildeszközöket a runtime-tól, és csak a szükséges artifactot másold át. A base image digestje azonosítja a bájtokat; a rögzítés mellé rendszeres frissítés kell. Ne másolj secretet image-layerbe, még akkor sem, ha később törlöd. A runtime felhasználó ne legyen root, és csak indokolt könyvtár legyen írható.

Az induló parancs exec formája segít, hogy a szerver közvetlenül kapja a jeleket; wrapper esetén explicit továbbítás kell. SIGTERM után readiness le, új munka leállítása, folyamatban lévő munka lezárása és kapcsolatok elengedése következik. A platform türelmi idejének illeszkednie kell a szerver leállítási keretéhez. A SIGKILL nem futtat finally blokkot.

## Tipikus hibák

A memory limit túllépése folyamatleállást okozhat; CPU-korlát esetén a lassulás timeoutokat indíthat. A worker megsokszorozása nagyobb heapet és poolt hoz, és tovább ronthatja az állapotot. Mérj egy workerrel, majd figyeld a throughput, tail latency, RSS és pool-várakozás változását.

A lokális fájlrendszer ne legyen az üzleti tartósság egyetlen helye. A log kerüljön a platform által gyűjthető kimenetre, az üzleti adat tartós tárolóba. Ez a fejezet kapacitásmodellt ad; nem állítja, hogy tényleges konténer vagy Kubernetes rollout futott.

## Interview questions

**Why can adding workers make a service slower?**

Válaszvázlat: Több memória, kapcsolat és versengés keletkezik; a DB vagy CPU-kvóta már telített lehet.

**Why include deployment surge in connection budgets?**

Válaszvázlat: A régi és új példányok együtt élhetnek, ezért a normál replikaérték alábecsüli a csúcsot.

## Önellenőrzés

- [ ] A poolméretet teljes deploymentre számolom.
- [ ] Megkülönböztetem a rendezett leállást a killtől.

## Kapcsolódó gyakorlat

[OP05 – önálló feladat](../../../exercises/senior-python/13-packaging-security-and-operations.md#op05)

## Forrás és továbbolvasás

[Docker build best practices](https://docs.docker.com/build/building/best-practices/)

[Kubernetes resource management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
