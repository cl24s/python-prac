# Python alapok – önálló feladatok

Állapot: a feladatkiírások elkészültek; megoldás és review még nincs. Csak standard library szükséges. Kész megoldás nincs mellékelve. A feladatok saját megoldásai és tesztjei később a `01-python-basics/` almappába kerülhetnek.

[Tananyag](../../knowledge/senior-python/01-python-basics/README.md)

## B01

**Cél:** objektumreferenciák és célzott másolás.

Írj `with_target(config, target)` függvényt. A config pontosan `targets: list[str]` és `retries: int` mezőket tartalmaz, a target string. Az eredmény új dict, a targets az eredeti lista elemei plusz a target; duplikáció megengedett. Az eredeti config ne változzon. A visszaadott targets későbbi módosítása se érintse az eredetit.

Példa: `{'targets': ['api'], 'retries': 2}` és `'worker'` eredménye `{'targets': ['api', 'worker'], 'retries': 2}`.

- [ ] Üres targets, ismétlődő target és két egymás utáni hívás ellenőrzött.
- [ ] A külső dict és a targets lista külön objektum.
- [ ] Nem használsz deep copy-t; megindoklod, miért elég a választott mélység.

**Interview:** Which references are shared, and which must be independent?

## B02

**Cél:** függvényszerződés, defaultok és closure.

Írj `make_checks(names, *, prefix='check')` függvényt, amely callback-listát ad. A names stringlista. Minden argumentum nélküli callback a saját nevéhez tartozó `prefix:name` stringet adja. A names későbbi módosítása ne változtassa meg a már létrehozott callbackek eredményét.

Példa: `['api', 'worker']` → a két callback eredménye `'check:api'`, `'check:worker'`.

- [ ] Üres lista, duplikált nevek és eltérő prefix támogatott.
- [ ] A callbackek fordított sorrendű vagy többszöri hívása is helyes.
- [ ] Két függvényhívás nem oszt rejtett módosítható állapotot.
- [ ] Elmagyarázod, hogyan kerülöd el a late bindingot.

**Interview:** When is each captured value evaluated?

## B03

**Cél:** lazy, egyszer bejárható pipeline.

Írj `positive_numbers(lines)` generátort. Minden inputelem string; strip után az üres sort kihagyja, más értéket `int()` segítségével alakít. A pozitív értékeket adja vissza, nullát és negatívat kihagy. Hibás számszövegnél a bejárás dobjon `ValueError`-t.

Példa: `[' 3 ', '', '0', '-2', '7']` → `3, 7`.

- [ ] Listával és egyszer fogyasztható iteratorral is működik.
- [ ] Nem materializálja az inputot.
- [ ] Csak a következő eredményhez szükséges inputot fogyasztja el.
- [ ] `['3', 'bad']` esetén az első elem még lekérhető, a következő lépés hibázik.

**Interview:** Where should the caller catch parsing errors?

## B04

**Cél:** decorator és kivételmegőrzés.

Írj `count_calls(stats)` decorator factory-t. A stats hívó által adott dict `started`, `succeeded`, `failed` számlálókkal, kezdetben nullával. Sync függvény minden hívása növeli a started értéket; normál visszatérés succeeded, `Exception` esetén failed nő. A hibát változtatás nélkül tovább kell engedni. A stats kizárólag ezt a három kulcsot tartalmazza; konkurens hívás most nem része a feladatnak.

- [ ] Visszatérési érték és positional/keyword argumentumok megmaradnak.
- [ ] A függvénynév és docstring megmarad.
- [ ] Két sikeres és egy sikertelen hívás után a számlálók `3, 2, 1`.
- [ ] A sikertelen hívásnál ugyanaz az exception objektum jut a hívóhoz.

**Interview:** What would need to change for an async function?

## B05

**Cél:** context manager és hibaút.

Írj `managed_resource(factory)` context managert. A factory argumentum nélkül egy `close()` metódusú erőforrást ad. A manager ezt adja a blokk használatára, és sikeres factory-hívás után pontosan egyszer lezárja. A blokk hibáját nem nyeli el. A close a feladatban nem dob hibát.

- [ ] Normál blokk és hibázó blokk esetén is pontosan egy close történik.
- [ ] Factory-hibánál a blokk nem fut; nincs close-kísérlet nem létező erőforráson.
- [ ] Hamis erőforrással, hálózat nélkül bizonyítod a hívássorrendet.

**Interview:** Who owns cleanup if acquiring the resource fails halfway through?

## B06

**Cél:** import-mellékhatások és memóriatulajdonlás diagnózisa.

Készíts később egy apró package-et `settings`, `worker`, `cli` modulokkal. A CLI írja ki a beállított workernevet; a worker puszta importálása ne indítson munkát és ne írjon ki semmit. A belépési pont `python -m package.cli` formában induljon. A csomagnév szabadon választható, a dokumentációban rögzítsd.

Külön rövid Markdown-válaszban elemezd: egy globális listába minden feldolgozott payload bekerül, majd a lokális nevet töröljük és `gc.collect()` fut. Miért nőhet tovább a memória? Javasolj kapacitás- vagy életciklus-szabályt.

- [ ] Importálás csendes és nem futtat feldolgozást.
- [ ] A CLI egyszer írja ki a workernevet, nulla kilépési kóddal.
- [ ] Nincs körkörös import és kézi `sys.path`-módosítás.
- [ ] A memóriaválasz az élő referenciát azonosítja, és nem csak gyakoribb GC-t ajánl.

**Interview:** Is module-level state shared between worker processes?
