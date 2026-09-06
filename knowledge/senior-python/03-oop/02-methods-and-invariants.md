# Metódusok, property-k és invariánsok

## Mit kell tudnod?

- Instance method, classmethod és staticmethod közül cél szerint választani.
- Érvénytelen állapotot megelőzni, sikertelen módosításnál az eredetit megőrizni.
- Megkülönböztetni a Python-konvenciót a valódi hozzáférésvédelemtől.

## Magyarázat

Az instance method a példányt (`self`), a classmethod az osztályt (`cls`) kapja automatikusan. A staticmethod egyiket sem. Alternatív konstruktorhoz a classmethod jó választás: a `cls(...)` a ténylegesen hívott osztályt példányosítja. A staticmethod csak névtérbe szervezett függvény; ha nem kötődik szorosan a típushoz, modulfüggvény is megfelel.

Az invariáns olyan feltétel, amelynek az objektum érvényes állapotaiban mindig teljesülnie kell. A konstruktor és minden módosító út őrizze. Több mezőt érintő változtatáshoz gyakran explicit metódus kell; külön setterek köztes érvénytelen állapotot engedhetnek.

## Kódpélda: validált kapacitás

```python
class Capacity:
    def __init__(self, limit: int) -> None:
        self._limit = self._validate(limit)

    @staticmethod
    def _validate(value: int) -> int:
        if isinstance(value, bool) or not isinstance(value, int):
            raise TypeError('limit must be an integer')
        if value <= 0:
            raise ValueError('limit must be positive')
        return value

    @classmethod
    def from_text(cls, text: str) -> 'Capacity':
        return cls(int(text))

    @property
    def limit(self) -> int:
        return self._limit

    @limit.setter
    def limit(self, value: int) -> None:
        validated = self._validate(value)
        self._limit = validated

capacity = Capacity.from_text('8')
capacity.limit = 10
try:
    capacity.limit = 0
except ValueError:
    pass
else:
    raise AssertionError('Invalid update must fail')
assert capacity.limit == 10
```

A validáció az állapotcsere előtt fut. Így a sikertelen módosítás nem rontja el a korábbi érvényes állapotot. A `bool` külön tiltása itt tudatos üzleti szabály, mert Pythonban a bool az int alosztálya.

## Encapsulation és property

A `_limit` belső használatot jelez; technikailag kívülről is elérhető. A `__limit` name manglingja főként öröklési névütközést kerül el, nem titkosít és nem biztosít jogosultságot. Az encapsulation lényege, hogy a normál felhasználó stabil műveletekkel dolgozik, és nem függ a belső reprezentációtól.

A property attribútumjellegű API mögé tehet számítást vagy validációt. Ne rejts mögé meglepő hálózati hívást vagy drága műveletet: a `fetch_status()` név jobban jelzi annak árát. Setter nélküli property olvasási felületet ad, de a teljes objektum ettől még nem válik immutable-lé.

## Mikor használnád és tipikus hibák

Kapacitás, állapotátmenet vagy értéktartomány szabályozásakor. Egyszerű adatátviteli objektumnál nem kötelező minden mezőhöz getter és setter.

- **Hiba:** előbb eltárolod az új értéket, utána ellenőrzöd. **Javítás:** validálás, majd egy jól meghatározott állapotváltás.
- **Hiba:** classmethodban hardcode-olt alap-osztályt adsz vissza. **Javítás:** `cls(...)`, ha az alosztályok ugyanazt a konstrukciós szerződést tartják.
- Több konkurens hívó esetén az invariáns megőrzése lockot vagy más koordinációt is igényelhet; property önmagában nem szinkronizál.

## Interview questions

**When should an alternative constructor be a classmethod?**

Válaszvázlat: az osztálytól függő példányosításhoz; a `cls` támogatja az öröklést, ha a konstruktor kompatibilis.

**Does a leading underscore make an attribute private?**

Válaszvázlat: konvenció, nem hozzáférésvédelem. A stabil publikus API és a közös szabályok őrzése a lényeg.

## Önellenőrzés

- [ ] Sikertelen setter után érvényes marad az objektum.
- [ ] Megindoklom a három metódusfajta választását.
- [ ] Nem keverem a name manglingot biztonsági védelemmel.


## Kapcsolódó gyakorlat

[O02 – önálló feladat](../../../exercises/senior-python/03-oop.md#o02)

## Forrás és továbbolvasás

[property, classmethod, staticmethod](https://docs.python.org/3/library/functions.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
