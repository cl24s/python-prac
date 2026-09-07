# Részleges hibák, eredménybizonytalanság és időkeretek

## Mit kell tudnod?

- A megfigyelt hibát elkülöníteni a szerver tényleges állapotától.
- Teljes műveleti deadline-t továbbadni részlépéseknek.
- Üzleti garanciát és mérhető szolgáltatási célt megfogalmazni.

## Magyarázat

Elosztott rendszerben a hívó és a végrehajtó eltérő tényeket láthat. A kliens timeoutot kap, miközben a szerver már commitolt; a worker befejezte a munkát, de az acknowledgement nem jutott el a brokerhez. A „nem kaptam választ” nem egyenlő a „nem történt meg” állítással.

| Megfigyelés | Amit tudsz | Amit még nem tudsz |
| --- | --- | --- |
| Kapcsolódási hiba | Az adott kapcsolódás nem sikerült | Másik korábbi próbálkozás végzett-e munkát |
| Olvasási timeout | Nem érkezett időben a várt adat | A szerver végrehajtotta-e a műveletet |
| Validációs elutasítás | A dokumentált kérés hibás | Érdemes-e más tartalommal újrapróbálni |
| DB commit után elveszett ack | Az üzleti adat megmaradhatott | A broker újra kézbesíti-e az üzenetet |

Ezért a siker fogalmát előre kell definiálni: elfogadva, tartósan rögzítve, feldolgozva vagy külső fél által visszaigazolva? A 202, a broker confirm és a domainállapot eltérő mérföldkő.

## Kódpélda: közös deadline, fogyó részkeretek

Minden új Python-blokk önálló pytest-modul. Másold `test_example.py` fájlba, majd `python -m pytest -q test_example.py`.

```python
import pytest

class DeadlineExceeded(Exception):
    pass

def step_budget(deadline, clock, per_step_cap):
    remaining = deadline - clock()
    if remaining <= 0:
        raise DeadlineExceeded('operation deadline reached')
    if per_step_cap <= 0:
        raise ValueError('positive cap required')
    return min(remaining, per_step_cap)

def test_shared_budget():
    now = [100.0]
    clock = lambda: now[0]
    deadline = clock() + 5.0
    assert step_budget(deadline, clock, 3.0) == 3.0
    now[0] += 4.0
    assert step_budget(deadline, clock, 3.0) == 1.0
    now[0] += 1.0
    with pytest.raises(DeadlineExceeded):
        step_budget(deadline, clock, 3.0)
```

Ez a függvény keretet számol, nem állít le önállóan semmit. A kapott keretet a kliensnek/drivernek is érvényesítenie kell. A poolban vagy queue-ban töltött idő is a teljes keretet fogyasztja. Éles kódban lokális időtartamhoz monoton órát használj.

## Határok és senior döntések

Egy processz monotonic() értéke nem hordozható elosztott abszolút időbélyeg. Másik szolgáltatásnak dokumentált timeout/deadline protokoll kell, hálózati késéssel és óraeltéréssel számolva. Az async timeout kooperatív; blokkolás vagy takarítás miatt a tényleges visszatérés későbbi lehet.

Saját példacél: „az elfogadott jobok 99%-a 60 másodpercen belül terminális állapotba jut”. Ehhez az elfogadás pillanatát, hibás terminális állapot kezelését és mérési ablakot is definiáld. A HTTP-handler p99 ideje nem méri a teljes háttérmunka korát. A queue legrégebbi elemének kora ezért gyakran fontosabb a puszta elemszámnál.

## Tipikus hibák

Minden részlépésnek új öt másodpercet adni nem öt másodperces összkeret. A hibákat sikeres alapértékké alakító fallback elrejtheti a kiesést. Ne nevezz „nem létező jobnak” egy DB-timeoutot: a bizonytalan eredmény külön kategória.

## Interview questions

**What does a timeout prove about a remote write?**

Válaszvázlat: Csak a hívó várakozásának lejártát; a távoli hatás lehet sikeres, ezért visszakeresés és idempotencia kell.

**Can a monotonic timestamp be shared between services?**

Válaszvázlat: Nem általánosan; lokális órához tartozik. A hálózati határon külön időkeret-protokoll szükséges.

## Önellenőrzés

- [ ] Elkülönítem az elfogadott, rögzített és befejezett állapotot.
- [ ] A queue-várakozást is beleszámolom az időkeretbe.

## Kapcsolódó gyakorlat

[R01 – önálló feladat](../../../exercises/senior-python/11-reliable-services.md#r01)

## Forrás és továbbolvasás

[Python monotonic clock](https://docs.python.org/3.12/library/time.html#time.monotonic)

[Async timeouts](https://docs.python.org/3.12/library/asyncio-task.html#timeouts)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
