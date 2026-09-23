# TM02 — Feladatok keresése, rendezése és összesítése

## Mit kell elkészítened?

A feladatkezelő lekérdezéseit: gyors ID szerinti elérést, rendezett tennivalólistát és rövid összesítést. Még nem kell osztály vagy API.

**Saját fájl:** `work/task_queries.py`. **Teszt:** `work/test_task_queries.py`. A TM01 adatmodelljét használod; annak függvényei helyett tesztben kézzel megadott érvényes rekordokkal is dolgozhatsz, hogy a lekérdezéseket külön teszteld.

## Bemeneti szerződés és mintaadat

A `tasks` lista érvényes, a TM01 mezőit tartalmazó dict-ekből áll. A cím és felelős már normalizált, a completed bool lehet True vagy False. Itt nem kell újra ellenőrizned minden mezőt. Duplikált ID csak az index_tasks hibatesztjében fordulhat elő; a másik két függvény egyedi ID-kat kap.

**Tesztadat, nem megoldás:**

```python
tasks = [
    {"id": 1, "title": "Write tests", "priority": 2, "assignee": "Anna", "completed": False},
    {"id": 2, "title": "Fix login", "priority": 1, "assignee": "Bela", "completed": True},
    {"id": 3, "title": "Update docs", "priority": 1, "assignee": None, "completed": False},
    {"id": 4, "title": "Add metrics", "priority": 1, "assignee": "Anna", "completed": False},
    {"id": 5, "title": "Archive logs", "priority": 3, "assignee": None, "completed": True},
]
```

## 1. rész — index_tasks(tasks)

Adj vissza egy új dictionaryt: `id -> eredeti feladatrekord`. Például az `index_tasks(tasks)[3]` az Update docs rekordot adja. Üres inputra üres dict kell.

Azonos ID két előfordulása `ValueError`, akkor is, ha a rekordok egyébként azonosak. Nem írhatod felül csendben az elsőt. Az index saját külső dict, az értékei szándékosan az eredeti rekordobjektumok; például `index[1] is tasks[0]` igaz. Ez nem snapshot: a rekord későbbi módosítása az indexen keresztül is látható.

## 2. rész — pending_tasks(tasks)

Adj vissza új listát csak a még nem kész feladatokról. Rendezés: először priority szerint növekvő, azonos prioritásnál id szerint növekvő. Ne rendezd helyben a bemeneti listát. Az eredmény rekordjai itt is az eredeti objektumokra hivatkoznak, nem kötelező másolatokat gyártani.

A fenti mintára az eredmény ID-sorrendje pontosan `[3, 4, 1]`. Ez a lista a rekordok helyett csak az ellenőrizendő sorrendet mutatja: a függvény teljes rekordokat adjon vissza.

## 3. rész — summarize_tasks(tasks)

Eredmény a fenti mintára:

```python
expected = {
    "total": 5,
    "completed": 2,
    "pending": 3,
    "assignees": ["Anna", "Bela"],
}
```

A három számláló minden bemeneti feladatra vonatkozik; completed és pending összege a total. Az assignees az összes feladat nem None felelőseinek egyedi, növekvő, kis-/nagybetűt megkülönböztető sorrendű listája, a kész feladatok felelőseit is beleértve. `"Anna"` és `"anna"` külön név. Az egyedi nevek gyűjtéséhez használj setet, az eredményhez listát.

Üres input eredménye: `{"total": 0, "completed": 0, "pending": 0, "assignees": []}`. A kimeneti dict kulcssorrendjére nincs külön követelmény. A bemenetet egyik függvény sem módosíthatja a hívás során.

## Mikor kész?

- [ ] A közös minta és az üres lista mindhárom függvénnyel tesztelt.
- [ ] Az index első/későbbi duplikált ID-nál is hibázik; az eredeti lista és rekordok nem változnak.
- [ ] Minden kész feladat esetén a pending lista üres; egyetlen feladat esete is működik.
- [ ] Fordított bemeneti sorrendnél is `[3, 4, 1]` a minta pending ID-sorrendje. A bemenet sorrendje a hívás után változatlan.
- [ ] Az index és pending eredmény külső konténerének módosítása nem módosítja a bemeneti listát. A közös rekordreferenciákat külön ellenőrzöd és megmagyarázod.
- [ ] Ismétlődő felelős, csak None felelősök és eltérő kis-/nagybetű külön tesztelt.
- [ ] A demo.py kiírja a prioritásos tennivalókat és az összesítést. A három feldolgozó függvény maga nem ír ki.

## Mit tanulsz közben?

List, dict, set, tuple mint összetett rendezési kulcs, bejárás, szűrés, aggregáció, rendezés és objektumreferenciák. Nem a legrövidebb kód a cél. A szokásos listabemenet elegendő; generátor és korlátos memóriájú feldolgozás most nem követelmény.

Az első helyes változat után mondd el, hány bejárást és milyen extra tárolást használsz. Nem kell saját rendezőalgoritmust írni vagy benchmarkot építeni. Opcionális későbbi változat: készültségi arány `0.0`–`1.0` között, üres inputra `0.0`; ez nem része a mostani kész állapotnak.

Csak szükség szerint: [szekvenciák](../../knowledge/senior-python/02-data-structures/01-sequences.md), [dict és set](../../knowledge/senior-python/02-data-structures/02-hashing-and-mappings.md), [komplexitás](../../knowledge/senior-python/02-data-structures/04-complexity.md), [rendezés](../../knowledge/senior-python/02-data-structures/05-sorting-searching-grouping.md).

[Vissza](README.md) · [Előző: TM01](01-types-and-validation.md) · [Következő: TM03](03-oop.md)
