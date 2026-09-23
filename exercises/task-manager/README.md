# Feladatkezelő — tanulás egy kis program építésével

**Állapot:** három kidolgozott feladatkiírás, nem kész alkalmazás. Az implementációt és a saját teszteket te írod. A Python-futtatás a felhasználó jelzése alapján működik; nem kezdjük újra a környezet telepítését.

Egy kis feladatkezelőt építesz: feladatokat hozol létre, keresel, rendezel, összesítesz és készre állítasz. Előbb egyszerű függvényekkel és dictionarykkel, majd objektumokkal dolgozol. Ugyanerre később FastAPI-felület kerülhet.

| Lépés | Kész kiírás | Mit készítesz? |
| --- | --- | --- |
| TM01 | [Típusok és validáció](01-types-and-validation.md) | Ellenőrzött feladatrekordot létrehozó függvények. |
| TM02 | [Adatszerkezetek](02-data-structures.md) | ID szerinti index, prioritásos lista és összesítés. |
| TM03 | [OOP](03-oop.md) | Task és TaskManager osztály, elkülönített állapottal. |

A táblázat a három meglévő kiíráshoz navigál; az aktuális feladatot és a tényleges haladást kizárólag a [ROADMAP](../../ROADMAP.md) vezeti. Most a TM01 az első, nem mindhárom egyszerre.

## A közös adatmodell

| Mező | Jelentés |
| --- | --- |
| `id` | Pozitív egész azonosító, nem bool; kezdetben a hívó adja. |
| `title` | Nem üres, a szélein whitespace-től megtisztított szöveg. |
| `priority` | 1, 2 vagy 3; 1 a legsürgősebb, alapérték 2. |
| `assignee` | Felelős neve vagy `None`, ha még nincs kiosztva. |
| `completed` | Bool; új feladatnál `False`. |

Nincs adatbázis, fájlmentés, hálózat, dátum vagy automatikus ID-generálás. Az első három feladatban standard library elég; a tesztekhez pytest használható. A következő programindításkor elvesző memóriaállapot most szándékos korlát.

## Hová írd a saját kódot?

Hozz létre itt egy `work/` mappát. A következő fájlok ajánlott célhelyek, **nem már elkészített megoldások**:

```text
exercises/task-manager/work/
    task_validation.py       # TM01
    task_queries.py          # TM02
    task_models.py           # TM03
    demo.py                  # rövid használati példa, te írod
    test_task_validation.py
    test_task_queries.py
    test_task_models.py
```

Most csak a `work/` mappát és a TM01-hez szükséges fájlokat hozd létre. Ne nevezd a modult `types.py`-nak: használjuk a fenti, egyértelmű fájlneveket. Egy munkakönyvtárban bővíted a programot; nem kell minden lépésnél új repó vagy package.

A `work/` mappából, a már működő Python-környezeteddel:

```bash
python demo.py
python -m pytest -q
```

A parancsok a saját fájlok megírása után használhatók. Pytest nélkül kezdetben saját assert-ellenőrzésekkel is dolgozhatsz. Ha a tesztcsomag még nincs a kiválasztott környezetben, a `python -m pip install pytest` telepíti; interpretert ne cserélj emiatt.

## Munkamód

Előbb olvasd el a feladatot, és kezdd el a megoldást. A kiírás végén csak az ahhoz szükséges elméleti hivatkozások szerepelnek; nincs előzetesen elvégzendő teljes kurzus. Dokumentációt használni és segítséget kérni szabad. Először egy kisebb részt fejezz be, utána review és a következő rész következik.

A használati példák **elvárt viselkedést** mutatnak; az importált függvények és osztályok még nincsenek megírva. Ezek nem kész megoldások és önmagukban nem futtatható programok. A saját tesztek ne csak a példákat másolják: a kiírt szélső eseteket is ellenőrizzék.

## Későbbi bővítés — most még nem feladat

TM03 után az egyik lehetséges következő lépés ugyanehhez a programhoz FastAPI: létrehozás, listázás, ID szerinti lekérés és készre állítás. Akkor külön rögzítjük a HTTP-szerződést, validációt és teszteket. Az API-hoz nem kell majd újra kitalálni az alapműködést. FastAPI-implementáció és részletes API-kiírás most nem készült.

[Összes gyakorlat](../README.md) · [START_HERE](../../START_HERE.md)
