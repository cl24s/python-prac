# Dataclass, értékobjektum és egyenlőség

## Mit kell tudnod?

- Adatátviteli objektumot és üzleti viselkedést hordozó objektumot megkülönböztetni.
- Megérteni a generált egyenlőség és hash korlátait.
- A `frozen=True` és `default_factory` hatását pontosan megnevezni.

## Magyarázat

A dataclass kódgeneráló kényelmi eszköz: többek között inicializálót, reprezentációt és mezőalapú egyenlőséget készíthet. Nem külön „csak adat” objektumfajta, és nem tiltja az üzleti metódusokat. Annotationjeit alapból nem használja runtime típusvalidációra.

Értékobjektumnál az érték határozza meg az azonosságot: ugyanaz a host és port ugyanazt a végpontértéket jelentheti. Entitásnál az üzleti identitás állandó lehet változó mezők mellett; nem biztos, hogy minden mezőt bevonó generált `__eq__` kell.

## Kódpélda: értékobjektum szabállyal

```python
from dataclasses import dataclass, field, FrozenInstanceError

@dataclass(frozen=True)
class Endpoint:
    host: str
    port: int

    def __post_init__(self) -> None:
        if not self.host.strip():
            raise ValueError('host must not be empty')
        if isinstance(self.port, bool) or not 1 <= self.port <= 65535:
            raise ValueError('port is out of range')

first = Endpoint('api.local', 443)
second = Endpoint('api.local', 443)
assert first == second and first is not second
assert {first: 'healthy'}[second] == 'healthy'
try:
    setattr(first, 'port', 80)
except FrozenInstanceError:
    pass
else:
    raise AssertionError('Frozen fields cannot be reassigned normally')

@dataclass
class Batch:
    items: list[str] = field(default_factory=list)

left, right = Batch(), Batch()
left.items.append('job-a')
assert right.items == []
```

A konstruktorban validált értéktartomány üzleti szabály. A példa belső, megfelelően típusozott bemenetet feltételez; külső ismeretlen payloadhoz külön parser kell. A `default_factory=list` minden példányhoz új listát ad.

## Frozen nem jelent mély immutabilitást

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)
class Labels:
    values: list[str] = field(default_factory=list)

labels = Labels()
labels.values.append('prod')
assert labels.values == ['prod']
try:
    hash(labels)
except TypeError:
    pass
else:
    raise AssertionError('A list field cannot be hashed')
```

A frozen az attribútumok szokásos újrakötését korlátozza, a bennük lévő mutable objektumot nem fagyasztja meg. A hash a mezőktől is függ. Mutable mezőknél az `unsafe_hash=True` nem általános javítás: a kulcs megváltoztatása hash-konténerben hibás keresést okozhat.

## Saját egyenlőség és senior döntések

Saját `__eq__` megírásakor idegen, nem támogatott típusra általában `NotImplemented` értékkel engedd a Python összehasonlítási mechanizmusát tovább dolgozni. Egyenlő objektumok hash-e legyen azonos és stabil. A dataclass generált egyenlősége azonos konkrét típusú példányokat hasonlít össze mezőnként; öröklésnél ez fontos.

A DTO megjeleníthet nyers állapotot; a domainobjektum őrizheti a szabályt. Egy `Job.start()` állapotátmenet jobb lehet, mint szabad `job.status = 'running'`, ha több mező és esemény összehangolása szükséges. Ne minden dataclass köré tegyél üres domainréteget: a valódi szabályok indokolják.

## Interview questions

**Does frozen=True make a dataclass deeply immutable?**

Válaszvázlat: nem; nested mutable mezők módosulhatnak. A reprezentáció megválasztása és a hash-szerződés is számít.

**When is generated field-by-field equality inappropriate?**

Válaszvázlat: identitásalapú entitásnál, vagy ahol technikai mezők nem részei az üzleti értéknek.

## Önellenőrzés

- [ ] Megkülönböztetem a frozen objektumot a mély immutable értéktől.
- [ ] Érték és identitás alapján tudok egyenlőségi szabályt választani.
- [ ] Nem feltételezek automatikus runtime validációt.


## Kapcsolódó gyakorlat

[O05 – önálló feladat](../../../exercises/senior-python/03-oop.md#o05)

## Forrás és továbbolvasás

[dataclasses](https://docs.python.org/3/library/dataclasses.html) · [Data model: equality and hashing](https://docs.python.org/3/reference/datamodel.html#object.__hash__)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
