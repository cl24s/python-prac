# FastAPI – önálló feladatok

A feladatokhoz kész megoldás nincs mellékelve. A kód és tesztek később saját almappába kerüljenek; a leírás és a review Markdown legyen. Minden megoldásnál rögzítsd a futtatott parancsot, eredményt és a nem ellenőrzött környezetet.

## F01

### Joblista és OpenAPI

**Cél és szerződés:** Készíts GET /jobs endpointot APIRouterben. A limit egész, 1–50, alapértéke 10; a response_model csak id és name mezőt adjon. A forrás egy befecskendezett, sorrendben rendezett fake lista.

**Elfogadási feltételek:**

- Hiányzó limitnél legfeljebb 10, limit=2 esetén az első két elemet adja vissza.
- 0, 51 és nem számszerű limit 422-t ad; belső mező nem kerül a válaszba.
- Az OpenAPI-ban stabil operation_id, limit és válaszséma szerepel; a tényleges választ is teszteld.

**Korlát és review:** Nem kell valódi tároló vagy hálózati szerver. A paraméter validációját ne külön kézi queryparser oldja meg.

## F02

### Create és PATCH modellek

**Cél és szerződés:** A CreateJob name mezője trim után 1–80 karakter, attempts strict egész 1–5. JobPatch description mezőjén a hiányzó érték nem változtat, null töröl, szöveg felülír. Extra mező tiltott.

**Elfogadási feltételek:**

- Whitespace név, bool és szöveges attempts elutasítása tesztelt.
- A PATCH három állapota külön tesztet kap; a forrásobjektum nem módosul véletlenül.
- A publikus válaszmodell nem tartalmaz tenant vagy belső token mezőt.

**Korlát és review:** A validátor ne végezzen adatbázis-I/O-t; a PATCH csak descriptiont engedjen, ne tetszőleges attribútumot.

## F03

### Dependency-életciklus

**Cél és szerződés:** Készíts yield dependencyt, amely nyitás/zárás eseményt rögzít, és egy requesten belül két dependency-fogyasztó osztozik rajta. Az endpoint opcionálisan 409-es hibát dob.

**Elfogadási feltételek:**

- Egy kérés egy nyitást és egy zárást eredményez, siker és hiba esetén is.
- Két külön kérés külön erőforrást kap.
- Dokumentáld a scope választását és azt, mi változna, ha a válaszstream olvasná az erőforrást.

**Korlát és review:** A hiba ne nyelődjön el. Az eseménylista tesztsegéd, nem konkurens éles naplózó.

## F04

### Blokkoló SDK bekötése

**Cél és szerződés:** Tervezz és valósíts meg endpointot egy sync, befecskendezett SDK-művelethez. A demóban a művelet threadazonosítót ad vissza. Válassz sync endpointot vagy explicit thread-offloadot.

**Elfogadási feltételek:**

- A teszt bizonyítsa, hogy a blokkoló művelet nem az async event loop threadjén fut; ne időalapú sleep-teszttel igazold.
- Az SDK hibája látható marad vagy dokumentált domainhibává alakul.
- Írd le, milyen korlát és timeout kellene valódi hívásnál, illetve mi történik request cancellation után.

**Korlát és review:** CPU-párhuzamosítás és valódi SDK nem kell. A threadhez tartozó futást ne keverd a processz izolációjával.

## F05

### Lifespan-erőforrás

**Cél és szerződés:** App factory hozzon létre alkalmazást, lifespanben egy AsyncClienttel. GET /dependency egy kontrollált mock transportot hívjon. A kliens az apphoz tartozik, nem kérésenként jön létre.

**Elfogadási feltételek:**

- Két kérés ugyanazt a nyitott klienst használja.
- TestClient contextből kilépve a kliens lezárult.
- Két külön app tesztje nem osztozik véletlenül ugyanazon kliensen.

**Korlát és review:** A mock nem igazol hálózati poolt. Ne nyiss kapcsolatot modulimportkor.

## F06

### Domainhiba és kérésazonosító

**Cél és szerződés:** Készíts saját JobConflict kivételt, 409-es Problem Details handlert és kérésenkénti request ID-t. A body és a header ugyanazt az azonosítót adja vissza.

**Elfogadási feltételek:**

- A publikus detail stabil, az eredeti belső hibaüzenet nem jelenik meg.
- Két külön kérés eltérő request ID-t kap.
- Írd le, hogy a kezeletlen 500 és a már elindult streaming válasz külön tesztet kíván.

**Korlát és review:** Ne map-elj minden ValueErrort klienshibára. Közvetlen JSONResponse esetén explicit teszteld a body szerződését.

## F07

### Auth és policy tesztmátrix

**Cél és szerződés:** Befecskendezett tesztverifier hitelesítse a bearer tokent. A jobot a tulajdonos vagy azonos tenant adminja olvashatja; más tenant tiltott.

**Elfogadási feltételek:**

- Hiányzó és érvénytelen token 401 + WWW-Authenticate; jogosulatlan principal 403.
- Tulajdonos, helyi admin és más tenant adminja külön teszteset.
- Legalább egy HTTP-teszt ne írja felül a teljes auth dependencyt, így a credential bekötése is fut.

**Korlát és review:** Nem kell saját JWT-kriptográfia. A README írja le a valódi verifier szükséges claim- és kulcsellenőrzéseit, és hogy ezek itt még nem futnak.

## F08

### Egyedi jobnév és rollback

**Cél és szerződés:** POST /jobs hozza létre a jobot SQLite + SQLAlchemy tárolóban, kérésenkénti sessionnel. A név nem üres, egyedi; siker 201, duplikáció 409. A válasz id és name mezőt ad.

**Elfogadási feltételek:**

- Új sessionből visszaolvasható a sikeres mentés.
- Duplikáció nem hoz létre új sort; utána másik érvényes kérés sikeres.
- Hibás bemenet 422, az adatbázis változatlan; a sessionök és engine lezárulnak.

**Korlát és review:** Teszthez tmp_path alatti fájlt használj. Ne tekintsd a create_all-t éles migrációnak vagy a SQLite-t PostgreSQL-izolációs tesztnek.

## F09

### Háttérmunka garanciái

**Cél és szerződés:** Készíts nem kritikus háttérértesítést BackgroundTasks-szal, és mellé architecture.md-t egy kritikus job tartós queue-ba küldéséről.

**Elfogadási feltételek:**

- A helyi callback megkapja a job ID-t; nincs request-session átadva.
- A dokumentum kitér a DB-commit előtti és utáni processzhibára, valamint az outbox relay ismételt publikálására.
- Rögzítsd, mit jelent a 202 és hogyan kérdezhető le a kritikus job állapota.

**Korlát és review:** Nem kell queue-szervert építeni. A callback tesztje nem tartóssági bizonyíték.

## F10

### HTTP + fake és valódi DB külön tesztje

**Cél és szerződés:** A F08 API-hoz készíts két teszthatárt: dependency override-dal vezérelt service és valódi ideiglenes SQLite-adapter.

**Elfogadási feltételek:**

- Az override sikeres és konfliktusos választ is elő tud idézni; finally-ben visszaáll.
- A valódi DB-teszt commitot és duplikáció utáni állapotot ellenőriz.
- Rögzítsd, mely rétegek voltak helyettesítve, és ne jelölj auth/deploy tesztet sikeresnek, ha nem futott.

**Korlát és review:** Külön appokkal kerüld a megosztott állapotot. Ne pusztán a státuszkódot ellenőrizd.

## F11

### Üzemeltetési terv

**Cél és szerződés:** Írj operations.md-t a job API-hoz: 3 replika, replikánként 2 worker, workerenként 5 alap DB-kapcsolat és 2 overflow. DB teljes kerete 60, ebből 12 más szolgáltatásoké.

**Elfogadási feltételek:**

- Számold ki a saját felső keretet és a fennmaradó tartalékot.
- Adj liveness/readiness/startup szerződést, request-deadline és graceful shutdown sorrendet.
- Nevezd meg a latency, pool-várakozás és hibaarány mérését, valamint a processzösszeomláskor elveszhető munkát.

**Korlát és review:** Ez terv, nem éles telepítés. A keep-alive timeoutot ne nevezd teljes request-deadline-nak.

[Tananyag](../../knowledge/senior-python/09-fastapi/README.md) · [Gyakorlatok](../README.md)
