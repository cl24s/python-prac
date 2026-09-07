# Senior szintű tervezés és együttműködés – önálló feladatok

Állapot: kidolgozott kiírások, megoldás és teljesítés nélkül. A kód később saját feladatkönyvtárba kerül; minden leírás Markdown. A tervezési feladatoknál a dokumentum és az érvelés a leadandó eredmény, nem szükséges mesterségesen kódot írni.

## SD01

### Job API követelményspecifikáció

**Cél:** Egy homályos kérésből ellenőrizhető műszaki szerződést készíteni.

**Bemenet:** Kérés: több tenant hosszú jelentéseket indíthat, majd lekérdezheti az eredményt. A terhelés, retention és megengedett várakozási idő még nem ismert.

**Kimenet:** requirements.md: kérdések, tények, feltételezések, nem célok, invariánsok, SLO-terv és hibaforgatókönyv-tábla.

**Korlátok:** Nincs implementáció. A hiányzó számokat nem nevezheted valós mérésnek; a saját feltételezéseket címkézd.

**Elfogadási feltételek:**

- [ ] Legalább 8 célzott kérdés, 3 invariáns és 2 mérhető szolgáltatási céljelölt szerepel.
- [ ] Legalább 5 megszakítási pontnál megadod a maradó állapotot és retry tulajdonosát.
- [ ] A 202 jelentése és a tulajdonos tenant ellenőrzése egyértelmű.
- [ ] Az elfogadási feltételekhez megnevezed a bizonyíték típusát és a nyitott üzleti döntéseket.

[Kapcsolódó fejezet](../../knowledge/senior-python/14-design-and-collaboration/01-requirements-and-failure-scenarios.md)

## SD02

### Modulhatárok és tesztelhető use case

**Cél:** A HTTP-t és tárolást leválasztani egy kis üzleti műveletről.

**Bemenet:** Use case: saját queued job lemondható; más tenant jobja nem megfigyelhető; running vagy completed job nem mondható le. A cancelled job ismételt lemondása legyen idempotens.

**Kimenet:** MD modul- és függőségi térkép, egyszerű implementáció fake tárolóval és tesztekkel.

**Korlátok:** A példabeli read_status másolása nem elég: itt állapotváltozás van. Valódi DB nem kötelező, de a konkurens queued→running verseny kezelését külön tervezd meg.

**Elfogadási feltételek:**

- [ ] Saját queued→cancelled és ismételt cancellation sikeres.
- [ ] Idegen/hiányzó job egységes publikus kategóriát kap; running/completed elutasított.
- [ ] A use case tesztelhető FastAPI és ORM nélkül.
- [ ] A DB-adapterhez atomikus feltételes update vagy más indokolt versenykezelés terve szerepel; a fake tesztet nem állítod konkurenciabizonyítéknak.

[Kapcsolódó fejezet](../../knowledge/senior-python/14-design-and-collaboration/02-module-boundaries-and-dependencies.md)

## SD03

### Architektúraválasztás két helyzetre

**Cél:** A megoldást változó követelményhez igazítani, nem kedvenc mintához.

**Bemenet:** A eset: 3 fejlesztő, egy régió, heti közös release, mérsékelt forgalom. B eset: külön csapatok és kiadási igény, a jelentésworker sokkal nagyobb CPU-kapacitást igényel. Minden szám saját feladatfeltételezés.

**Kimenet:** architecture-options.md: moduláris monolit, közös kódbázisú API/worker és külön szolgáltatások összevetése.

**Korlátok:** Nincs implementáció. A B esetben sem automatikus a microservice választás; a külön processzskálázás lehetőségét vizsgáld.

**Elfogadási feltételek:**

- [ ] Mindkét helyzetre egyértelmű választás, legalább 2 előny és 2 vállalt költség.
- [ ] Adatgazda, releasehatár, hibamódok és operációs tulajdonos szerepel.
- [ ] Legalább 3 konkrét jelhez kötöd a későbbi kiválasztást.
- [ ] Megnevezed a legolcsóbb kísérletet, amely egy bizonytalan döntési feltételt ellenőrizne.

[Kapcsolódó fejezet](../../knowledge/senior-python/14-design-and-collaboration/03-simplicity-and-evolution.md)

## SD04

### Rövid ADR és szóbeli indoklás

**Cél:** Egyetlen technikai döntés érveit visszakereshetően rögzíteni.

**Bemenet:** Válassz a saját SD01/SD03 anyagodból egy döntést, például tartós feldolgozás vagy deploymenthatár. Használd a korábban rögzített feltételezéseket.

**Kimenet:** adr-001.md és egy 120–180 szavas angol döntésmagyarázat ugyanebben a fájlban.

**Korlátok:** A fejezet mintadöntésének átírása önmagában nem elég; saját kritérium és vállalt hátrány kell. Ne pontozz mérést nem látott teljesítményt tényként.

**Elfogadási feltételek:**

- [ ] Státusz, kontextus, legalább 2 érdemi alternatíva és következmények szerepelnek.
- [ ] Van ellenőrzési terv, felelős szerep és felülvizsgálati trigger.
- [ ] A rövid angol változat elején világos a döntés és meghatározó ok.
- [ ] Meg tudod mondani, milyen új információ esetén választanád a másik opciót.

[Kapcsolódó fejezet](../../knowledge/senior-python/14-design-and-collaboration/04-decisions-and-communication.md)

## SD05

### Kompatibilis mező- és üzenetmigráció

**Cél:** A régi és új rendszer átmeneti együttélését biztonságosan megtervezni.

**Bemenet:** V1 üzenet {version:1,id:...}, V2 {version:2,job_id:...}; régi üzenet 7 napig újrajátszható. A DB-ben legacy_id helyett job_id kell. A példák itt mezőleírások, nem szó szerinti JSON inputok.

**Kimenet:** migration-plan.md állapotmátrixszal, rollout/rollback lépésekkel, backfill és ellenőrzési tervvel; saját kompatibilitási tesztek.

**Korlátok:** Éles DB-módosítás nincs. Ne feltételezd, hogy a kódrollback visszavonja a V2 üzeneteket. A backfill újraindítható legyen.

**Elfogadási feltételek:**

- [ ] Minden producer/consumer verziópárhoz leírod az elfogadott üzenetet.
- [ ] Az olvasók frissítése megelőzi az új írást, és a 7 napos replayablak befolyásolja a contractot.
- [ ] A live írás/backfill versenyre és eltérésmérésre is van terv.
- [ ] Van megállítási feltétel, visszafordíthatatlan pont és annak helyreállítási módja.

[Kapcsolódó fejezet](../../knowledge/senior-python/14-design-and-collaboration/05-safe-change-and-migration.md)

## SD06

### Kockázatalapú review

**Cél:** A megfigyelt kódviselkedéshez kapcsolni a kritikát és segítséget.

**Bemenet:** Fiktív PR: a worker commit előtt ackol; a HTTP handler queryből kapott tenant alapján olvas; a log teljes Authorization headert ír; a függvénynév nem tetszik neked. Ezek review-esetek, nem futtatandó kód.

**Kimenet:** review.md: négy angol megjegyzés kategóriával, magyar indoklással és ellenőrzési javaslattal; rövid mentorálási terv.

**Korlátok:** Ne gyárts további hibát bizonyíték nélkül. A személy helyett a viselkedést értékeld. A megjegyzések nem kerülnek elküldésre másnak.

**Elfogadási feltételek:**

- [ ] Mindhárom működési/biztonsági kockázathoz konkrét hibaút és tesztötlet tartozik.
- [ ] A névpreferencia elkülönül a blockertől.
- [ ] Legalább két kérdés segíti a szerzőt az önálló hibafelismerésben.
- [ ] A lezáráshoz szükséges bizonyíték világos, a stíluskérdés nem rejti el a súlyos hibát.

[Kapcsolódó fejezet](../../knowledge/senior-python/14-design-and-collaboration/06-review-and-mentoring.md)

## SD07

### Három saját interjútörténet

**Cél:** Valós szakmai tapasztalatot tömören és bizonyítékkal bemutatni.

**Bemenet:** Saját incidens, teljesítményjavítás és migráció/tervezési döntés. Ha valamelyikből nincs saját tapasztalat, azt írd le; ne találj ki munkatörténetet.

**Kimenet:** interview-stories.md: történetenként rövid angol válasz és magyar részletes jegyzet; 3 lehetséges utánkérdés történetenként.

**Korlátok:** Nincs ügyfélnév, secret, belső cím vagy nyers incidenslog. Nem mért javuláshoz ne rendelj százalékot. A csapat eredményét ne tulajdonítsd teljesen magadnak.

**Elfogadási feltételek:**

- [ ] Probléma, saját szerep, alternatívák, döntés, eredmény és tanulság mind megjelenik.
- [ ] Tény, feltételezés és utólagos magyarázat elkülönül.
- [ ] A rövid változat szóban körülbelül 2 perc alatt elmondható.
- [ ] Legalább egy elvetett hipotézis vagy utólag másként választandó döntés szerepel, indoklással.

[Kapcsolódó fejezet](../../knowledge/senior-python/14-design-and-collaboration/07-interview-stories-and-learning.md)

[Témakör áttekintése](../../knowledge/senior-python/14-design-and-collaboration/README.md) · [Gyakorlatok](../README.md)
