# Öröklődés, helyettesíthetőség és super()

## Mit kell tudnod?

- Öröklődést viselkedési szerződés alapján alkalmazni.
- Megérteni, hogy a `super()` az MRO következő elemét követi.
- Felismerni a nem kooperatív többszörös öröklés hibáit.

## Magyarázat

Öröklődéskor az alosztály az alap típus helyén is használható kell legyen az ígért szerződés szerint. Ha egy általános tároló `save` metódusa sikert ígér érvényes adatnál, egy „csak olvasható” alosztály, amely mindig hibázik save-nél, rossz helyettesítő lehet. Érdemes külön olvasási és írási interfészt használni.

A felüldefiniálás (overriding) nemcsak azonos metódusnevet jelent. Bemenetek, eredmények, hibák és mellékhatások is számítanak. Szűkebb bemenet elfogadása vagy meglepően más eredmény megtöri a hívó feltételezéseit akkor is, ha a kód lefut.

## Kódpélda: kooperatív metóduslánc

```python
class Base:
    def steps(self) -> list[str]:
        return ['base']

class Audit(Base):
    def steps(self) -> list[str]:
        return ['audit', *super().steps()]

class Metrics(Base):
    def steps(self) -> list[str]:
        return ['metrics', *super().steps()]

class Worker(Audit, Metrics):
    pass

assert Worker().steps() == ['audit', 'metrics', 'base']
assert [cls.__name__ for cls in Worker.__mro__] == [
    'Worker', 'Audit', 'Metrics', 'Base', 'object'
]
```

Az `Audit` forrásában a közvetlen bázis `Base`, mégis a Worker esetén a `super().steps()` a Metrics metódusát találja meg. A super a tényleges példány MRO-jában folytatja a keresést az aktuális osztály után. Ezért pontatlan „a szülő hívásaként” kezelni.

## Tipikus hiba és javítás

Ha az Audit explicit `Base.steps(self)`-et hívna, kimaradna a Metrics viselkedése. Kooperatív hierarchiában minden résztvevő kompatibilis szignatúrát használ, és továbbhív, ahol a lánc ezt megköveteli. Konstruktoroknál ez különösen kényes: duplán inicializált vagy kihagyott állapot lehet az eredmény.

A C3 MRO megőrzi az öröklési sorrend következetességét; ellentmondó hierarchiát a Python osztálylétrehozáskor elutasíthat. A sorrend fejből történő levezetése helyett tudd megvizsgálni a `__mro__`-t, és tartsd egyszerűen a hierarchiát.

## Mikor használnád és senior szempontok

- Framework által megadott bővítési pontnál vagy kis, valóban helyettesíthető típuscsaládnál indokolt lehet az öröklődés.
- A mixin keskeny kiegészítő viselkedés legyen, világos elvárásokkal. Sok rejtett attribútumfeltétel vagy IO nehezen követhetővé teszi.
- Ha a tárolás, formátum és retry egymástól függetlenül változik, az öröklési kombinációk helyett composition használható.
- A base osztály publikus szerződésének változása minden alosztályra hat; teszteld ugyanazt a szerződést több implementáción is.

## Interview questions

**Does super() always call the direct parent?**

Válaszvázlat: nem; a tényleges MRO-ban keres tovább. Mutasd meg az Audit → Metrics átmenetet.

**How can an override violate substitutability?**

Válaszvázlat: szigorúbb előfeltétel, gyengébb eredménygarancia vagy meglepő mellékhatás. Az azonos név és annotáció nem elég.

## Önellenőrzés

- [ ] Előre megmondom a példa hívási sorrendjét.
- [ ] Felismerem, mikor jobb külön interfész, mint öröklődés.
- [ ] Nem keverem a kódújrahasználatot a helyettesíthetőséggel.


## Kapcsolódó gyakorlat

[O03 – önálló feladat](../../../exercises/senior-python/03-oop.md#o03)

## Forrás és továbbolvasás

[super](https://docs.python.org/3/library/functions.html#super) · [Inheritance](https://docs.python.org/3/tutorial/classes.html#inheritance)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
