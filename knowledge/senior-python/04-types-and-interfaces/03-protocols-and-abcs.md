# Protocol, ABC és strukturális típusosság

## Mit kell tudnod?

- Strukturális és névleges illeszkedést megkülönböztetni.
- Kis fogyasztói interfészt írni Protocolként.
- Az ABC és a runtime-checkable Protocol korlátait ismerni.

## Magyarázat

Névleges típusosságnál a deklarált típuskapsolat számít, például az öröklés. Strukturális megközelítésnél az számít, hogy az objektum biztosítja-e az elvárt tagokat megfelelő típusokkal. A Python Protocol ez utóbbit írja le a statikus ellenőrzőnek; a megvalósítónak nem kötelező a Protocolból örökölnie.

Az ABC explicit alap-osztályt és absztrakt műveleteket adhat, közös implementációval együtt. A közvetlen ABC-alosztály addig nem példányosítható, amíg absztrakt metódusai nincsenek megvalósítva. Ez runtime korlát, de nem ellenőrzi teljes egészében a metódusok típus- és viselkedési szerződését.

## Kódpélda: adapter explicit öröklés nélkül

```python
from typing import Protocol

class Clock(Protocol):
    def now(self) -> int: ...

class FixedClock:
    def now(self) -> int:
        return 100

def expires_at(clock: Clock, ttl: int) -> int:
    if ttl < 0:
        raise ValueError('ttl must not be negative')
    return clock.now() + ttl

assert expires_at(FixedClock(), 30) == 130
```

Az illeszkedéshez nem elég a `now` név; a paraméterek, visszatérés és szükség szerint a keyword argumentumnevek is számítanak. Csak a fogyasztó által használt műveletet írd az interfészbe.

## ABC és hiányzó implementáció

```python
from abc import ABC, abstractmethod

class Parser(ABC):
    @abstractmethod
    def parse(self, text: str) -> int:
        raise NotImplementedError

class IntegerParser(Parser):
    def parse(self, text: str) -> int:
        return int(text)

assert IntegerParser().parse('12') == 12
assert Parser.__abstractmethods__ == frozenset({'parse'})
```

A Parser közvetlen példányosítása `TypeError`-t okozna. A virtual subclass regisztráció (`register`) nem másol át metódusokat, és nem kényszeríti ki ugyanígy a megvalósítást; az explicit örökléssel nem egyenértékű.

## Runtime Protocol: nem validátor

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Reader(Protocol):
    def read(self) -> str: ...

class WrongReader:
    def read(self) -> int:
        return 42

candidate: object = WrongReader()
assert isinstance(candidate, Reader)
```

A runtime ellenőrzés nem vizsgálja a teljes metódusszignatúrát vagy visszatérési típust. A példa ezért futáskor sikeres, miközben a WrongReader nem lenne helyes statikus Reader-megvalósítás. Untrusted plugin viselkedését ez nem bizonyítja.

## Döntési szempontok és tipikus hibák

- Protocol: külső osztályok, fake-ek és fogyasztói igények összekapcsolása kevés öröklési kötöttséggel.
- ABC: kontrollált típuscsalád és közös alapviselkedés, explicit bővítési szerződéssel.
- **Hiba:** `runtime_checkable`-t inputvalidációnak tekintesz. **Javítás:** külön validáció és szerződésteszt.
- Módosítható Protocol-attribútum esetén az írási lehetőség a típuskompatibilitást is korlátozza; csak olvasáshoz property-t érdemes deklarálni.
- Egyik eszköz sem bizonyítja a timeoutot, sorrendet vagy hibaszemantikát: ezeket dokumentáld és teszteld.

## Interview questions

**When would you prefer Protocol over ABC?**

Válaszvázlat: strukturális integráció és keskeny fogyasztói felület, különösen nem saját osztályoknál; közös implementációhoz az ABC hasznos lehet.

**What does runtime_checkable fail to verify?**

Válaszvázlat: teljes szignatúra, visszatérési típus és szemantika. A metódus jelenléte kevés.

## Önellenőrzés

- [ ] Saját fake explicit öröklés nélkül illeszkedik a Protocolhoz.
- [ ] Megmagyarázom az ABC runtime példányosítási korlátját.
- [ ] Nem keverem a strukturális típust a viselkedés igazolásával.


## Kapcsolódó gyakorlat

[T03 – önálló feladat](../../../exercises/senior-python/04-types-and-interfaces.md#t03)

## Forrás és továbbolvasás

[Protocol specification](https://typing.python.org/en/latest/spec/protocol.html) · [ABC](https://docs.python.org/3/library/abc.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
