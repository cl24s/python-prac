# Decoratorok és függvényburkolók

## Mit kell tudnod?

- Érteni a dekorálás időpontját és a wrapper hívását.
- Megőrizni a függvény metaadatait és a visszatérési értékét.
- Felismerni a sync/async és a decorator-sorrend jelentőségét.

## Magyarázat

A decorator fogad egy objektumot, és visszaadja a névhez kötendő eredményt. Függvényeknél ez gyakran egy új függvény, amely az eredetit hívja. A `@audit` a definíció után lényegében a `function = audit(function)` műveletet fejezi ki.

Fontos különválasztani a dekorálást és a hívást. A decorator a definíció végrehajtásakor, sokszor import során fut. A visszaadott wrapper a későbbi hívásoknál fut. Több decorator esetén a függvényhez legközelebbi alkalmazódik először: `@outer` és `@inner` jelentése `outer(inner(function))`.

## Kódpélda: ellenőrizhető audit wrapper

```python
from functools import wraps

history = []

def audit(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        history.append(('start', function.__name__))
        try:
            return function(*args, **kwargs)
        finally:
            history.append(('end', function.__name__))
    return wrapper

@audit
def divide(total, count):
    """Compute an average."""
    return total / count

assert divide(12, count=3) == 4
assert divide.__name__ == 'divide'
assert divide.__doc__ == 'Compute an average.'
assert divide.__wrapped__(6, 2) == 3

try:
    divide(1, 0)
except ZeroDivisionError:
    pass
else:
    raise AssertionError('The wrapper must propagate errors')
assert history == [('start', 'divide'), ('end', 'divide')] * 2
```

A `return` továbbadja az eredményt. A `finally` a sikertelen hívás lezárását is rögzíti, miközben az eredeti hiba továbbterjed. A `wraps` metaadatokat és `__wrapped__` hivatkozást ad; nem végez runtime típusellenőrzést, és nem alakítja át a wrapper végrehajtási természetét.

## Mikor használnád?

Egységes méréshez vagy naplózáshoz, ha a viselkedés valóban több függvényen azonos. Paraméterezett decoratornál van egy külső függvény a beállításoknak, belül a tényleges decorator, azon belül a wrapper. Nem kell minden közös két sort dekorátorrá alakítani: egy explicit függvényhívás átláthatóbb lehet.

## Tipikus hibák és senior szempontok

- **Hiba:** elmarad a `return function(...)`; a hívó `None`-t kap. **Javítás:** őrizd meg a visszatérési szerződést.
- **Hiba:** minden exceptiont elnyelő logoló wrapper. **Javítás:** a naplózás után az eredeti hibát engedd tovább.
- Egy sync wrapper async függvénynél csak coroutine-t hozhat létre; a tényleges futást nem méri. Async méréshez `async def` és `await` kell. Ezt a concurrency részben mélyítjük el.
- Cache és authorization sorrendjénél vizsgáld, megkerülhető-e az ellenőrzés cache találatkor, és szerepel-e a jogosultsági kontextus a cache-kulcsban.
- A példabeli globális lista oktatási eszköz: élesben korlátlanul nőne, és érzékeny adatot sem szabad válogatás nélkül naplózni.

## Interview questions

**What does functools.wraps do, and what does it not do?**

Válaszvázlat: metaadatokat és az eredeti függvényre mutató kapcsolatot őriz; nem garantálja a wrapper helyességét vagy async-kompatibilitását.

**Why does decorator order matter?**

Válaszvázlat: egymásba ágyazott hívásokat eredményez; cache, jogosultság és mérés esetén ez a tényleges viselkedést változtatja.

## Önellenőrzés

- [ ] Leírom a `@decorator` nélküli ekvivalens alakot.
- [ ] A wrapperem eredményt ad vissza és megtartja az exceptiont.
- [ ] Megmondom, mikor fut a dekorálás és mikor a wrapper.


## Kapcsolódó gyakorlat

[Önálló feladat: B04](../../../exercises/senior-python/01-python-basics.md#b04)

## Forrás és továbbolvasás

[functools.wraps](https://docs.python.org/3/library/functools.html#functools.wraps)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
