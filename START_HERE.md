# Itt kezdd

## Most mi a cél?

**Tanulás kész, gyakorlati feladatkiírásokból.** Nem előbb az egész elmélet, majd egyszer egy projekt: egy kis feladatkezelőt építesz, és közben vesszük elő a szükséges Python-fogalmakat. A kész kiírás nem kész megoldás; a kódot és a saját teszteket te írod.

**A Python-futtatás kész a felhasználó 2026-09-23-i jelzése alapján.** Nem kell újra telepítési lépésekkel vagy kötelező D01-szintfelmérővel kezdeni. Ebből más készség elsajátítását nem következtetjük ki.

## A következő egy lépés

Nyisd meg a [TM01 — Típusok és validáció](exercises/task-manager/01-types-and-validation.md) kiírást. Elsőként csak az **1. rész: normalize_title** függvényt és a hozzá tartozó teszteket írd meg. Ehhez hozd létre az `exercises/task-manager/work/` mappát és a kiírásban javasolt saját fájlokat.

A további két kiírás is elkészült: [TM02 — Adatszerkezetek](exercises/task-manager/02-data-structures.md), [TM03 — OOP](exercises/task-manager/03-oop.md). Ezek nem egyszerre kiadott házi feladatok. [A feladatkezelő áttekintése](exercises/task-manager/README.md) megmutatja, hogyan kapcsolódnak össze; a tényleges állapotot a [ROADMAP](ROADMAP.md) vezeti.

## Egy munkamenet

1. Elolvasod az adott feladat vagy rész bemenetét, kimenetét és elfogadási feltételeit.
2. Önállóan próbálkozol. Dokumentáció, célzott elméleti magyarázat és kis segítség használható közben.
3. Saját tesztekkel ellenőrzöl, majd célzott review jön. Nem írjuk át automatikusan az egész megoldást helyetted.
4. Feljegyezzük, mi ment önállóan, mi akadt el, mi futott és mi a következő rész.
5. Egy későbbi alkalommal új változaton is használod az adott készséget. Nem kell azonnal időre dolgozni.

A `knowledge/` most háttéranyag, nem előzetesen kipipálandó kurzus. A régi D01, L01–L08 és más gyakorlatok megmaradnak célzott kisegítésre, de nem kell ugyanazt a készséget minden feladatsoron újrakezdeni.

## Mi jöhet később?

A TaskManager objektumos változatára később FastAPI-felület kerülhet. Ez külön következő feladat lesz; most az alapműködést építed fel HTTP, adatbázis és async nélkül. Nem kezdünk újabb tananyaggyártásba vagy infrastruktúra-építésbe a saját kód helyett.
