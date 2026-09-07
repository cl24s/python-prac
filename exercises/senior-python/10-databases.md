# Adatbázisok és adatkezelés – önálló feladatok

A feladatokhoz kész megoldás nincs mellékelve. A kód és tesztek később saját almappába kerüljenek; a leírás és a review Markdown legyen. Minden megoldásnál rögzítsd a futtatott parancsot, eredményt és a nem ellenőrzött környezetet.

## DB01

### Tulajdonosonkénti jobstatisztika

**Cél és szerződés:** Készíts owners és jobs sémát. Minden ownerhez add vissza a queued jobok számát, a nullás ownerekkel együtt. Paraméterezett státuszszűrést használj.

**Elfogadási feltételek:**

- Legalább három owner: több queued jobbal, csak done jobbal, job nélkül.
- Idegen kulcs, kötelező mező és status constraint védje a sémát; legalább egy megsértésükre legyen teszt.
- Írd le az ON/WHERE különbséget és a COUNT(*) buktatóját.

**Korlát és review:** SQLite-nál kapcsold be a foreign_keys ellenőrzést. A tesztadat ne csak egy szülő/egy gyermek eset legyen.

## DB02

### Lekérdezés és indexterv

**Cél és szerződés:** Készíts tenant, status, created_at, id mezős jobtáblát és tenant/status szerint szűrt, created_at/id szerint rendezett, limitált lekérdezést.

**Elfogadási feltételek:**

- Legalább 1000 determinisztikus tesztsor és több tenant legyen.
- Index nélkül és indexszel rögzítsd a tervet egy plan.md-ben; az eredmény azonos maradjon.
- Indokold az index oszlopsorrendjét és az írási költséget; ne állíts sebességarányt mérés nélkül.

**Korlát és review:** Nem kell PostgreSQL-szerver. A terv szövege motor- és verziófüggő; ne legyen általános API-szerződés.

## DB03

### Készlet és foglalás egy tranzakcióban

**Cél és szerződés:** Egy pozitív egész quantity foglalása csökkentse a termék készletét és hozzon létre reservation sort. Hiányzó termék, elégtelen készlet vagy második írás hibája esetén ne maradjon részleges módosítás.

**Elfogadási feltételek:**

- Siker esetén mindkét változás új kapcsolatból látszik.
- Injektálj hibát a készletcsökkentés után, a reservation írása előtt; az eredeti készlet maradjon meg.
- Nulla és negatív mennyiség elutasított; készlet nem lehet negatív.

**Korlát és review:** A tranzakció tulajdonosa egyetlen felhasználási eset legyen. Használj valós ideiglenes DB-fájlt; külső hálózati hívás nem szükséges.

## DB04

### Optimista állapotváltás

**Cél és szerződés:** A job status és version mezőt tárol. change_status(id, expected_version, new_status) egy SQL-utasítással módosítson és növelje a verziót. Elavult/hiányzó rekord Conflict hibát ad.

**Elfogadási feltételek:**

- Két ugyanabból a verzióból induló írásból csak az első sikeres.
- A második nem írja felül az első eredményét.
- Írj külön tervet PostgreSQL FOR UPDATE megoldásra, deadlock/serialization failure és retry esetére.

**Korlát és review:** A kontrollált stale-read teszt nem egyidejű threadteszt. A PostgreSQL-terv ne legyen végrehajtottként jelölve.

## DB05

### N+1 kimutatása és javítása

**Cél és szerződés:** Készíts SQLAlchemy owner–job kapcsolatot legalább 4 ownerrel, ownerenként eltérő jobszámmal. Mérd a lazy és egy választott eager stratégia SELECT-jeit.

**Elfogadási feltételek:**

- Mindkét mérés friss sessionben induljon; a kapott domainadat azonos.
- Az eredeti 1+N queryszám és a javított szám rögzítve legyen.
- Készíts session bezárása után is használható publikus DTO-kat, rejtett SQL nélkül.

**Korlát és review:** Nem kell benchmarkküszöb. Az identity map ne fedje el a rossz betöltési stratégiát.

## DB06

### Kompatibilis priority migráció

**Cél és szerződés:** A régi jobs(id,name) sémához priority mezőt kell hozzáadni. Régi és új alkalmazás átmenetileg együtt fut. Készíts migration-plan.md-t és SQLite expand/backfill demót.

**Elfogadási feltételek:**

- A régi olvasó a bővítés után is működik.
- A backfill ismétlése nem ront adatot; egy késői régi író beszúrását is kezeld a tervben.
- A NOT NULL/contract feltétel, batch-elés, ellenőrző query és rollback/forward-fix döntés legyen leírva.

**Korlát és review:** A példa nem helyettesít Alembic-review-t vagy nagy táblás locktesztet. Destructive downgrade esetén nevezd meg az adatvesztést.

## DB07

### Cache-kulcs, TTL és invalidálási race

**Cél és szerződés:** Készíts kis cache-modellt befecskendezett monoton órával és (tenant,id) kulccsal. A nem található értéket egyértelmű miss jelölje; döntsd el, hogy None tárolható-e.

**Elfogadási feltételek:**

- Teszteld a lejárat előtti, pontos és utáni olvasást sleep nélkül.
- Tenantok között nincs kulcsütközés; invalidálás ismételhető.
- Egy külön consistency.md vezesse le a régi olvasás → új DB-commit → invalidálás → régi visszatöltés sorrendet, és válasszon indokolt védekezést.

**Korlát és review:** Nem kell Redis-szerver. Írd le a memóriakorlát, stampede és cache-kiesés további kockázatait; a helyi dict nem elosztott koordináció.

[Tananyag](../../knowledge/senior-python/10-databases/README.md) · [Gyakorlatok](../README.md)
