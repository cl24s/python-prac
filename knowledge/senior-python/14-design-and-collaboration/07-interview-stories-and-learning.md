# Saját éles példák és szakmai tanulságok bemutatása

## Mit kell tudnod?

- Saját szerepet és csapateredményt pontosan elválasztani.
- Probléma, döntés, eredmény és tanulság mentén történetet építeni.
- Nem ismert számot vagy okot nem tényként előadni.

## Magyarázat

Senior interjún a konkrét történet mutatja meg, hogyan gondolkodsz bizonytalan helyzetben. Egy eszközlista nem árulja el, milyen problémát oldottál meg és mit vállaltál érte. Készülj olyan esettel is, ahol az első hipotézis téves volt, vagy a választott megoldásnak számottevő hátránya volt.

A történet legyen két mélységben elmondható: körülbelül kétperces áttekintés és részletes szakmai folytatás. A rövid változatban csak a következtetést megértető technikai részlet maradjon. Mélyebb kérdésre tudj tranzakcióhatárt, mérést, rolloutot és alternatívát mutatni.

## Történetvázlat

| Rész | Mit mondj el? | Mitől hiteles? |
| --- | --- | --- |
| Probléma | Ki mit tapasztalt, mi volt a hatás? | Időablak, konkrét hibajelenség |
| Saját feladat | Miért feleltél te? | Egyéni és csapatmunka külön |
| Vizsgálat | Milyen hipotéziseket ellenőriztél? | Mérés vagy megfigyelés |
| Döntés | Mit választottatok és miért? | Alternatíva és hátrány |
| Eredmény | Mi változott? | Azonos mérési feltételek, vagy őszinte kvalitatív eredmény |
| Tanulság | Mit csinálnál másként? | Megváltozott gyakorlat, teszt vagy monitor |

A váz a STAR történetet technikai döntésekkel és utólagos tanulással egészíti ki. Nem kell mindent időrendben elmondani: az interjúztató számára a lényeg legyen korán világos.

## Kitölthető angol válaszkeret

> The problem was [observable impact]. I was responsible for [my scope]. We considered [alternatives], and chose [decision] because [constraint or evidence]. The main trade-off was [cost]. We verified the change using [measurement or test]. The result was [supported outcome]. Looking back, I would [specific improvement].

Ez sablon, nem kész személyes történet. Csak tényleges saját tapasztalattal töltsd ki. Ha a hatás nem volt számszerűen mérve, mondd ezt: „We did not have a reliable before-and-after latency measurement, but the recurring incident stopped during the observation period.” Ez is csak akkor használható, ha igaz, és az időszakot meg tudod nevezni.

## Három történettípus

Egy incidensben a korlátozott információ alatti döntés és helyreállítás érdekes. Egy teljesítményjavításban a baseline, reprezentatív mérés és regresszióvédelem. Egy migrációban a kompatibilitás, kockázatcsökkentés és visszaállítási pont. Mindegyikhez készülj azzal, milyen bizonyíték cáfolta volna az első elképzelésedet.

Az „én megoldottam” és „a csapat megoldotta” helyett pontos munkamegosztást írj: te elemezted a logokat és javasoltad a módosítást; valaki más készítette a rolloutot. Ettől a történet pontosabb, nem gyengébb.

## Incidens utáni tanulás

A postmortem az események és rendszerfeltételek vizsgálata. Ne zárd le azzal, hogy valaki hibázott. Mi tette lehetővé, hogy egy hibás konfiguráció átmenjen, és miért nem látszott hamarabb? Egy intézkedésnek felelős, határidő és ellenőrizhető eredmény kell. A „legyünk óvatosabbak” nem erős megelőző kontroll.

A saját felkészülési jegyzetből anonimizáld az ügyfelet és belső címeket, és ne másolj incidenslogot vagy titkot a repóba. A szakmai döntés bemutatható a bizalmas adat nélkül is. A tanulási roadmap csak az átbeszélt, bizonyított megértéstől változzon, a történetfájl lététől ne.

## Interview questions

**Tell me about a technical decision you would make differently today.**

Válaszvázlat: Valós esetet választok, megadom az akkori információt és korlátot, majd azt, mit tanultam és hogyan változtatnám a döntést.

**How do you distinguish evidence from assumptions in an incident story?**

Válaszvázlat: A megfigyelt tényeket, a hipotézist és a bizonyított okot külön mondom; a nem mért eredményhez nem találok ki számot.

## Önellenőrzés

- [ ] Van rövid és részletes változatom ugyanarra a történetre.
- [ ] A saját szerepem és a bizonyíték világos.

## Kapcsolódó gyakorlat

[SD07 – önálló feladat](../../../exercises/senior-python/14-design-and-collaboration.md#sd07)

## Forrás és továbbolvasás

[Google SRE: postmortem culture](https://sre.google/sre-book/postmortem-culture/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
