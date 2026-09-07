# Csomagolás, biztonság és üzemeltetés – önálló feladatok

Állapot: kidolgozott kiírások, megoldás és teljesítés nélkül. A kód később saját feladatkönyvtárba kerül; minden leírás Markdown. A tervezési feladatoknál a dokumentum és az érvelés a leadandó eredmény, nem szükséges mesterségesen kódot írni.

## OP01

### Telepíthető CLI és artifact-ellenőrzés

**Cél:** Egy kis programot a forrásfától függetlenül telepíthető csomaggá alakítani.

**Bemenet:** Saját, egyszerű status CLI: argumentum nélkül írjon ki egy fix státuszt és lépjen ki 0-val; ismeretlen opcióra nem nulla exit code kell. A státusz tartalma tetszőleges, de dokumentált.

**Kimenet:** pyproject.toml, src elrendezés, tesztek, valamint README.md pontos build/telepítés/futtatás paranccsal és verziókkal. A csomagnév legyen helyi gyakorlónév; nem kell publikálni.

**Korlátok:** Nincs PyPI-feltöltés vagy éles deploy. A forráskód saját formátumában, minden dokumentáció MD-ben legyen. Előbb válassz és rögzíts build backendet.

**Elfogadási feltételek:**

- [ ] Wheel és sdist épül; az sdistből is létrehozható wheel.
- [ ] Új venvben a wheelből telepített import és CLI a checkouton kívül is működik.
- [ ] A README külön kezeli a fejlesztői és runtime függőségeket, és leírja a tranzitív rögzítést.
- [ ] Rögzíted a Python-, platform- és buildeszköz-verziót, és jelzed, ha a build nem bitazonos.

[Kapcsolódó fejezet](../../knowledge/senior-python/13-packaging-security-and-operations/01-packaging-and-dependencies.md)

## OP02

### Validált settings és rotációs terv

**Cél:** Tesztelhető konfigurációs beolvasást készíteni globális környezeti állapot nélkül.

**Bemenet:** String mapping: API_KEY kötelező nem üres; TIMEOUT_SECONDS alapérték 2, véges szám (0,30]; MAX_JOBS egész 1–1000, alapérték 20; DEBUG kizárólag true/false, alapérték false.

**Kimenet:** Típusos settings objektum vagy mezőnevet megadó ValueError; egy MD rotációs terv a már élő klienskapcsolatokra.

**Korlátok:** Kizárólag mesterséges secret. A hiba nem idézheti a kapott értéket. Ne használj bool(string) konverziót. A secret megjelenítése ne szerepeljen reprben.

**Elfogadási feltételek:**

- [ ] Default és explicit értékek helyesek; true/false külön tesztelt.
- [ ] Hiányzó/whitespace secret, nan/inf, határértékek és hibás integer elutasítva.
- [ ] Sikeres és hibás utak kimenetében sincs benne a tesztsecret.
- [ ] Leírod, mely serializer képes a repr-védelmet megkerülni, és hogyan újulnak meg a kapcsolatok rotációkor.

[Kapcsolódó fejezet](../../knowledge/senior-python/13-packaging-security-and-operations/02-configuration-and-secrets.md)

## OP03

### Megfigyelhetőségi szerződés

**Cél:** Egy későbbi job API hibáját követhetővé tenni túlzott adatgyűjtés nélkül.

**Bemenet:** Három szintetikus esemény: sikeres POST /jobs, sikertelen DB-művelet, worker retry. Mindegyikhez saját, nem érzékeny korrelációs azonosító.

**Kimenet:** JSON eseményséma, mintakimenetek és MD dashboard/probe terv; a serializerhez unit tesztek.

**Korlátok:** Nincs szükség collector vagy Prometheus telepítésre. A request body, token és nyers URL nem lehet logmező. ID nem lehet metric label.

**Elfogadási feltételek:**

- [ ] A log egy fizikai sor, típusos duration mezővel, eseménynévvel és korrelációval; sortöréses input tesztelt.
- [ ] A metrikákhoz útvonalsablon és bounded dimenziók vannak; a cardinality indokolt.
- [ ] Startup/readiness/liveness külön jelentést és olcsó ellenőrzési tervet kap.
- [ ] A DB-kiesés és opcionális függőséghiba reakciója indokolt; van legalább egy felhasználói hatáshoz kötött riasztás.

[Kapcsolódó fejezet](../../knowledge/senior-python/13-packaging-security-and-operations/03-observability-and-health.md)

## OP04

### CI és release terv

**Cél:** Bizonyítható kapcsolatot teremteni commit, teszteredmény és kiadott artifact között.

**Bemenet:** Az OP01 csomagja; ha még nincs kész, először annak implementálása szükséges. A feladat két fázisú: terv, majd helyi ellenőrzés.

**Kimenet:** MD pipeline-terv és a helyileg futtatott parancsok eredménye; külön PR- és release-jogosultsági táblázat.

**Korlátok:** Workflow bekapcsolása és publikálás nem része a feladatnak. Nem futtatott kaput ne jelölj sikeresnek. A használt lint/type/build eszköz verzióját rögzítsd.

**Elfogadási feltételek:**

- [ ] Lint, type check, unit teszt, build és artifact smoke gate célja és hibakimenete leírt.
- [ ] Az artifactot tiszta környezetben vizsgálod és digesttel azonosítod.
- [ ] A cache nélküli út és a nem megbízható PR kezelése szerepel.
- [ ] A release ugyanazt az artifactot vinné tovább; nincs szükség éles secretre a PR-tesztekhez.

[Kapcsolódó fejezet](../../knowledge/senior-python/13-packaging-security-and-operations/04-ci-and-artifact-promotion.md)

## OP05

### Deployment-kapacitás és leállítás

**Cél:** A worker- és replikaszámot összekötni a DB és memória korlátaival.

**Bemenet:** Feltételezett API: 4 replika, rollout surge 1, 3 worker/pod, pool 4 és overflow 1/worker. Külön háttérworker maximum 10 DB-kapcsolatot használ, operációs tartalék 15; DB-budget összesen 100. Memóriabecsléshez 120 MiB/worker plusz 80 MiB/pod overhead, limit 512 MiB/pod.

**Kimenet:** Tiszta számolófüggvény tesztekkel és MD kapacitás/rollout/leállítási terv.

**Korlátok:** A számok oktatási feltételezések, nem mérések. A plusz terminating podot külön érzékenységi esetként vizsgáld. Ne telepíts clustert.

**Elfogadási feltételek:**

- [ ] Normál és surge állapotban a teljes kapcsolatigényt kiszámolod a külön workerrel és tartalékkal.
- [ ] Megállapítod, milyen tartalék marad, és mi változik egy további terminating podnál.
- [ ] A memóriamodellből következtetsz, de jelzed a mért RSS és peak hiányát.
- [ ] SIGTERM utáni sorrend, maximális drain idő és SIGKILL utáni recovery külön szerepel.

[Kapcsolódó fejezet](../../knowledge/senior-python/13-packaging-security-and-operations/05-containers-and-capacity.md)

## OP06

### Inputbiztonsági review és javítás

**Cél:** Az adat/utasítás határt több végrehajtási felületen védeni.

**Bemenet:** Készíts kisméretű helyi keresőfüggvényt SQLite-tal: name érték és sort választás (name vagy created). Emellett tervezz fix executable-t hívó adaptert, amelynek csak egy adatargumentuma változhat.

**Kimenet:** Paraméterezett keresés tesztekkel; subprocess-adapter terve vagy veszélytelen helyi echo-szerű tesztje; MD deszerializációs döntés.

**Korlátok:** Nem kell sérülékeny parancsot futtatni vagy pickle payloadot készíteni. Ne használd evalt, shell=True-t vagy külső célpontot.

**Elfogadási feltételek:**

- [ ] Idézőjeles SQL-input nem bővíti a találati halmazt; ismeretlen sort elutasított.
- [ ] A célprogram és opciók nem külső inputból származnak; shellkarakter adat marad.
- [ ] Argument injection, időlimit és outputméret külön tárgyalt.
- [ ] Nem megbízható pickle helyett választott formátumhoz séma- és méretkorlát szerepel.

[Kapcsolódó fejezet](../../knowledge/senior-python/13-packaging-security-and-operations/06-injection-and-deserialization.md)

## OP07

### SSRF fenyegetési modell

**Cél:** Az endpoint checker kimenő hálózati jogosultságait korlátozott szerződéssel leírni.

**Bemenet:** Mesterséges esetek: engedélyezett publikus cél; loopback; privát cím; IPv6 loopback; publikus host tiltott címre redirectel; DNS két eltérő címet ad; túlméretes válasz.

**Kimenet:** MD fenyegetési modell és fake resolver/transport tesztterv, opcionálisan ennek helyi implementációja. A biztonsági döntéseket minden esetnél indokold.

**Korlátok:** Semmilyen valódi metadata vagy belső végpontot ne kérj le. URL-regex önmagában nem fogadható el teljes védelemként. A tényleges hálózati biztonság nem bizonyítható csak fake-ekkel.

**Elfogadási feltételek:**

- [ ] A sémák, portok, userinfo és redirect szabálya explicit.
- [ ] A feloldott és tényleges kapcsolati cím összekötése, több IP és DNS-változás kezelése leírt.
- [ ] TLS hostname-ellenőrzés, egress szabály, teljes timeout és kibontott méretkorlát szerepel.
- [ ] A publikus hiba nem tartalmaz belső címet/secretet; külön lista mutatja a későbbi valódi integrációs teszteket.

[Kapcsolódó fejezet](../../knowledge/senior-python/13-packaging-security-and-operations/07-ssrf-and-data-exposure.md)

[Témakör áttekintése](../../knowledge/senior-python/13-packaging-security-and-operations/README.md) · [Gyakorlatok](../README.md)
