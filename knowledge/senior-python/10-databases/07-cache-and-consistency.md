# Cache, invalidálás és konzisztencia

## Mit kell tudnod?

- A cache szerepét és elavulási keretét rögzíteni.
- Cache-aside írás/olvasás versenyhelyzetét felismerni.
- A cache-kulcsot tenant és reprezentáció szerint tervezni.

## Magyarázat

Cache-aside esetén az alkalmazás először a cache-t nézi, missnél a forrást olvassa és betölti az eredményt. A cache gyorsítótár; a tartós igazság forrását és a megengedett elavulást külön kell megnevezni. A TTL korlátozhatja az adott cache-bejegyzés életét, de nem teszi az olvasást lineárisan konzisztenssé.

A kulcsnak minden releváns dimenziót tartalmaznia kell: tenant, objektum, szükség esetén nyelv, szűrés vagy sémaverzió. A jogosultság ellenőrzése nem hagyható ki cache-hit esetén sem. A DB identity map, a processzmemória és a megosztott Redis cache eltérő hatókör.

## Kódpélda: TTL óra-injektálással, tenant-határral

```python
class Cache:
    def __init__(self, clock):
        self.clock = clock
        self.entries = {}
    def put(self, tenant, key, value, ttl):
        if ttl <= 0:
            raise ValueError('positive ttl required')
        self.entries[(tenant, key)] = (self.clock() + ttl, value)
    def get(self, tenant, key):
        entry = self.entries.get((tenant, key))
        if entry is None:
            return None
        deadline, value = entry
        if self.clock() >= deadline:
            del self.entries[(tenant, key)]
            return None
        return value
    def invalidate(self, tenant, key):
        self.entries.pop((tenant, key), None)

def test_ttl_tenant_and_invalidation():
    now = [10.0]
    cache = Cache(lambda: now[0])
    cache.put('a', 'job:1', 'queued', ttl=5)
    assert cache.get('a', 'job:1') == 'queued'
    assert cache.get('b', 'job:1') is None
    now[0] = 15.0
    assert cache.get('a', 'job:1') is None
    cache.put('a', 'job:1', 'done', ttl=5)
    cache.invalidate('a', 'job:1')
    assert cache.get('a', 'job:1') is None
```

Ez a modell egyprocesszes, nem thread-safe, nincs háttérben futó eviction, és a None miss érték miatt None adat tárolására nem alkalmas szerződés. Nem Redis-implementáció és nem korlátos memóriájú cache. Csak a kulcs, TTL-határ és invalidálás szemantikáját vizsgáljuk.

## Az invalidálás után is visszakerülhet régi adat

1. A olvasó cache-misst kap és DB-ből kiolvassa a v1 adatot.
2. B író commitolja a v2 adatot, majd törli a cache-kulcsot.
3. A beírja a korábban kiolvasott v1-et a cache-be.

Az egyszerű „commit után delete” nem zárja ki ezt a versenyt. Lehetséges válasz verziózott cache-frissítés, megfelelő koordináció, rövid TTL és vállalt elavulás, vagy az adott kritikus olvasás cache-elésének elhagyása. Nincs követelményektől független tökéletes minta.

## Senior döntések és tipikus hibák

Stampede esetén sok egyidejű miss ugyanazt a DB-munkát indítja. Single-flight/coalescing, jitterelt lejárat vagy stale-while-revalidate segíthet, de a stale adat használatának üzleti korlátja legyen explicit. Egy processzlock nem koordinál minden replikát.

A negatív cache rövid időre hiányt is tárolhat, de új rekord létrehozása után így is elavult válasz jöhet. Cache-kieséskor a teljes DB-terhelés hirtelen visszaeshet az adatbázisra; fallbackhez kapacitás- és konkurenciakorlát kell.

Jogosultság, pénzügyi állapot vagy egyszer felhasználható token esetén a kis késés sem feltétlenül elfogadható. Előbb a konzisztenciakövetelményt mondd ki, utána válassz cache-t. Mérd a hit rate mellett a miss költségét, az elavulás hatását és az invalidálási hibákat is.

## Interview questions

**Why can stale data reappear after invalidation?**

Válaszvázlat: Egy korábbi olvasó az invalidálás után töltheti vissza a régi verziót; a delete önmagában nem koordinál minden olvasást.

**What does a high cache hit rate not prove?**

Válaszvázlat: Nem bizonyít helyes kulcsot, jogosultságot vagy friss adatot; üzleti konzisztenciát külön kell mérni.

## Önellenőrzés

- [ ] A tenant része a kulcsnak, de ez nem helyettesíti a policyt.
- [ ] A TTL-t nem nevezem erős konzisztenciának.

## Kapcsolódó gyakorlat

[DB07 – önálló feladat](../../../exercises/senior-python/10-databases.md#db07)

## Forrás és továbbolvasás

[Redis caching concepts](https://redis.io/docs/latest/develop/use/client-side-caching/)

[Redis key expiration](https://redis.io/docs/latest/commands/expire/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
