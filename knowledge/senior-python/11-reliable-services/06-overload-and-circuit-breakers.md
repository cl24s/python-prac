# Túlterhelés, bulkhead és circuit breaker

## Mit kell tudnod?

- A timeout, konkurenciakorlát és breaker eltérő szerepét megnevezni.
- Részleges kiesésnél kontrollált visszautasítást választani.
- Half-open próbák és fallback korlátait kezelni.

## Magyarázat

Timeout korlátozza a várakozást, konkurenciakorlát az egyszerre folyó munkát, rate limit az adott időszakban engedett forgalmat. A circuit breaker tartós hibázás után ideiglenesen elutasít bizonyos hívásokat, majd kevés próbával ellenőrzi a javulást. Ezek nem egymás szinonimái.

Bulkhead esetén egy hibás/terhelt függőség nem használhatja el minden más funkció kapacitását. Például a riportgenerálás külön workerkeretet kap az interaktív jobállapot-olvasástól. A közös adatbázis ettől még lehet közös szűk keresztmetszet.

## Kódpélda: soros breaker állapotgép injektált órával

```python
import pytest

class CircuitOpen(Exception):
    pass

class Breaker:
    def __init__(self, clock, threshold=2, cooldown=5):
        self.clock, self.threshold, self.cooldown = clock, threshold, cooldown
        self.failures, self.open_until = 0, None
    def call(self, operation):
        if self.open_until is not None and self.clock() < self.open_until:
            raise CircuitOpen('dependency is temporarily blocked')
        try:
            result = operation()
        except OSError:
            self.failures += 1
            if self.failures >= self.threshold:
                self.open_until = self.clock() + self.cooldown
            raise
        self.failures, self.open_until = 0, None
        return result

def test_open_then_probe():
    now = [0.0]
    breaker = Breaker(lambda: now[0])
    calls = []
    def fail():
        calls.append('failure')
        raise OSError('unavailable')
    for _ in range(2):
        with pytest.raises(OSError):
            breaker.call(fail)
    with pytest.raises(CircuitOpen):
        breaker.call(fail)
    assert len(calls) == 2
    now[0] = 5.0
    assert breaker.call(lambda: 'healthy') == 'healthy'
    assert breaker.failures == 0
```

A modell soros hívásokat feltételez; a cooldown utáni következő hívás a próbája. Nem thread-safe, és nincs konkurens half-open limitje. Éles breakerben több kérés egyszerre érkezhet a határra; csak korlátozott számú próba induljon, a régi hívások késői eredménye ne írja felül tévesen az új állapotot.

## Fallback és megfigyelés

Egy korábbi riport visszaadása lehet elfogadható fallback, ha megjelölöd az elavultságát. Egy aktuális jogosultság ellenőrzésénél az „engedjük át, mert a szolgáltatás lassú” más kockázat. Ne alakítsd a függőség hibáját hiányzó adattá vagy hamis sikeres eredménnyé.

A breaker állapot, elutasítások száma, pool-várakozás, queue-kor és downstream hibaarány együtt ad képet. Ha mindent retryzol a nyitott breaker fölött, helyben is terhelést generálsz. A breaker hibaosztályozása ne büntesse ugyanúgy az üzleti elutasítást és a szolgáltatás kiesését.

## Senior döntések és tipikus hibák

Ha a sor korlátlan, csak a hiba időpontját tolod későbbre. Admission controlnál dokumentált elutasítás kell, például megfelelő 429/503 a konkrét szerződés szerint. Ne ígérj kapacitást azért, mert nőtt a workerszám: a DB-keret nem nőtt vele.

A példabeli threshold/cooldown validált konfigurációt feltételez. A gyártási döntéshez valós terhelési mérés kell; egy soros állapotgép unit tesztje nem bizonyítja a teljes szolgáltatás stabilitását.

## Interview questions

**How is a circuit breaker different from a timeout?**

Válaszvázlat: A timeout az adott várakozást korlátozza; a breaker korábbi eredmények alapján új hívásokat is visszautasíthat.

**What can go wrong in half-open state?**

Válaszvázlat: Sok egyidejű próba újra túlterhelhet, és régi hívások késői eredménye összezavarhatja az állapotot.

## Önellenőrzés

- [ ] Megnevezem a fallback üzleti kockázatát.
- [ ] A soros mintát nem használom változtatás nélkül konkurens breakernek.

## Kapcsolódó gyakorlat

[R06 – önálló feladat](../../../exercises/senior-python/11-reliable-services.md#r06)

## Forrás és továbbolvasás

[Azure circuit breaker pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker)

[Azure bulkhead pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/bulkhead)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
