# Technikai döntések, ADR és kompromisszumok

## Mit kell tudnod?

- Röviden rögzíteni a döntés kontextusát és következményeit.
- Érdemi alternatívákat összehasonlítani azonos kritériumok mentén.
- Bizonyítékhoz és felülvizsgálati feltételhez kötni a választást.

## Magyarázat

Egy technikai döntés dokumentuma azt őrzi meg, miért tűnt helyesnek az adott választás az akkori feltételek mellett. Az ADR nem teljes rendszerleírás. Egyetlen lényeges döntést, státuszt, kontextust, alternatívát és következményt érdemes tartalmaznia. Ha változik, új döntés hivatkozzon a régire; a történet ne tűnjön el.

A jó összehasonlításban a másik út is komolyan vehető. Nem tisztességes egy általános „rossz legacy” és egy részletesen optimalizált új terv közül választani. Ugyanazokat az üzleti, működtetési és migrációs szempontokat vizsgáld mindkettőnél.

## Mintadöntés: hol fusson a jelentésgenerálás?

**Állapot:** oktatási példa, javasolt. **Kontextus:** a jelentés hosszú, és elfogadás után folyamatújraindítás esetén sem veszhet el észrevétlenül. **Döntés:** tartós job/outbox és külön worker. A pontos broker választása ettől külön döntés.

| Opció | Elfogadás utáni helyreállás | Egyszerűség | Költség |
| --- | --- | --- | --- |
| Kérésen belüli végrehajtás | Kliens és szerver megszakítása kezelendő | Kevés komponens | Hosszú kapcsolat, bizonytalan eredmény |
| Processzen belüli háttértask | Tartósságot önmagában nem ad | Könnyű indulás | Restartkor elveszhet a munka |
| Tartós queue/worker folyamat | Újrakézbesítés és reconciliation tervezhető | Több komponens | Deduplikáció, monitoring, üzenetséma |

**Következmény:** a HTTP elfogadás és a tényleges elkészülés külön státusz. Duplikációval számolunk; exactly-once külső mellékhatást nem ígérünk. **Felülvizsgálat:** ha a feladat rövid, elveszthető és kizárólag best-effort, az egyszerűbb út újra értékelhető.

Ez a mintadöntés a tananyag saját forgatókönyve. Nem hajt végre projektváltozást és nem írja elő minden háttérfeladathoz ugyanezt.

## Újrahasználható ADR-vázlat

A saját feladatban egy Markdown-fájlt készíts a következő mezőkkel: cím és azonosító; státusz és dátum; probléma; tények és feltételezések; döntési kritériumok; legfeljebb néhány érdemi alternatíva; döntés; vállalt hátrányok; ellenőrzési terv; felülvizsgálati jel; felelős.

A „scalable”, „clean” és „future-proof” szavakat váltsd konkrét állításra. Például: az API és worker külön kapacitást kap; a domain teszt HTTP nélkül fut; az adaptercsere egy kijelölt szerződésen történik. Ne adj kitalált pontszámot ismert mérés helyére. Súlyozott táblázat csak akkor segít, ha a súlyok és pontok értelme is világos.

## Döntés kommunikálása

Interjún először egy mondatban válaszolj: „Ebben a helyzetben X-et választanék, mert Y a meghatározó követelmény.” Utána mondd el a legfontosabb hátrányt és azt, milyen adat változtatná meg a döntést. A hallgató így követni tudja a gondolatmenetet, mielőtt eszközrészletekbe mennél.

Eltérő véleménynél előbb a kritériumról egyezzetek meg. Ha az egyik fél kiadási sebességet, a másik incidenskockázatot optimalizál, újabb könyvtárnevek felsorolása nem oldja fel a vitát. Időkorlátos kísérlet vagy visszafordítható döntés hasznosabb lehet.

## Interview questions

**What makes an ADR useful six months later?**

Válaszvázlat: Megőrzi a kontextust, alternatívákat, vállalt hátrányokat és újraértékelési feltételeket, nem csak a választott eszközt.

**How would you explain a trade-off to a non-specialist?**

Válaszvázlat: Az üzleti hatással kezdek: mit nyerünk, mit fizetünk érte, és milyen bizonyíték alapján döntünk.

## Önellenőrzés

- [ ] A választott opció hátrányát is ki tudom mondani.
- [ ] Megnevezem, mitől változna meg a döntés.

## Kapcsolódó gyakorlat

[SD04 – önálló feladat](../../../exercises/senior-python/14-design-and-collaboration.md#sd04)

## Forrás és továbbolvasás

[Michael Nygard: Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
