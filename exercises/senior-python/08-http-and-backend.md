# HTTP és backend alapok – önálló feladatok

A feladatokhoz nincs kész megoldás. A kód és tesztek később saját almappába kerüljenek; a dokumentáció maradjon Markdown. A feladat végén írd le, mely szerződést ellenőrizted, mi futott és mi nem. A tananyag elkészülte nem jelent feladatteljesítést.

## H01

### Job API szerződésterve

**Cél:** Metódust, státuszt és retry-viselkedést összehangolni.

**Bemenet, kimenet és viselkedés:** Készíts api-contract.md-t job létrehozására, állapotolvasására és törlésére. A job erőforrás és a háttérben futó munka életciklusa külön fogalom. A létrehozásnak legyen kliens által megadott idempotency key-je.

**Elfogadási feltételek:**

- Minden művelethez útvonal, metódus, sikeres státusz, releváns header és hibaforgatókönyv tartozik.
- Indokold a 201 vagy 202 választását; 202 esetén definiáld, hogyan lehet később eredményt kérni.
- Írd le az azonos kulcs/azonos body, azonos kulcs/eltérő body és két egyidejű kérés viselkedését.
- Szerepeljen kliens-timeout a szerveroldali mentés után, valamint kulcslejárat és újraindítás esete.

**Korlát és review:** Most tervezési dokumentum kell, nem FastAPI-implementáció. Az idempotenciát ne állítsd tartósnak egy processzmemóriás dict alapján.

## H02

### Feltételes olvasás és írás

**Cél:** ETag, cache és atomikus verzióellenőrzés használatát megtervezni.

**Bemenet, kimenet és viselkedés:** A /jobs/42 erőforrásnak erős, opaque ETagje van. Készíts request/response példákat Markdownban, majd opcionálisan in-memory modellt. A központi feladat a szerződés és az ellenőrzési terv.

**Elfogadási feltételek:**

- Szerepeljen feltétel nélküli 200-as GET és egyező If-None-Match esetén 304.
- Íráskor egyező If-Match sikeres módosítást és új verziót, eltérő érték 412-t eredményezzen.
- Definiáld a hiányzó If-Match viselkedését, és indokold a döntést.
- Írd le két versengő írás tesztjét: ugyanabból a verzióból legfeljebb egy nyerhet.
- Indokold, milyen Cache-Control kell a személyre szabott válaszhoz.

**Korlát és review:** A modellezett ellenőrzés nem bizonyít adatbázis-atomikusságot. Ne készíts általános HTTP-header parsert; az éles implementáció könyvtári parserre támaszkodjon.

## H03

### Egységes Problem Details fordítás

**Cél:** Belső hibából stabil, érzékeny adatot nem kiszivárogtató választ adni.

**Bemenet, kimenet és viselkedés:** Írj frameworkfüggetlen error-mapper függvényt. Bemenete saját InvalidInput, MissingJob, JobConflict vagy ismeretlen Exception és ellenőrzött request ID; kimenete HTTP-státusz, headerek és JSON-szerializálható objektum. A státuszok rendre 400, 404, 409, 500.

**Elfogadási feltételek:**

- Minden kategóriához legyen teszt a státuszra és application/problem+json Content-Type-ra.
- A body status mezője egyezzen a HTTP-státusszal, a request ID maradjon meg.
- Ismeretlen exception érzékenynek jelölt szövege ne kerüljön a bodyba.
- A kliens gépi feldolgozásához legyen dokumentált stabil hibatípus vagy kód; a detail szövege ne legyen azonosító.

**Korlát és review:** Nem kell még HTTP-framework. A típus URI-ját vagy saját extension meződet dokumentáld; ne állítsd, hogy a mapper naplóz vagy streaming hibát is megold.

## H04

### WSGI és ASGI teszthatár

**Cél:** Alkalmazásinterfészt működtetni hálózati szerver nélkül.

**Bemenet, kimenet és viselkedés:** Készíts egyszerű WSGI és ASGI alkalmazást. Mindkettő GET /health kérésre 200 és {"status":"ok"}, ismeretlen útvonalra 404, /health útvonalon más metódusra 405 és Allow: GET választ adjon. A demó szerződésében a HEAD kezelése is 405; ez tudatosan szűk tanpélda.

**Elfogadási feltételek:**

- HTTPX WSGITransport/ASGITransport segítségével mindhárom ágat ellenőrizd mindkét appon.
- A JSON Content-Type és a válaszbody egyezzen a szerződéssel.
- A kliens életciklusa lezáruljon, ne kelljen fix port vagy külső szolgáltatás.
- Írj rövid leírást arról, mi nem lett tesztelve: TCP, TLS, reverse proxy és lifespan.

**Korlát és review:** Nem kell saját általános router vagy teljes ASGI-szerver. Nem HTTP ASGI scope-nál legyen explicit korlát; a lifespan ne tűnjön ellenőrzöttnek.

## H05

### Tenant-határos jogosultsági mátrix

**Cél:** Objektumszintű jogosultságot külön policyként tesztelni.

**Bemenet, kimenet és viselkedés:** Bemenet: már hitelesített principal (user_id, tenant_id, roles), job (owner_id, tenant_id) és művelet (read vagy cancel). Read: tulajdonos vagy azonos tenant adminja. Cancel: csak azonos tenant adminja. Ismeretlen művelet tiltott. Más tenant mindig tiltott.

**Elfogadási feltételek:**

- Parametrizált tesztmátrix ellenőrizze a tulajdonost, idegen felhasználót, helyi admint és más tenant adminját mindkét műveletnél.
- Ismeretlen műveletre false az eredmény; a policy nem módosít bemenetet.
- Írd le, honnan származik a megbízható principal, és miért nem fogadható el role a request bodyból.
- Külön tervezd meg a listaendpoint jogosultsági szűrésének helyét a lapozáshoz képest.

**Korlát és review:** Nem kell JWT-aláírás vagy jelszókezelés. A policy tesztje nem bizonyít tokenhitelesítést.

## H06

### Keyset lapozás változó adatokkal

**Cél:** Egyedi rendezési kulccsal, dokumentált konzisztenciával lapozni.

**Bemenet, kimenet és viselkedés:** Írj függvényt egyedi id-jú, created_at egész időbélyegű rekordok lapozására növekvő (created_at, id) szerint. Bemenet rekordlista, opcionális utolsó kulcs és 1–100 közötti limit; kimenet elemek és next_cursor vagy None. A cursor ebben a modellben belső tuple.

**Elfogadási feltételek:**

- Azonos időbélyegű rekordok több oldalon sem veszhetnek el és nem ismétlődhetnek változatlan bemenetnél.
- Üres lista, egy teljes oldal és több oldal is tesztelt; 0 és 101 limit ValueErrort ad.
- Mutass példát két oldal közti beszúrásra a cursor elé és mögé, és dokumentáld az eredményt.
- Készíts rövid kompatibilitási elemzést új enumérték és új kötelező mező bevezetéséről.

**Korlát és review:** A teljes listát rendező modell elfogadható, de ne állítsd nagy adatra optimálisnak vagy snapshot-konzisztensnek. Külső opaque cursor kódolását nem kell most implementálni.

## H07

### Külső HTTP-függőség tesztelhető adaptere

**Cél:** HTTP- és transporthibát megkülönböztetni, indokolatlan retry nélkül.

**Bemenet, kimenet és viselkedés:** Írj fetch_job(client, id) adaptert befecskendezett HTTPX klienssel. 200 és megfelelő JSON objektum esetén adja a job adatát; 404-re saját JobNotFound, 503-ra UpstreamUnavailable, timeout esetén UpstreamTimeout, hibás JSON/séma esetén InvalidPayload hibát adjon. A séma: id pozitív egész (bool nem elfogadott), status queued vagy done.

**Elfogadási feltételek:**

- MockTransporttal teszteld a sikert, 404-et, 503-at, injektált ReadTimeoutot, hibás JSON-t és hibás sémát.
- A releváns eredeti exception cause-ként megmarad; a request útvonalát is ellenőrizd.
- A függvény nem zárja le a kívülről kapott klienst, és nem végez automatikus retryt.
- Dokumentálj connect/read/write/pool beállítást és egy két downstream hívást tartalmazó teljes deadline-tervet.
- Írd le külön, hogy a mock nem mér valódi timeoutot és pool-kimerülést.

**Korlát és review:** A fent nem definiált HTTP-hibastátuszokra HTTPStatusError terjedjen tovább, egyéb transporthibák is maradjanak láthatók. Nem kell élő szolgáltatás, titok vagy hálózati terheléspróba.

[Kapcsolódó tananyag](../../knowledge/senior-python/08-http-and-backend/README.md) · [Gyakorlatok áttekintése](../README.md)
