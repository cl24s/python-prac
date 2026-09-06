# SOLID és gyakori patternök Pythonban

## Mit kell tudnod?

- Az elveket konkrét változtatási problémához kapcsolni.
- Strategy, Adapter, Factory és Repository szerepét megkülönböztetni.
- Felismerni, ha az absztrakció többe kerül, mint amennyit megold.

## Tervezési szempontok

| Elv | Gyakorlati kérdés | Python-példa |
| --- | --- | --- |
| Single responsibility | Mely változtatások miatt kell ezt módosítani? | Parsing és adatbázisírás külön felelősség |
| Open/closed | Valódi változási tengelyt könnyű bővíteni? | Átadott formázó új exportformátumhoz |
| Liskov substitution | Megmarad a hívó szerződése? | Fake és éles tároló azonos hibaszemantikája |
| Interface segregation | Csak a szükséges műveletre függünk? | Olvasó API nem igényel delete metódust |
| Dependency inversion | Ki határozza meg az interfészt? | Az alkalmazásigényhez igazodik az adapter |

Az SRP nem „egy osztály, egy metódus”. Az OCP nem tiltja meglévő kód módosítását. Egy első változatban gyakran egy világos elágazás a legegyszerűbb, és később a ténylegesen változó részt emeljük ki.

## Patternök és egyszerű megvalósításuk

- **Strategy:** cserélhető algoritmus. Pythonban egy callable is lehet, nem feltétlen osztály.
- **Adapter:** meglévő, eltérő API-t fordít az alkalmazás szerződésére; adatokat és hibákat is átalakíthat.
- **Factory:** az objektumépítés döntését egy helyre teszi; egyszerű függvény gyakran elegendő.
- **Repository:** domainhez illő adat-hozzáférési felület. A tranzakció és konzisztencia szabályát nem tünteti el.

## Kódpélda: Strategy osztályhierarchia nélkül

```python
from collections.abc import Callable, Sequence
import json

def as_json(names: Sequence[str]) -> str:
    return json.dumps(list(names), ensure_ascii=False)

def as_lines(names: Sequence[str]) -> str:
    return '\n'.join(names)

class Exporter:
    def __init__(self, render: Callable[[Sequence[str]], str]) -> None:
        self._render = render

    def export(self, names: Sequence[str]) -> str:
        return self._render(sorted(names))

assert Exporter(as_lines).export(['worker', 'api']) == 'api\nworker'
assert json.loads(Exporter(as_json).export(['worker', 'api'])) == ['api', 'worker']
```

A rendezés közös viselkedés, a formázás cserélhető. Ha nincs más közös állapot vagy művelet, az Exporter helyett egy `export(names, render)` függvény is teljesen megfelelő. Az osztály itt a composition bemutatását szolgálja.

## Tipikus hiba: túl általános repository

Egy `Repository[T]` húsz CRUD-metódussal látszólag újrahasznosítható, de elfedheti, hogy az egyik use case tranzakciós zárolást, a másik egyszerű lekérdezést igényel. Egy `find_pending_jobs(limit)` jobban kifejezheti a szükségletet. Az absztrakció legyen kicsi, de ne hazudjon a művelet költségéről vagy garanciájáról.

## Mikor használnád és senior szempontok

Új provider vagy új feldolgozási szabály bevezetésekor keresd, mely komponens változik valóban függetlenül. Csak ezt tedd cserélhetővé. A többi maradhat konkrét és egyszerű.

- A Factory ne váljon minden objektumot kezelő globális regiszterré.
- Az Adapter ne keverje össze a vendor hibakódját a domain döntésével.
- A Repository nem automatikusan gyorsabb vagy tisztább, mint jól szervezett közvetlen SQL; indokold a tesztelési vagy domainelőnyét.
- Pattern nevét interjún mindig konkrét probléma, választás és költség kövesse.

## Interview questions

**Can a Python function implement the Strategy pattern?**

Válaszvázlat: igen; callable szerződés és átadás elég, ha nincs szükség külön állapotkezelő objektumra.

**When can an abstraction make a design worse?**

Válaszvázlat: nem létező változást modellez, eltérő szemantikát mos össze, vagy felesleges navigációs és karbantartási költséget hoz.

## Önellenőrzés

- [ ] Mind a négy patternhöz tudok egy konkrét használati helyzetet.
- [ ] A SOLID-ot nem mechanikus osztálydarabolásként használom.
- [ ] Meg tudom mondani, mely absztrakciót hagynám el az első verzióból.


## Kapcsolódó gyakorlat

[O06 – önálló feladat](../../../exercises/senior-python/03-oop.md#o06)

## Forrás és továbbolvasás

[Callable types](https://docs.python.org/3/library/collections.abc.html#collections.abc.Callable) · [Protocols](https://typing.python.org/en/latest/spec/protocol.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
