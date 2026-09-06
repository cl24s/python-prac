# Shallow copy, deep copy és adat-tulajdonlás

## Mit kell tudnod?

- Felismerni, mely objektumokat oszt meg két adatstruktúra.
- Megkülönböztetni a hozzárendelést, a sekély és a mély másolást.
- A szükséges mélységű másolatot választani automatikus `deepcopy` helyett.

## Magyarázat és példa

Egy sekély másolat új külső konténert hoz létre, de a benne tárolt objektumokra mutató referenciákat átveszi. Emiatt a külső lista bővítése elszigetelt, a belső dict módosítása közös lehet. A `deepcopy` rekurzívan másol, és egy másolási műveleten belül nyilvántartja a már feldolgozott objektumokat: ez kezeli a ciklusokat és őrzi a megosztás szerkezetét.

```python
from copy import deepcopy

original = {'targets': ['api'], 'retries': 2}
shallow = original.copy()
deep = deepcopy(original)

shallow['targets'].append('worker')
shallow['retries'] = 5
assert original == {'targets': ['api', 'worker'], 'retries': 2}
assert deep == {'targets': ['api'], 'retries': 2}
assert shallow['targets'] is original['targets']
assert deep['targets'] is not original['targets']
```

Az integer újrakötése csak a `shallow` dict egyik bejegyzését cseréli ki. A listamutáció viszont mindkét dictből látható. Hasonló sekély másolat a lista `copy()` metódusa vagy a `[:]` szeletelés.

## Tipikus hiba: ismétlés nem jelent új elemeket

```python
bad_rows = [[]] * 3
bad_rows[0].append('failed')
assert bad_rows == [['failed'], ['failed'], ['failed']]

rows = [[] for _ in range(3)]
rows[0].append('failed')
assert rows == [['failed'], [], []]

from copy import deepcopy
shared = []
clone = deepcopy([shared, shared])
assert clone[0] is clone[1]
assert clone[0] is not shared
```

Az ismétlés ugyanazt a belső listát használja háromszor. A comprehension minden körben újat hoz létre. A `deepcopy` példája pedig mutatja: a mély másolás nem feltétlenül szünteti meg az összes belső aliasingot; a másolt gráfon belüli eredeti megosztást megőrzi.

## Mikor használnád?

Egy kérés konfigurációját szeretnéd kiegészíteni úgy, hogy az alapkonfiguráció ne változzon. Ha csak egy ismert lista bővül, elég lehet a külső dict és annak a listának a másolása:

```python
base = {'targets': ['api'], 'retries': 2}
request_config = {**base, 'targets': [*base['targets'], 'worker']}
assert base['targets'] == ['api']
assert request_config['targets'] == ['api', 'worker']
```

Ez tudatosan csak a módosított ágat választja le. Más nested mezők későbbi módosítása előtt újra meg kell vizsgálni a szerződést.

## Senior szempontok

- A `deepcopy` nem alkalmazás-szintű snapshotgarancia. Egy DB-kapcsolatot, lockot vagy külső erőforrást nem kapsz meg értelmes, független üzleti másolatként.
- Nagy objektumgráfnál a másolás CPU- és memóriaigényes; mély lánc rekurziós problémát is okozhat.
- A concurrent írásokkal szembeni konzisztenciát külön kell biztosítani.
- Ha csak adatok kellenek tovább, jobb lehet kifejezetten új adatmodellt létrehozni a szükséges mezőkből.

## Interview questions

**Why can changing a shallow copy affect the original?**

Válaszvázlat: a külső konténer új, a beágyazott elemek közösek; mutáció és elemcsere különbözik.

**When would you avoid deepcopy?**

Válaszvázlat: nagy gráf, külső erőforrás, tisztázatlan ownership; célzott másolás vagy új adatmodell egyszerűbb lehet.

## Önellenőrzés

- [ ] Lerajzolás nélkül is azonosítom a közös belső listát.
- [ ] Elmagyarázom, miért hibás a `[[]] * n` független sorokhoz.
- [ ] Indoklom, milyen mélységű másolat kell egy adott függvényhez.


## Kapcsolódó gyakorlat

[Önálló feladat: B01](../../../exercises/senior-python/01-python-basics.md#b01)

## Forrás és továbbolvasás

[copy — shallow and deep copy](https://docs.python.org/3/library/copy.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
