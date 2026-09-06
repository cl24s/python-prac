# Job processing API

Állapot: tervezett; projektváz.

## Cél

API-tervezés, háttérmunka, állapotkezelés és megbízhatóság.

## Mérföldkövek

- [ ] M1: Tervezd meg a job létrehozó és állapotlekérdező API-t, majd készíts memóriabeli változatot.
- [ ] M2: Adj háttérfeldolgozást, korlátozott retry-t és világos állapotátmeneteket.
- [ ] M3: Vezess be tartós tárolást, idempotenciát, strukturált naplózást és graceful shutdownt.

## Elfogadási irányelvek

Az állapotátmenetek és a retry-határ tesztelt; az ismételt beküldés viselkedése és az újraindítás utáni működés dokumentált.

Minden mérföldkő előtt pontosítjuk az interfészt, a teszteseteket és a kész állapot feltételeit.

## Később hozzáadandó dokumentáció

- Futtatás és tesztelés a projekt README-jében.
- DESIGN.md: architektúra és döntések, ha a projekt mérete indokolja.
- NOTES.md: saját tanulságok és review, amikor lesz mit rögzíteni.

## Interview follow-up

How would you operate this application in production, and which failure modes would you address first?

[Vissza a projektekhez](../README.md)
