# Párhuzamos endpoint checker

Állapot: tervezett; projektváz.

## Cél

HTTP, korlátozott concurrency, timeout és cancellation.

## Mérföldkövek

- [ ] M1: Valósíts meg egy végpontellenőrzést állapot- és válaszidő-adatokkal.
- [ ] M2: Adj konfigurálható párhuzamossági limitet, kérésenkénti timeoutot és megszakítást.
- [ ] M3: Készíts összesítést és bizonyítsd teszttel a limitet és a rendezett leállást.

## Elfogadási irányelvek

A tesztek helyi tesztszervert használnak; lassú, hibás és sikeres válaszokra is kiterjednek; nincs hátramaradó munka leállás után.

Minden mérföldkő előtt pontosítjuk az interfészt, a teszteseteket és a kész állapot feltételeit.

## Később hozzáadandó dokumentáció

- Futtatás és tesztelés a projekt README-jében.
- DESIGN.md: architektúra és döntések, ha a projekt mérete indokolja.
- NOTES.md: saját tanulságok és review, amikor lesz mit rögzíteni.

## Interview follow-up

How would you operate this application in production, and which failure modes would you address first?

[Vissza a projektekhez](../README.md)
