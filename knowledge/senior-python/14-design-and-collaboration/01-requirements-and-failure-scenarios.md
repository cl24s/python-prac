# Követelmények, invariánsok és hibaforgatókönyvek

## Mit kell tudnod?

- Kódolás előtt megnevezni a sikert és a nem vállalt garanciákat.
- Mérhető követelményt és üzleti invariánst külön kezelni.
- Bizonytalanságot explicit feltételezésként dokumentálni.

## Magyarázat

A senior tervezés első eredménye nem osztálydiagram, hanem közös problémaleírás. Ki indítja a műveletet, mit tekint sikernek, mennyi ideig várhat, és mit tehet hiba után? A „legyen gyors és megbízható” nem ellenőrizhető követelmény. A „valid kérések 99%-a 300 ms alatt kap elfogadási választ az egyeztetett terhelésen” már vizsgálható, de külön kell definiálni a mérési pontot, időablakot és kizárásokat.

Az invariáns mindig igaznak szánt üzleti szabály. Például egy tenant csak saját jobját olvashatja, és egy üzleti esemény helyi hatása nem alkalmazható kétszer. Az SLO viszont megengedett hibaarányt írhat le. Nem helyettesítik egymást: egy jó latency SLO mellett is lehet súlyos jogosultsági hiba.

## Esettanulmány: jelentésgeneráló job API

Ez saját oktatási forgatókönyv, nem a kész projekt specifikációja. A kliens jelentést kér, kap egy job ID-t, majd lekérdezi az állapotot. A generálás hosszabb, mint egy szokásos HTTP-kérés.

| Tisztázandó kérdés | Esetben választott feltételezés | Következmény |
| --- | --- | --- |
| Mit jelent az elfogadás? | Tartós job és outbox rekord egy tranzakcióban | 202 csak commit után |
| Mi történik kliensretrykor? | Tenantonként stabil idempotenciakulcs | Payload-eltérésre konfliktus |
| Mi történik workerhibánál? | Újrakézbesítés lehetséges | Idempotens hatás vagy kompenzáció |
| Ki láthatja az eredményt? | Hitelesített tulajdonos tenant | Szűrés minden olvasási útvonalon |
| Meddig tartjuk meg? | Még nyitott üzleti döntés | Retention és törlés nem becsülhető késznek |

A táblázatból látszik, hogy a DB és queue közötti hiba nem ritka kivételként kezelendő utógondolat: alapvetően befolyásolja az elfogadási szerződést. A leállás utáni visszaállás útját már az első tervben meg kell tudni mutatni.

## Hibaforgatókönyvek végigvezetése

1. Validáció előtt megszakad a kérés: nincs elfogadási ígéret.
2. Commit előtt hibázik a DB: nincs tartós job, a kliens újrapróbálhat.
3. Commit után elveszik a válasz: azonos kulccsal ugyanaz a job kereshető vissza.
4. Publikálás után leáll a relay: duplikált kézbesítés lehetséges.
5. Feldolgozás közben elérhetetlen a tároló: a végső eredmény nem jelölhető késznek pusztán a számítás befejezése miatt.

Minden pontnál kérdezd: milyen tartós bizonyíték maradt, ki próbál újra, mi a korlát, és hogyan látja ezt az ügyfél? Ezekből következnek a tesztek és operációs jelzések.

## Senior döntések és tipikus hibák

A hiányzó adatot ne rejtsd el precíz szám mögött. Kapacitásbecsléshez külön jelöld a felhasználói adatot, a saját feltételezést és a mérendő bizonytalanságot. Egy 30 perces célzott kísérlet olcsóbb lehet, mint egy bizonytalan alapra épített architektúra.

A nem célok szűkítik a vállalást: például nincs több régiós aktív-aktív működés az első változatban. Ez nem ürügy a kritikus jogosultsági vagy adatvesztési esetek kihagyására. A terv végét elfogadási feltételek zárják, nem „majd teszteljük”.

## Interview questions

**What would you clarify before designing a job API?**

Válaszvázlat: Elfogadás jelentése, tulajdonos, latency/terhelés, retry és duplikáció, eredménytartósság, retention, hibák és működtetés.

**How is an invariant different from an SLO?**

Válaszvázlat: Az invariáns üzleti helyességi szabály, az SLO egy mérési időablak szolgáltatási célja és hibakerete.

## Önellenőrzés

- [ ] Legalább öt megszakítási pontot végig tudok vezetni.
- [ ] A feltételezéseket nem keverem össze a tényekkel.

## Kapcsolódó gyakorlat

[SD01 – önálló feladat](../../../exercises/senior-python/14-design-and-collaboration.md#sd01)

## Forrás és továbbolvasás

[Google SRE: service level objectives](https://sre.google/sre-book/service-level-objectives/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
