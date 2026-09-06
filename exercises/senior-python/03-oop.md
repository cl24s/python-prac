# OOP és objektumtervezés – önálló feladatok

Állapot: feladatkiírás kész; megoldás és review még nincs. Kész megoldás nincs mellékelve. A megoldások és tesztek később a `03-oop/` almappába kerülhetnek. Külső szolgáltatás nem szükséges.

[Tananyag](../../knowledge/senior-python/03-oop/README.md)

## O01

**Cél:** példányállapot és ownership.

Készíts `RecentEvents(limit)` osztályt `add(event)` és `snapshot()` művelettel. A limit pozitív int, bool nem fogadható el. Az event string. A tároló az utolsó limit eseményt őrzi beküldési sorrendben; a snapshot új listát ad. Példa: limit 2, majd a/b/c → snapshot b/c.

- [ ] Két példány állapota különálló; a snapshot módosítása nem hat a tárolóra.
- [ ] Nem int limit `TypeError`, nem pozitív limit `ValueError`.
- [ ] Üres állapot és a limit pontos elérése tesztelt.
- [ ] Indoklod, miért osztály és milyen belső konténer illik hozzá.

**Interview:** Which object owns the event collection?

## O02

**Cél:** invariáns és sikertelen állapotváltás.

Írj `WorkerConfig(min_workers, max_workers)` osztályt olvasható property-kkel és `resize(new_min, new_max)` metódussal. Az értékek int-ek, bool nélkül; `0 <= min <= max`. A resize mindkét új értéket együtt ellenőrzi, és hiba esetén egyik régi mező sem változhat. Nem int `TypeError`, tartományhiba `ValueError`.

- [ ] Érvénytelen konstrukció elutasított.
- [ ] Sikertelen resize után mindkét régi érték megmarad.
- [ ] Nincs publikus független setter, amely köztes érvénytelen állapotot engedne.

**Interview:** Why is one resize method safer than two independent setters here?

## O03

**Cél:** MRO és helyettesíthetőség elemzése.

Készíts `Base`, `Logging`, `Timing`, `Service(Logging, Timing)` osztályokat. A Base `run()` eredménye `['base']`; minden mixin a saját nevét tegye a `super().run()` eredménye elé. A teljes eredmény `['logging', 'timing', 'base']` legyen. A metódus ne mutáljon megosztott állapotot.

- [ ] Dokumentáld a tényleges MRO-t és azt, mire mutat a super a Logging belsejében.
- [ ] Fordított mixinsorrendhez is add meg az elvárt eredményt.
- [ ] Rövid Markdown-válaszban mutasd be, mikor választanál inkább compositiont.

**Interview:** How could an explicit Base.run(self) call break the chain?

## O04

**Cél:** dependency injection és hibaút.

Készíts `HealthReporter` osztályt injektált `clock` és `sink` függőséggel. A clock `now() -> int`, a sink `write(message: str) -> None` műveletet biztosít. A `report(service)` pontosan egyszer kér időt és pontosan egyszer ír `timestamp:service` formátumban. Üres/whitespace service `ValueError`, még clock-hívás előtt. A service string, egyéb inputtípust most nem kell támogatni.

- [ ] Fake clockkal és recording sinkkel ellenőrizhető a tartalom és a hívásszám.
- [ ] Clock-hiba után nincs írás; sink-hiba változatlanul továbbterjed.
- [ ] Az osztály nem zárja le a kívülről kapott függőségeket.

**Interview:** Where should production dependencies be constructed?

## O05

**Cél:** értékobjektum és hash-szerződés.

Írj `ServiceKey(name, region)` értékobjektumot. Mindkét mező nem üres string; whitespace-only érték hibás. Az értékeket ne normalizáld, a kis- és nagybetű számít. Azonos mezők esetén legyen értékegyenlőség és azonos hash; normál attribútum-újrakötés ne legyen megengedett.

- [ ] Külön példányok azonos értékkel ugyanahhoz a dictkulcshoz tartoznak.
- [ ] Eltérő region külön kulcs.
- [ ] Nem string `TypeError`, üres vagy whitespace-only `ValueError`.
- [ ] Elmagyarázod, miért más a helyzet egy listát tartalmazó frozen dataclassnál.

**Interview:** Is this object an entity or a value object, and why?

## O06

**Cél:** több felelősség szétválasztása.

Tervezz és valósíts meg kis `register_service(name)` use case-t. Érvényes név: nem üres string; trimelve tárold, whitespace-only `ValueError`. A kapott repository `add(name)` duplikációnál `ValueError`-t dob; a notifier `registered(name)` értesít. Sorrend: validáció → tárolás → értesítés. A függőségek kívülről érkezzenek.

- [ ] Validációs és tárolási hiba után nincs értesítés.
- [ ] Értesítési hiba továbbterjed; a már tárolt rekord megmarad (ez a mostani explicit szerződés).
- [ ] Memóriabeli repository és fake notifier teszteli a sikert és részleges hibát.
- [ ] Rövid DESIGN.md-ben indoklod a felelősségeket és egy későbbi megbízható értesítési megoldás irányát; azt most nem kell megépíteni.

**Interview:** Why is retrying the entire operation unsafe without an idempotency policy?
