# Kódmegértés, review és mentorálás

## Mit kell tudnod?

- A viselkedést és kockázatot megérteni stílusvélemény előtt.
- Bizonyítékkal és javítási iránnyal adni review-megjegyzést.
- A fejlesztő önállóságát növelő segítséget adni.

## Magyarázat

Review előtt értsd meg a változás célját, a bemenetet, a látható eredményt és a függőségeket. Olvasd el a tesztet és keresd meg a hibaútvonalat. Egy diff önmagában nem mutatja, milyen tranzakcióban vagy lifecycle-ban fut a kód.

Elsőként a helyességet, jogosultsági határokat, adatvesztést, versenyhelyzetet és kompatibilitást vizsgáld. Utána következik az olvashatóság és felesleges összetettség. Az automatikusan ellenőrizhető formázást eszköz kezelje, hogy a review figyelme az érdemi kérdésekre maradjon.

## Esettanulmány: túl korai worker ack

Egy PR leírása szerint „gyorsabb worker”. A változás az acknowledgementet a feldolgozás elé mozgatja. A happy-path teszt zöld és a mért lokális késleltetés csökken. A döntő kérdés: mi történik, ha ack után, commit előtt leáll a folyamat? Ha nincs külön helyreállítási mechanizmus, az elfogadott munka elveszhet.

Használható review-megjegyzés angolul:

> Blocking: if the worker stops after the acknowledgement but before the database commit, the broker may no longer redeliver this job. Could we acknowledge after the commit and add a failure-path test for that boundary?

Ez saját mintaszöveg. Megnevezi a konkrét végrehajtási utat, az ügyfélhatást és egy ellenőrizhető változtatási irányt. Nem minősíti a szerzőt. A „this is bad” nem ad ennyi segítséget.

## Megjegyzések osztályozása

| Kategória | Példa | Elvárt lezárás |
| --- | --- | --- |
| Blocking | Tenantellenőrzés hiányzik | Javítás vagy a kockázatot megszüntető bizonyíték |
| Question | Nem világos a retry tulajdonosa | Szerződés és felelősség tisztázása |
| Suggestion | Egy hosszú függvény kettébontható | Mérlegelhető javítás |
| Nit | Lokális névválasztási preferencia | Ne blokkoljon önmagában |

A súlyosságot ne a megjegyzés határozottsága, hanem a hatás és valószínűség indokolja. Az alacsonyabb kockázatú javaslatot ne fogalmazd kötelező biztonsági hibaként.

## Meglévő kód megértése

Egy ismeretlen modulnál kövesd végig az entry pointot, az állapotváltozást, a mellékhatásokat és a kivételek útját. Ha a kívánt viselkedés nincs dokumentálva, characterization test rögzítheti a megfigyelt működést. Ez nem jelenti, hogy a régi működés üzletileg helyes; a hibát külön döntéssel kell módosítani.

Refaktorálást és új funkciót lehetőleg review-zható lépésekre bonts. Ha egy PR egyszerre nevez át mindent, formáz és változtat tranzakciót, nehezebb észrevenni a jelentős eltérést. A kis diff sem automatikusan kis kockázat: egyetlen ack-sor elég a garancia megváltoztatásához.

## Mentorálás

Előbb kérdezz rá a gondolatmenetre: „Melyik állapot marad meg processzhiba után?” Adj egy kisméretű ellenpéldát, majd kérj tesztet. Ha a tanuló elakad, fokozatosan adj támpontot; ne írd át rögtön helyette az egész megoldást. Review végén fogalmazzatok meg újrahasználható tanulságot és konkrét következő lépést.

A review ideje is csapatköltség. A nagy, kockázatos változásról érdemes kódolás előtt rövid design review-t tartani; a nézetkülönbséget így olcsóbb rendezni.

## Interview questions

**How do you make a review comment actionable?**

Válaszvázlat: Konkrét sorhoz vagy működéshez kötöm, elmondom a hatást, a súlyosságot és a kívánt bizonyítékot vagy javítási irányt.

**How would you help a developer without solving everything for them?**

Válaszvázlat: A gondolatmenetet kérdezem, kis ellenpéldát és tesztcélt adok, majd fokozatos segítséggel hagyom önállóan javítani.

## Önellenőrzés

- [ ] A kritikám a kódra és következményre vonatkozik.
- [ ] Meg tudom különböztetni a blocker és preferencia szerepét.

## Kapcsolódó gyakorlat

[SD06 – önálló feladat](../../../exercises/senior-python/14-design-and-collaboration.md#sd06)

## Forrás és továbbolvasás

[Google engineering practices: code review](https://google.github.io/eng-practices/review/reviewer/looking-for.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
