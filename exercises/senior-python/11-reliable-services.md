# Megbízható szolgáltatások és háttérfeldolgozás – önálló feladatok

A feladatokhoz kész megoldás nincs mellékelve. A kód és tesztek később saját almappába kerüljenek; a leírás és a review Markdown legyen. Minden megoldásnál rögzítsd a futtatott parancsot, eredményt és a nem ellenőrzött környezetet.

## R01

### Közös műveleti időkeret

**Cél és szerződés:** Készíts két egymást követő függőséghívás keretét kezelő modellt injektált órával. Az összes időkeret 4 s, egy lépés legfeljebb 3 s lehet. A hívások és várakozások idejét fake-ek vezérlik.

**Elfogadási feltételek:**

- Az első lépés 3 s után visszatérve legfeljebb 1 s keretet hagy a másodiknak.
- Lejárt deadline után nem indul új hívás; ezt hívásnaplóval igazold.
- Írj failure-model.md-t arról, mit tud és nem tud a kliens egy távoli írás timeoutja után.

**Korlát és review:** A fake clock nem állítja át az event loop óráját. A modell nem garantálja a valódi driver timeoutjának betartását.

## R02

### Korlátos retry végrehajtó

**Cél és szerződés:** Implementálj retry(operation, clock, pause, jitter) függvényt. Maximum 3 összes próbálkozás; csak saját TemporaryFailure retryzható; a teljes deadline 2 s. Backoff felső határa 0,2 s, majd 0,4 s; full jitter legyen befecskendezve.

**Elfogadási feltételek:**

- Két átmeneti hiba után siker: pontosan 3 hívás és 2 várakozás.
- Az utolsó hiba után nincs további várakozás; a végső exception maradjon látható.
- Nem átmeneti hiba azonnal továbbterjed; várakozás közben elfogyó keret után nem indul új hívás.
- Dokumentáld, miért biztonságosan ismételhető a választott operation.

**Korlát és review:** Ne valódi sleepből vagy véletlenből következtess a helyességre. A retry nem általános írási biztonsági garancia.

## R03

### Tartós esemény-deduplikáció

**Cél és szerződés:** Egy SQLite-fájlban ownerenként foglalásszámot vezetsz. Egy (tenant,event_id) esemény pontosan egy helyi növelést jelent; ugyanaz az ID más deltaértékkel konfliktus. A marker és a növelés közös tranzakció.

**Elfogadási feltételek:**

- Az első feldolgozás alkalmazza az eseményt, második és új kapcsolatból történő replay nem növel újra.
- A hatás és marker közé injektált exception után egyik sem marad meg.
- Azonos event_id más tenantnál külön hatókör, azonos tenantnál eltérő payload konfliktus.

**Korlát és review:** A teszt újrakapcsolódást és rollbacket igazol; nem áramkimaradást vagy külső mellékhatást. A marker megőrzési idejét írd le.

## R04

### Ack, retry és DLQ policy

**Cél és szerződés:** Készíts tiszta döntési függvényt committed, transient, invalid és unknown kimenetre. Siker ack, invalid hibasor, transient legfeljebb 4 összes kézbesítésig késleltetett retry. Unknown ne váljon csendes sikerré.

**Elfogadási feltételek:**

- Minden kimenet, első és utolsó próbálkozás parametrizált tesztet kap.
- A broker-adapter tervében szerepeljen, hol tárolod a próbálkozásszámot és mi történik hibás DLQ-routingnál.
- Replay-plan.md írja le a hibasor felelősét, a javítás feltételét, a replay limitjét és a duplikációvédelmet.

**Korlát és review:** Nem kell RabbitMQ/Celery telepítés. A policy eredménye önmagában nem ack vagy tényleges dead-letter kézbesítés.

## R05

### Outbox publikálási hiba

**Cél és szerződés:** Készíts SQLite job + outbox modellt és befecskendezett publish függvényt. A job és esemény létrehozása atomikus. Relay a kiküldött eseményt csak publish-siker után jelöli meg.

**Elfogadási feltételek:**

- Injektált létrehozási hiba esetén sem job, sem outbox rekord ne maradjon.
- Publish utáni, sent jelölés előtti hiba után ugyanaz az event ID újrapublikálható; ezt a teszt szándékosan mutassa ki.
- Végleg sent eseményt a következő relay-futás már ne publikálja.
- Tervezd meg több relay esetén a claim/lease és az elavult tulajdonos írásának védelmét.

**Korlát és review:** A többrelayes rész terv, nem végrehajtott elosztott teszt. Nem kell exactly-once állítást vagy valódi brokert beépíteni.

## R06

### Breaker állapotgép

**Cél és szerződés:** Készíts soros breaker modellt: két egymást követő átmeneti hiba után 10 s nyitott állapot, majd egy próbahívás. Próbasiker zár, próbahiba újabb 10 s nyitást eredményez. Injektált órát használj.

**Elfogadási feltételek:**

- Nyitott állapotban az operation egyáltalán nem hívódik meg.
- Pontos cooldown-határon sikeres és hibás próbát is tesztelj.
- Üzleti elutasítás nem növeli a függőséghiba-számlálót; ennek további állapotkezelését dokumentáld.
- Írd le, mi kellene a modellhez konkurens half-open kérések esetén.

**Korlát és review:** Ne nevezd a soros modellt thread-safe breakernek. A konfiguráció pozitív thresholdot és cooldown-t fogadjon el.

## R07

### Worker drain és cancellation

**Cél és szerződés:** Egy aktív, Eventtel vezérelt munkát végző worker állapotát teszteld: normál befejezés és commit előtti cancellation. Előbbi completion + ack, utóbbi cleanup, de nincs sikeres ack.

**Elfogadási feltételek:**

- Állapotszinkronizálás vezérelje a tesztet, ne időzített sleep.
- A teszt hibás ágán is rendezett legyen minden task.
- Írj shutdown.md-t az új kézbesítések leállításáról, drain-deadline-ról, lejáró lease-ről és végső processzleállításról.
- Külön magyarázd el a commit és ack közti crash esetét.

**Korlát és review:** A helyi eseménylista nem tartós ack. A processz SIGKILL kezelése nem finally-garancia.

[Tananyag](../../knowledge/senior-python/11-reliable-services/README.md) · [Gyakorlatok](../README.md)
