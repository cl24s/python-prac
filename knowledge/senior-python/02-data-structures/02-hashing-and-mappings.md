# Dict, set, hash-elhetőség és egyenlőség

## Mit kell tudnod?

- A `dict`-et kulcs szerinti hozzáférésre, a `set`-et tagságvizsgálatra használni.
- Megérteni a hash és az egyenlőség kapcsolatát.
- Dönteni a hiányzó kulcs, ismétlődő kulcs és sorrend kezeléséről.

## Magyarázat

A hash-alapú konténer a hash segítségével szűkíti a keresést, az egyenlőséggel dönti el, hogy az adott kulcsot találta-e meg. Egyenlő objektumokhoz azonos hash szükséges; azonos hash nem jelenti, hogy az objektumok egyenlők. A hash-ütközés önmagában nem adatvesztés.

A kulcs hash-ének stabilnak kell maradnia az objektum életében, és az egyenlőségi szerződéssel összhangban kell működnie. A mutable built-in lista és dict nem hash-elhető; a tuple csak akkor, ha az elemei is azok. Saját osztálynál az `__eq__` és `__hash__` együtt tervezendő. Ha `__eq__`-t definiálsz és nem adsz megfelelő hash-t, a szokásos esetben az objektum hash-elhetetlenné válik.

## Kódpélda: kulcs és sorrend

```python
connections = {('api.local', 443): 'healthy'}
assert connections[('api.local', 443)] == 'healthy'

try:
    hash((['api'], 443))
except TypeError:
    pass
else:
    raise AssertionError('A tuple with a list is not hashable')

counts = {'api': 1, 'worker': 2}
counts['api'] = 3
assert list(counts) == ['api', 'worker']
del counts['api']
counts['api'] = 4
assert list(counts) == ['worker', 'api']
```

A dict megőrzi a beszúrási sorrendet. Egy meglévő érték frissítése nem új beszúrás; törlés és újbóli beszúrás igen. A setnek nincs ilyen felhasználható sorrendgaranciája. API-kimenethez szükség esetén rendezz explicit módon.

## Hiány, duplikáció és normalizálás

```python
settings = {'timeout': None}
assert settings.get('timeout') is None
assert settings.get('missing') is None
assert 'timeout' in settings
assert 'missing' not in settings

services = ['api', 'worker', 'api']
assert list(dict.fromkeys(services)) == ['api', 'worker']
assert set(services) == {'api', 'worker'}

mixed = {True: 'boolean', 1: 'integer'}
assert len(mixed) == 1
assert mixed[True] == 'integer'
```

A `get()` default nélkül összemossa a hiányzó és a `None` értékű kulcsot. Ha ez számít, tagságvizsgálatot vagy saját sentinelt használj. A `True` és `1` egyenlő kulcsként viselkedik; heterogén azonosítóknál emiatt fontos az input típusának tisztázása.

## Mikor használnád és milyen hibákat keress?

- Sok ismételt azonosítókereséshez építs dict-indexet vagy setet; a létrehozás memória- és időigényét is számold hozzá.
- **Hiba:** `{row['id']: row for row in rows}` csendben elfedi a duplikált ID-t. **Javítás:** döntsd el, első, utolsó, összevonás vagy hiba a kívánt szerződés.
- A `.get(key, expensive())` defaultja a hívás előtt kiértékelődik akkor is, ha a kulcs létezik. Lusta előállításhoz explicit elágazás kell.
- Futások között stabil adatbázis- vagy cache-azonosítót ne a beépített `hash()`-ből képezz; bizonyos típusok hash-e processzenként változhat.

## Interview questions

**What is the contract between equality and hashing?**

Válaszvázlat: egyenlő objektumok hash-e azonos; fordítva nem igaz; a hash stabil, az egyenlőség konzisztens kell legyen.

**How do you preserve order while removing duplicates?**

Válaszvázlat: hash-elhető elemeknél `dict.fromkeys`, vagy `seen` set és kimenetlista; tisztázd az első előfordulás megtartását és a memóriaigényt.

## Önellenőrzés

- [ ] Megmagyarázom a hash-ütközés és kulcsegyenlőség különbségét.
- [ ] Nem támaszkodom set-iterálási sorrendre.
- [ ] Explicit szabályom van hiányzó és ismétlődő kulcsra.


## Kapcsolódó gyakorlat

[Önálló feladat: D02](../../../exercises/senior-python/02-data-structures.md#d02)

## Forrás és továbbolvasás

[Mapping types](https://docs.python.org/3/library/stdtypes.html#mapping-types-dict) · [Hashable glossary](https://docs.python.org/3/glossary.html#term-hashable)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
