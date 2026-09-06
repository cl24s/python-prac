# Típusok és interfészek – önálló feladatok

Állapot: feladatkiírás kész; megoldás és review még nincs. Kész megoldás nincs mellékelve. A megoldások és tesztek később a `04-types-and-interfaces/` almappába kerülhetnek. A pozitív példákra mypy strict ellenőrzést használj; a külön negatív minták elvárt statikus hibáját dokumentáld.

[Tananyag](../../knowledge/senior-python/04-types-and-interfaces/README.md)

## T01

**Cél:** pontos függvényhatárok.

Írj annotált `count_names(names)` függvényt, amely egyszer bejárható string-iterable-ből string → int darabszám-dictet ad. Ne igényeljen listát, ne módosítsa a bemenetet, ne használjon Any-t. Üres input üres dict. Példa: api/worker/api → api:2, worker:1.

- [ ] Mypy strict és runtime teszt is sikeres.
- [ ] Generátoros input támogatott.
- [ ] Külön negatív mintában egy int-elemeket adó inputot a checker elutasít.

**Interview:** What does the annotation promise about iteration count?

## T02

**Cél:** object szűkítése cast nélkül.

Írj `parse_limit(value: object) -> int` függvényt. Int fogadható el 1–1000 között, bool nem. Stringből kizárólag ASCII számjegyekből álló alak fogadható el külső whitespace eltávolítása után; előjel és tizedespont nem. Hibás típus `TypeError`, hibás forma vagy tartomány `ValueError`.

- [ ] A `10` és `' 10 '` eredménye 10; `True`, `'1.5'`, `'-1'`, `''`, 0 és 1001 elutasított.
- [ ] Nincs Any, cast vagy type: ignore.
- [ ] A típus- és tartományellenőrzés runtime is működik.

**Interview:** Why is isinstance(value, int) alone insufficient?

## T03

**Cél:** keskeny Protocol és szerződés.

Írj `Lookup` Protocolt `get(key: str) -> str | None` metódussal és `require_value(store, key) -> str` függvényt. Hiányzó értéknél `KeyError(key)`, egyébként változatlan string az eredmény, az üres string is érvényes. A memóriabeli implementáció ne örököljön a Protocolból.

- [ ] A fake statikusan illeszkedik, siker/hiány/üres érték tesztelt.
- [ ] Külön negatív mintában a `get() -> int` implementációt a checker elutasítja.
- [ ] Dokumentáld, mit nem bizonyítana egy runtime_checkable ellenőrzés.

**Interview:** Who should determine the shape of the Lookup interface?

## T04

**Cél:** generikus kapcsolat és read-only felület.

Írj `last(items: Sequence[T]) -> T` függvényt TypeVar segítségével. Üres szekvenciára `ValueError`; nem üresből az utolsó elem eredeti objektumát adja. Lista és tuple is támogatott, bemenet nem módosul.

- [ ] String inputelemnél str, integernél int az inferált eredmény; assert_type-pal ellenőrizd.
- [ ] A visszaadott objektum identitása megmarad.
- [ ] Külön magyarázd el, miért nem jelent immutable snapshotot a Sequence annotáció.

**Interview:** Could the function accept Iterable instead without changing its implementation?

## T05

**Cél:** külső payload és belső modell.

Definiálj `CreateTaskPayload` TypedDictet kötelező `name: str`, opcionálisan jelen lévő `retries: int` mezővel. Készíts runtime parsert `Mapping[str, object]` bemenetre és frozen belső adatmodellt. A name trim után nem lehet üres, retries default 0, megengedett 0–5 int, bool nélkül; ismeretlen kulcs `ValueError`.

- [ ] Hiányzó name és extra mező `ValueError`, rossz mezőtípus `TypeError`, rossz tartomány `ValueError`.
- [ ] A hiányzó retries és explicit None eltérően viselkedik: az utóbbi hiba.
- [ ] A parser nem használ castot validáció helyett.
- [ ] A belső modell saját konstrukciós útja is megőrzi az érvényességi szabályokat.

**Interview:** Which checks belong to the input format, and which to the domain model?

## T06

**Cél:** típusmegőrző decorator.

Írj `trace_calls(events)` decorator factory-t ParamSpec és visszatérési TypeVar segítségével. A külső events stringlista; a sync wrapper hívás előtt `start`, normál visszatérés után `success`, `Exception` esetén `failure` elemet tesz bele. Az eredményt és az exception objektumot változatlanul továbbadja.

- [ ] Positional és keyword-only paraméteres függvény típusa megmarad.
- [ ] Mypy strict alatt nincs Any vagy ignore a wrapperben.
- [ ] A név és docstring megmarad, siker és hiba eseménysorrendje tesztelt.
- [ ] Egy szándékosan hibás argumentumú hívást külön negatív fájlban a checker elutasít.

**Interview:** Why are functools.wraps and ParamSpec both needed here?
