# Adatszerkezetek – önálló feladatok

Állapot: a feladatkiírások elkészültek; megoldás és review még nincs. Csak standard library szükséges. Kész megoldás nincs mellékelve. A megoldások és tesztek később a `02-data-structures/` almappába kerülhetnek.

[Tananyag](../../knowledge/senior-python/02-data-structures/README.md)

## D01

**Cél:** szekvencia és bemenetmegőrzés.

Írj `without_status(records, excluded)` függvényt. A records `(id: str, status: str)` tuple-ök listája. Új listát adj azokról, amelyek státusza nem excluded. A bemeneti sorrend és az ismétlődő rekordok maradjanak meg, a bemenet ne módosuljon.

Példa: `[('a', 'ok'), ('b', 'bad'), ('c', 'bad')]`, `'bad'` → `[('a', 'ok')]`.

- [ ] Üres, minden elemet kizáró, semmit sem kizáró és egymás melletti kizárt rekordok tesztelve.
- [ ] A bemenet változatlan; nincs iterálás közbeni törlés.

**Interview:** Is a new outer list sufficient here, and why?

## D02

**Cél:** dict-index és explicit duplikációs szabály.

Írj `index_records(records)` függvényt. A bemenet egyszer bejárható dict-iterable; minden rekordnak kötelező nem üres string `id` mezője van, további mezők tetszőlegesek. Az eredmény az ID-ket az eredeti rekordobjektumokra képezi. Duplikált ID, hiányzó, nem string vagy üres ID esetén `ValueError` kell. A bemeneti rekordok nem módosulhatnak.

Példa: `[{'id': 'a', 'status': 'ok'}]` → `{'a': {'id': 'a', 'status': 'ok'}}`.

- [ ] Első és késői duplikáció is hibázik, nem ír felül csendben.
- [ ] Üres input üres dictet ad.
- [ ] A rekordok referencia-megosztását dokumentálod.

**Interview:** How would the contract change if callers needed independent snapshots?

## D03

**Cél:** stabil prioritási sorrend.

Írj `schedule(jobs)` függvényt. A jobs `(priority: int, payload: dict)` párok iterable-je. Az eredmény új payload-lista, növekvő prioritás szerint; holtversenyben beküldési sorrend. A payload dictnek nincs rendezési metódusa, és nem módosítható. A feladatban használj heapet.

Példa: prioritások `[2, 1, 1]`, ID-k `['a', 'b', 'c']` → `['b', 'c', 'a']`.

- [ ] Azonos prioritású eltérő dict payloadok nem okoznak összehasonlítási hibát.
- [ ] Negatív prioritás és üres input támogatott.
- [ ] Elmondod, mikor lenne egyszerűbb sima rendezést használni, és mikor előny a fokozatos queue.

**Interview:** What is the cost of inserting one additional job?

## D04

**Cél:** komplexitás és mérés.

Írj két változatot `allowed_events(events, allowed)` néven külön modulokban vagy eltérő függvénynévvel: listás tagságvizsgálattal és egyszer előállított set-indexszel. Mindkét input stringlista; az eredmény events eredeti sorrendjében tartalmazza az engedélyezett elemeket, duplikációkkal együtt.

- [ ] Mindkét implementáció azonos eredményt ad üres és duplikált bemenetekre is.
- [ ] Leírod az idő- és extra memóriaigényt n eseményre és m engedélyezett névre.
- [ ] Legalább két bemeneti mérettel mérsz, több ismétléssel; a setépítés idejét is számolod.
- [ ] Nem kötsz a tesztbe gépfüggő, kötelező gyorsulási arányt.

**Interview:** When might building the index not pay off?

## D05

**Cél:** számlálás és determinisztikus rendezés.

Írj `error_ranking(events)` függvényt. A bemenet `(service: str, status: int)` párok iterable-je; minden service nem üres, status 100–599 közötti. Csak az 500–599 értékű rekord hibás. Az eredmény `(service, error_count)` párok listája: csökkenő hibaszám, holtversenynél service szerinti növekvő, kis- és nagybetűt megkülönböztető rendezés. Nulla hibás szolgáltatás ne kerüljön bele.

Példa: `[('b', 500), ('a', 503), ('b', 200)]` → `[('a', 1), ('b', 1)]`.

- [ ] Üres input és kizárólag sikeres események üres listát adnak.
- [ ] A bemeneti sorrend megváltoztatása nem változtatja a kimenetet.
- [ ] A költséget n esemény és k hibás szolgáltatás függvényében megadod.

**Interview:** Why is stable sorting alone insufficient for this output contract?

## D06

**Cél:** egyszeri bejárás és korlátos állapot.

Írj `summarize_levels(lines)` függvényt. Minden sor `LEVEL|message` formájú string; az első `|` választ el. Engedélyezett szintek pontosan `INFO`, `WARN`, `ERROR`. Üres üzenet és további `|` az üzenetben megengedett; a szintet ne normalizáld. Hiányzó elválasztó vagy ismeretlen szint hibás rekord.

Az eredmény `{'counts': {'INFO': i, 'WARN': w, 'ERROR': e}, 'invalid': n}`; mindhárom számláló mindig szerepel. A sorok tartalmát ne gyűjtsd össze, csak a számlálókat tartsd meg. I/O-hibát ne minősíts hibás rekordnak, engedd tovább.

Példa: `['INFO|start', 'ERROR|', 'bad', 'DEBUG|x']` → counts `1, 0, 1`, invalid `2`.

- [ ] Egyszer fogyasztható bemenettel és üres inputtal is működik.
- [ ] Nagy generált bemenetet dolgoz fel teljes listába gyűjtés nélkül.
- [ ] Megadod a memóriaigényt, figyelembe véve az aktuális sor hosszát is.
- [ ] Elmagyarázod, hogyan változna a memóriaigény, ha szint helyett minden egyedi message-re számolnál.

**Interview:** What would you do if a single line could exceed available memory?
