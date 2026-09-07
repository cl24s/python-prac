# Tesztelés és kódminőség – önálló feladatok

A feladatokhoz nincs kész megoldás. A kód és tesztek később saját almappába kerüljenek; a dokumentáció maradjon Markdown. A feladat végén írd le, mely szerződést ellenőrizted, mi futott és mi nem. A tananyag elkészülte nem jelent feladatteljesítést.

## Q01

### Teszthatárok a logelemzőnél

**Cél:** Kockázatonként indokolni a legszűkebb hasznos tesztet.

**Bemenet, kimenet és viselkedés:** Először test-plan.md készüljön egy tervezett CLI-hez. A CLI UTF-8 fájlt olvas, az ERROR kezdetű sorokat számolja, stdouton egy egész számot ad, sikerre 0, nem olvasható fájlra 2 exit code-dal. A fájlhiba üzenete stderrre kerül.

**Elfogadási feltételek:**

- Legalább 6 kockázatot rendelj unit-, integrációs vagy end-to-end teszthez; mindenhez legyen bemenet és konkrét elvárt eredmény.
- Szerepeljen üres fájl, nem ASCII szöveg, ERROR szó sor közepén, hiányzó fájl, helytelen CLI-argumentum és helyes futás.
- A CLI-argumentum hibájának exit code-ját és a naplósor felismerésének pontos szabályát külön rögzítsd.
- Válassz ki két eltérő határú tesztet; az implementáció elkészülte után ezek legyenek ténylegesen futtathatók.

**Korlát és review:** A terv nem helyettesíti a későbbi futtatást. A valódi fájlolvasást ne mockold az adapter-integrációs tesztben. A hiányzó fájl determinisztikusabb első hibapélda, mint a futtató jogosultságaitól függő chmod-teszt.

## Q02

### Parametrizált timeout-parser

**Cél:** Határértékeket és tesztadat-izolációt ellenőrizni pytesttel.

**Bemenet, kimenet és viselkedés:** A parse_timeout(text: str) egész számként értelmezett 1–30 másodpercet fogad el. A környező whitespace megengedett. Szintaktikai hibánál és tartományhibánál ValueError, tartományhibánál pontosan "timeout out of range" az üzenet. A tesztmodul és a parser külön fájl legyen.

**Elfogadási feltételek:**

- Parametrizáltan ellenőrizd: "1", "30", " 8 "; külön hibás esetek: "0", "31", "-1", "1.5", "many".
- A tesztek beszédes ID-kat és pontos exceptiontípust használnak.
- Egy function-scope fixture adjon módosítható konfigurációs dictet; két külön teszt bizonyítsa, hogy nincs állapotszivárgás.
- A parser módosítható globális állapot nélkül működjön.

**Korlát és review:** Az exceptionkezelés blokkja csak a hibára várt műveletet tartalmazza. Külön indokold, mikor választanál szélesebb fixture scope-ot.

## Q03

### Repository fake és adapter-szerződés

**Cél:** Ugyanazzal a teszttel ellenőrizni a fake és egy valódi helyi tároló viselkedését.

**Bemenet, kimenet és viselkedés:** A repository add(id: str, value: str) és get(id: str) műveletet kínál. Duplikált id: saját DuplicateId hiba; hiányzó id: KeyError; a value lehet üres szöveg. Készíts in-memory és SQLite-adaptert, majd közös parametrizált szerződésteszteket.

**Elfogadási feltételek:**

- Mindkét adapteren fusson a mentés/visszaolvasás, üres érték, hiányzó id és duplikáció tesztje.
- Duplikáció után az eredeti érték maradjon meg; a sikertelen add ne írjon felül.
- A SQLite-teszt tmp_path alatti valódi fájlt használjon, a kapcsolat lezárása fixture-teardownban történjen.
- A SQLite egyediségét constraint védje, és az adapter a megfelelő hibát fordítsa DuplicateId-ra.

**Korlát és review:** Nem kell általános ORM vagy szerver. A tárolóhoz már elérhető standard library sqlite3 elegendő. A többprocesszes konkurencia és a tranzakcióizoláció teljes tesztelése nem része ennek a feladatnak; ezt a korlátot írd le.

## Q04

### Flaky várakozás kiváltása

**Cél:** Sorrendet és időfüggő döntést valódi várakozás nélkül tesztelni.

**Bemenet, kimenet és viselkedés:** Készíts egy async worker demót, amely started Eventtel jelzi az indulást, vár egy release Eventre és finally blokkban cleanupot jelez. Teszteld a normál befejezést és az indulás utáni külső cancellationt. Külön expires_at ellenőrző függvény kapjon befecskendezett órát.

**Elfogadási feltételek:**

- Normál úton a release előtt a worker még nem fejeződik be; release után a várt eredményt adja.
- Cancellation esetén az awaitelő CancelledErrort kap és a cleanup bizonyítottan megtörtént.
- A teszt saját hibája után sem marad pending task.
- Az expiry-teszt deadline előtt, pontosan rajta és utána is ellenőriz, sleep nélkül.

**Korlát és review:** Event vagy explicit állapot vezérelje a tesztet. Felső védő timeout megengedett, időtartam-alapú sebességassertion nem szükséges. Nem kell async pytest-plugin; asyncio.run-os teszt is megfelel.

## Q05

### Viselkedésmegőrző refaktorálás

**Cél:** Független elvárással és regressziós tesztekkel védeni a szerződést.

**Bemenet, kimenet és viselkedés:** Írj egy kezdeti group_levels(lines: list[str]) függvényt: a sor első whitespace-del elválasztott szava a csoport kulcsa, üres/whitespace sor kimarad, kis- és nagybetű különbözik. Eredmény dict[str, int]. Ezután refaktoráld kisebb, érthető részekre.

**Elfogadási feltételek:**

- Előbb legyen teszt üres bemenetre, whitespace-ra, ismételt szintre és eltérő kis-/nagybetűre.
- Konkrét bemenethez kézzel meghatározott eredményt írj; ne az implementációval számold az expected értéket.
- A bemeneti lista nem módosul; ezt teszt ellenőrizze.
- Írj le egy szándékos hibát, amelyet a tesztek észrevennének, és mutasd be a célzott bukást, majd a helyreállított sikeres futást.

**Korlát és review:** Nem kell kész hibás változatot a main ágon hagyni. A review-ban térj ki arra, hogy a sorrend vagy a későbbi generátoros átírás a jelen szerződés része-e.

## Q06

### Érdemi code review

**Cél:** Kockázatot és javítási elvárást világosan megfogalmazni.

**Bemenet, kimenet és viselkedés:** A Q02 vagy Q05 saját megoldásáról készíts review.md-t. A dokumentum tartalmazza a szerződést, a helyességi és tesztelhetőségi megállapításokat, valamint a futtatott ellenőrzéseket.

**Elfogadási feltételek:**

- Legalább három konkrét, kódrészhez vagy viselkedéshez kapcsolt vizsgálati pont legyen; ha nincs hiba, ne találj ki hibát.
- Különítsd el a blokkoló hibát az opcionális javítástól, és mindkettőnél nevezd meg a következményt.
- A tesztfuttatás pontos parancsa és eredménye szerepeljen; a nem futtatott lint/mypy ne legyen sikeresnek jelölve.
- Egy releváns javítás vagy indokolt „nincs változtatás” döntés után foglald össze a fennmaradó korlátot.

**Korlát és review:** A megjegyzések ne általános clean code jelszavak legyenek. Nem szükséges új CI-rendszert bevezetni a feladathoz.

[Kapcsolódó tananyag](../../knowledge/senior-python/07-testing-and-quality/README.md) · [Gyakorlatok áttekintése](../README.md)
