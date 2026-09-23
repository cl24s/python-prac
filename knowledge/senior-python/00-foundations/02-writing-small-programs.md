# Rövid programok írása

## Mit kell tudnod?

A bemenetből kimenetet előállító kis függvényt önállóan megírni. Kezdetben nem a legrövidebb vagy leginkább „pythonic” kód a cél, hanem a helyes, érthető működés.

## Függvény, feltétel és visszatérés

A paraméter a függvény bemenete, a `return` visszaad egy eredményt a hívónak. A `print` csak kiír; nem helyettesíti a visszatérési értéket. Az `if`/`elif`/`else` elágazások és a behúzás határozzák meg, mely sorok futnak.

```python
def shipping_cost(weight: int) -> int:
    if weight < 0:
        raise ValueError("negative weight")
    if weight == 0:
        return 0
    if weight <= 5:
        return 800
    return 1500

assert shipping_cost(0) == 0
assert shipping_cost(5) == 800
assert shipping_cost(6) == 1500
```

A példa egész számú, nem bool bemenetet feltételez. A type hint dokumentál és statikus ellenőrzőt segíthet; önmagában nem ellenőrzi a futás közbeni típust. Ne adj a feladathoz nem kért validációs frameworköt.

## Ciklus és állapot

A `for` bejár egy kollekciót. A gyűjtő változó általában a ciklus előtt jön létre, az összes eredményt visszaadó `return` pedig a ciklus után. A `continue` az aktuális lépés hátralévő részét hagyja ki; a `break` a ciklusból lép ki.

```python
def positive_total(values: list[int]) -> int:
    total = 0
    for value in values:
        if value <= 0:
            continue
        total += value
    return total

assert positive_total([3, -1, 0, 5]) == 8
assert positive_total([]) == 0
```

`while` akkor hasznos, amikor feltételig ismételsz, és nem egy kész kollekciót jársz be. Ilyenkor külön ellenőrizd, mi változtatja meg a kilépési feltételt. Listabejáráshoz ne vezess be feleslegesen indexet és while-ciklust.

## Konténer és jelentés

| Eszköz | Kezdő használati helyzet |
| --- | --- |
| `list` | Elemek sorrendben, ismétlődés is lehet. |
| `tuple` | Összetartozó értékek, például `(name, quantity)`. |
| `dict` | Kulcshoz tartozó érték, például azonosítóhoz rekord. |
| `set` | Egyedi elemek és tagságvizsgálat; ne várj szerződés szerinti sorrendet. |

```python
prices = {"pear": 300, "apple": 200}
assert prices["apple"] == 200
assert prices.get("plum", 0) == 0
assert "pear" in prices
assert sorted(prices) == ["apple", "pear"]

pairs = [("x", 2), ("y", 4)]
labels = []
for name, quantity in pairs:
    labels.append(f"{name}={quantity}")
assert labels == ["x=2", "y=4"]
```

A `dict` bejárása alapból kulcsokat ad; kulcs és érték együtt az `.items()` eredményéből kérhető. A `sorted` új listát ad, a `list.sort()` helyben módosít és `None`-t ad. Új külső konténer nem jelent automatikusan mély másolatot; erre külön gyakorlat a B01.

## String és kis építőelemek

A `strip()` a szöveg szélső whitespace-ét távolítja el, a `split()` részekre bont, a `join()` stringelemeket fűz össze. Ezeket a szerződés szerint használd: ne normalizáld automatikusan a kis- és nagybetűket, ha az számít. Az `enumerate` indexet és értéket ad; a `range` egész számok bejárásához használható.

A comprehension később egy egyszerű szűrés vagy leképezés rövidítése lehet, de egy olvasható for-ciklus önmagában nem rossz megoldás. Mélyebb anyag: [iterálás](../01-python-basics/04-iteration-and-comprehensions.md), [szekvenciák](../02-data-structures/01-sequences.md), [dict és set](../02-data-structures/02-hashing-and-mappings.md).

## Munkamódszer és önellenőrzés

Írj kézzel egy normál és egy üres bemenethez tartozó elvárt eredményt. Mondd el, milyen állapotot tartasz a bejárás során. Ezután írj kódot és tesztet. Ha megakadsz, különítsd el: az algoritmus hiányzik, vagy csak egy Python-művelet neve?

**Explain:** What is the difference between printing a result and returning it?

Kapcsolódó feladat: [L01–L04](../../../exercises/senior-python/00-foundations.md), [D01–D02](../../../exercises/senior-python/02-data-structures.md), [B01](../../../exercises/senior-python/01-python-basics.md#b01).

[Python control flow](https://docs.python.org/3/tutorial/controlflow.html) · [Adatszerkezetek](https://docs.python.org/3/tutorial/datastructures.html) · [Vissza](README.md)
