# TM01 — Feladat létrehozása: típusok és validáció

## Mit kell elkészítened?

Egy programrészletet, amely a hívó által megadott adatokból érvényes feladatrekordot készít. A hibás adatot nem alakítja át találomra, hanem a megadott hibát jelzi.

**Saját fájl:** `work/task_validation.py`. **Teszt:** `work/test_task_validation.py`. Kész implementáció nincs mellékelve. Előfeltétel csak a működő Python; a kis függvényekhez szükséges fogalmakat közben is megtanulhatod.

## 1. rész — normalize_title

Írj `normalize_title(title: str) -> str` függvényt.

| Bemenet | Elvárt eredmény |
| --- | --- |
| `"  Write tests  "` | `"Write tests"` |
| `"Árvíz teszt"` | `"Árvíz teszt"` |
| `"Fix  login"` | `"Fix  login"` — a belső szóközök megmaradnak. |
| `""` vagy `" \t\n "` | `ValueError` |
| `None`, `42`, `True` vagy lista | `TypeError` |

A szélső whitespace-t távolítsd el; kis-/nagybetűt és belső whitespace-t ne változtass. A függvény visszaadjon egy stringet, ne csak kiírja. **Elsőként csak ezt a részt és a tesztjeit írd meg.**

## 2. rész — validate_priority

Írj `validate_priority(priority: int) -> int` függvényt. Csak az 1, 2, 3 egész érték érvényes; érvényes inputra ugyanazt a számértéket add vissza.

| Bemenet | Elvárt eredmény |
| --- | --- |
| `1`, `2`, `3` | Az adott int. |
| `0`, `-1`, `4` | `ValueError` |
| `"2"`, `2.0`, `None`, `True`, `False` | `TypeError` |

Nincs automatikus string–szám átalakítás. A bool itt nem prioritás. A hibatípus kötelező, a hibaüzenet pontos szövege nem; legyen érthető.

## 3. rész — make_task

Írj ilyen hívási felületű függvényt:

```text
make_task(task_id: int, title: str, priority: int = 2,
          assignee: str | None = None) -> dict[str, object]
```

Ez a felület leírása, nem kész függvénytörzs. A szerződés:

- `task_id`: pozitív int, bool nélkül. Nem megfelelő típus → `TypeError`; nulla vagy negatív egész → `ValueError`. Nem kell ID-t generálni vagy egyediséget ellenőrizni.
- `title`: a normalize_title szabálya szerint; használd a saját, már megírt függvényedet.
- `priority`: a validate_priority szabálya szerint; ha kimarad, 2.
- `assignee`: `None`, vagy string, amely strip után nem üres. Üres/whitespace string → `ValueError`; más típus → `TypeError`. A nevet stripelve, kis-/nagybetűt megőrizve add vissza. Nincs névjegyzék vagy felhasználó-ellenőrzés.
- Az eredmény új dict, pontosan az `id`, `title`, `priority`, `assignee`, `completed` mezőkkel. A `completed` értéke mindig `False`.

Több egyidejű hibánál nem kötjük meg, melyiket jelzed először. Az elfogadási tesztek egyszerre egy bemeneti hibát kérnek számon. Egyedi Python-alosztályok és külső könyvtárak skalártípusai nem részei a feladatnak.

**Elvárt használat — csak a saját implementáció elkészülte után futtatható:**

```python
from task_validation import make_task

result = make_task(1, "  Write tests  ", assignee=" Anna ")
assert result == {
    "id": 1,
    "title": "Write tests",
    "priority": 2,
    "assignee": "Anna",
    "completed": False,
}
```

## Mikor kész?

- [ ] A két kisebb függvény táblázatának minden esete tesztelt.
- [ ] A make_task működik csak id/title megadásával is; ilyenkor priority 2, assignee None, completed False.
- [ ] Tesztelt az érvénytelen azonosító, cím, prioritás és felelős, külön-külön.
- [ ] Két make_task hívás két külön dictet ad. Az első dict title mezőjének későbbi átírása nem változtatja a másodikat.
- [ ] A függvények nem kérnek inputot, nem írnak ki és nem használnak módosítható globális állapotot.
- [ ] A saját demo.py létrehoz és kiír két feladatot; az alkalmazási függvények helyett a demo felel a kiírásért.

Elég először egyszerű assert és célzott try/except ellenőrzés; pytest is használható. Nem kell Pydantic, TypedDict, mypy vagy osztály. A `dict[str, object]` egyelőre tág típusjelölés; nem követelmény szigorú statikus típusellenőrzés.

## Mit tanulsz közben?

String, int, bool, None, default paraméter, visszatérési érték, típusellenőrzés, értékellenőrzés és új dict. A type hint önmagában nem futásidejű validáció. A bool az int alosztálya, ezért a fenti szerződéshez erre is figyelned kell.

Review-kérdések: mi a különbség a `None` és az üres string között? Miért nem fogadjuk el automatikusan a `"2"` értéket? Mit csinál a type hint, és mit kell a saját kódodnak ellenőriznie?

Csak szükség szerint olvasd: [rövid programok](../../knowledge/senior-python/00-foundations/02-writing-small-programs.md), [korai tesztelés](../../knowledge/senior-python/00-foundations/03-testing-and-debugging.md). Hivatalos referencia: [Python típusok](https://docs.python.org/3.12/library/stdtypes.html), [type hintek](https://docs.python.org/3.12/library/typing.html).

[Vissza a feladatsorhoz](README.md) · [Következő: TM02](02-data-structures.md)
