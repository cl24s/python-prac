# Composition, polimorfizmus és dependency injection

## Mit kell tudnod?

- Viselkedést együttműködő objektumokból összeállítani.
- Függőséget átadni ahelyett, hogy az üzleti kód rejtetten létrehozná.
- Megkülönböztetni a duck typingot, a szerződést és az infrastruktúrát.

## Magyarázat

Composition esetén az objektum egy másik objektum szolgáltatását használja: a notifier tartalmaz egy sendert. A notifiernek nem kell SMTP-senderből öröklődnie, mert nem ugyanazt a felelősséget képviselik. A polimorfizmus itt azt jelenti, hogy több eltérő megvalósítás ugyanazon szükséges műveleten keresztül használható.

A dependency injection egyszerűen a függőség kívülről történő átadása. Nem igényel konténert vagy frameworköt. A konstruktor paraméterei láthatóvá teszik, mire van szükség. Az összeszerelés helye (composition root) választ konkrét adaptert és felel az életciklusért.

## Kódpélda: tesztelhető együttműködés

```python
from typing import Protocol

class Sender(Protocol):
    def send(self, message: str) -> None: ...

class RecordingSender:
    def __init__(self) -> None:
        self.messages: list[str] = []

    def send(self, message: str) -> None:
        self.messages.append(message)

class Notifier:
    def __init__(self, sender: Sender) -> None:
        self._sender = sender

    def notify_failure(self, service: str) -> None:
        if not service.strip():
            raise ValueError('service must not be empty')
        self._sender.send(f'failed:{service}')

sender = RecordingSender()
notifier = Notifier(sender)
notifier.notify_failure('api')
assert sender.messages == ['failed:api']
try:
    notifier.notify_failure(' ')
except ValueError:
    pass
else:
    raise AssertionError('Invalid input must fail before sending')
assert sender.messages == ['failed:api']
```

A RecordingSender nem örököl Senderből: a szükséges műveletet megfelelő szignatúrával biztosítja. A Protocol ennek statikusan olvasható leírása; részleteit a következő témakör bontja ki. Annotation nélkül a Python ugyanúgy a tényleges metódust hívná (duck typing), csak kevesebb géppel ellenőrizhető szerződésünk lenne.

## Mikor használnád?

Email-küldés, időforrás, véletlenszám, tároló vagy HTTP-kliens leválasztására. Tesztben a fake determinisztikus viselkedést ad, valódi hálózat nélkül. Időfüggő üzleti szabálynál átadható egy `clock` callable; nem feltétlenül kell hozzá külön osztályhierarchia.

## Tipikus hibák és senior szempontok

- **Hiba:** a Notifier belül példányosítja az éles klienst. **Javítás:** kívül készül és paraméterként érkezik; így konfiguráció és életciklus is látható.
- **Hiba:** a konstruktor helyett globális service locatorból kérjük ki a függőséget. **Javítás:** explicit paraméter, amelyet a hívó és a teszt is lát.
- Az interfész az ügyfél szükségletét írja le; nem kell a vendor SDK összes metódusát lemásolni.
- Az injektált erőforrást ki zárja le? Alapértelmezett jó szerződés, hogy aki létrehozta, az birtokolja; az eltérést dokumentáld.
- Fake és valódi adapter azonos szemantikát tartson. Ha a valódi adapter hibázhat, a teszt ne kizárólag sikeres fake-et használjon.

## Interview questions

**Does dependency injection require a container?**

Válaszvázlat: nem; constructor vagy function injection elég. A konténer az összeszerelés automatizálását segítheti.

**How does composition reduce coupling?**

Válaszvázlat: a fogyasztó a szükséges műveletre támaszkodik; a konkrét megvalósítás és annak öröklési fája kevésbé szivárog át.

## Önellenőrzés

- [ ] Valódi hálózat nélkül tesztelem az együttműködést.
- [ ] Megnevezem a függőségek létrehozóját és tulajdonosát.
- [ ] Indoklom, miért ilyen kicsi az interfész.


## Kapcsolódó gyakorlat

[O04 – önálló feladat](../../../exercises/senior-python/03-oop.md#o04)

## Forrás és továbbolvasás

[Structural subtyping](https://typing.python.org/en/latest/spec/protocol.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
