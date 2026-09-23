# TM03 — Feladatkezelő két osztállyal

## Mit kell elkészítened?

A korábbi program objektumos változatát. A Task egy feladat adatait és készre állítását kezeli; a TaskManager több feladatot tart nyilván és ID alapján eléri őket. Nem új problémát kezdünk, hanem az eddigi működést szervezzük át.

**Saját fájl:** `work/task_models.py`. **Teszt:** `work/test_task_models.py`. A TM01–TM02 megoldásait és tesztjeit őrizd meg. A validációt használd újra, ne készíts három egymástól eltérő szabálykészletet.

## 1. rész — Task

A konstruktor hívási felülete: `Task(task_id, title, priority=2, assignee=None)`. Az elfogadott bemenetek és a hibák ugyanazok, mint a TM01 make_task függvényénél. Az új feladat completed értéke False.

Elvárt publikus műveletek:

| Művelet | Szerződés |
| --- | --- |
| `task.id`, `.title`, `.priority`, `.assignee`, `.completed` olvasása | A normalizált, aktuális értékek elérhetők. |
| `task.complete()` | Az adott példány completed értéke True lesz; visszatérés None. Ismételt hívás nem hiba, az állapot True marad. |
| `task.to_dict()` | Új dict a TM01 öt mezőjével és az aktuális completed értékkel. A dict későbbi átírása nem változtatja a Task állapotát. |

Az id publikus felületen legyen csak olvasható: a `task.id = 9` normál hozzárendelés `AttributeError`-t adjon. Ehhez használható property. Más mezők közvetlen felülírásának kezelését most nem teszteljük; az alkalmazás a készre állítást kizárólag complete-on keresztül végzi. A privát attribútumok szándékos megkerülése nem része a feladatnak.

Először csak a Task osztályt és tesztjeit írd meg. Sima osztály legyen saját konstruktorral; a dataclass későbbi összehasonlító változat lehet. Nem kell öröklődés, ABC, Protocol vagy saját egyenlőség/hash.

## 2. rész — TaskManager

Paraméter nélküli konstruktorral induló, üres nyilvántartás. Minden példány saját gyűjteménnyel rendelkezik. Az add_task garantáltan Task példányt kap; az ID-val hívott metódusok pozitív, nem bool intet kapnak. Itt ezekhez nem kell újabb típusvalidáció.

| Művelet | Szerződés |
| --- | --- |
| `add_task(task)` | Eltárolja a kapott Task példányt; visszatérés None. Duplikált ID → ValueError, az eredeti rekord megmarad. |
| `get_task(task_id)` | A tárolt Task példányt adja, nem másolatot. Hiányzó ID → KeyError. |
| `complete_task(task_id)` | A keresett Task complete műveletét hívja; visszatérés None. Hiányzó ID → KeyError. |
| `list_pending()` | Új listában a nem kész Task példányok, priority, majd id szerint növekvően. Üres állapotban üres lista. |

A manager nem férhet hozzá a Task privát attribútumaihoz, a publikus műveleteket használja. A publikus ID változtathatatlansága segít, hogy a nyilvántartás kulcsa ne váljon el a benne tárolt feladattól.

**Referencia-megosztási szabály:** a manager az átadott objektumot tárolja. Ha a hívó ugyanazon Task complete metódusát hívja, a manageren keresztül is késznek látszik. A list_pending új listájának ürítése viszont nem töröl feladatokat a managerből. Két manager üres gyűjteménye nem közös; ha ugyanazt a Task példányt szándékosan mindkettőbe felveszed, csak azt az egy objektumot osztják meg.

## Elvárt használat

Ez a viselkedés bemutatása, nem kész megoldás; csak a saját osztályaid megírása után futtatható:

```python
from task_models import Task, TaskManager

manager = TaskManager()
manager.add_task(Task(1, "Write tests", priority=2))
manager.add_task(Task(2, "Fix login", priority=1, assignee="Anna"))

assert [task.id for task in manager.list_pending()] == [2, 1]
manager.complete_task(2)
assert manager.get_task(2).completed is True
assert [task.id for task in manager.list_pending()] == [1]
```

## Mikor kész?

- [ ] Task konstrukcióra a TM01 fontos normál és hibás bemenetei működnek.
- [ ] Két Task közül az egyik készre állítása nem állítja készre a másikat. A complete kétszer is hívható.
- [ ] A to_dict új dict; a módosítása nem hat vissza a Taskra. Az id publikus újrakötése AttributeError.
- [ ] Üres manager és egyfeladatos manager működik. Hiányzó get/complete KeyError.
- [ ] Duplikált add_task nem írja felül az eredetit, akkor sem, ha más tartalmú objektumot adsz.
- [ ] A get_task ugyanazt az objektumot adja vissza, amelyet felvettél.
- [ ] Azonos prioritású feladatoknál id dönti el a sorrendet; list_pending nem ad kész feladatot.
- [ ] A visszaadott lista módosítása nem üríti ki a managert; két külön manager nem oszt véletlen közös gyűjteményt.
- [ ] A demo.py bemutat létrehozást, listázást, készre állítást és az új listát. A régi TM01–TM02 tesztek továbbra is futnak.

A teszteket lehet kisebb csoportokban írni: Task, majd üres/egyfeladatos manager, végül hibák és közös referenciák. Nem kell hálózatot vagy adatbázist mockolni, mert ilyen még nincs.

## Mit tanulsz közben?

Osztály, példány, self, konstruktor, példányattribútum, metódus, property és composition. A TaskManager használja a Taskot, nem abból öröklődik. Review-ban indokold, melyik felelősség tartozik a Taskhoz, melyik a managerhez, és miért veszélyes közös osztályattribútumban tárolni minden manager feladatait.

Későbbi bővítés lehet ugyanennek a managernek FastAPI-felületet adni. Az még külön feladat; most nincs HTTP, Pydantic, async, tartós tárolás vagy production-garancia.

Csak szükség szerint: [OOP-témakör](../../knowledge/senior-python/03-oop/README.md), [objektummodell](../../knowledge/senior-python/01-python-basics/01-object-model.md). Hivatalos referencia: [Python classes](https://docs.python.org/3.12/tutorial/classes.html).

[Vissza](README.md) · [Előző: TM02](02-data-structures.md)
