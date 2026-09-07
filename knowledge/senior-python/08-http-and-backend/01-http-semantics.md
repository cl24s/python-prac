# HTTP-metódusok, státuszkódok és idempotencia

## Mit kell tudnod?

- A HTTP-szerződést a művelet jelentéséhez választani.
- Megkülönböztetni a safe, idempotent és cache-elhető tulajdonságokat.
- Timeout után az eredmény bizonytalanságával számolni.

## Magyarázat

Az API-szerződés a kliensnek ígért viselkedés. A metódus és státuszkód nem puszta route-dekoráció: retryt, cache-t és klienslogikát befolyásol. A safe metódus olvasási szándékú; az idempotens művelet ismétlése ugyanazt a szándékolt szerverállapotot eredményezi. A válasz közben változhat: a második DELETE adhat 404-et, miközben az erőforrás továbbra is törölt.

| Metódus | Tipikus használat | Safe | Idempotens szemantika |
| --- | --- | --- | --- |
| GET / HEAD | Reprezentáció / fejlécek lekérése | Igen | Igen |
| POST | Feldolgozás vagy létrehozás | Nem | Nem általánosan |
| PUT | A cél-erőforrás állapotának létrehozása/cseréje | Nem | Igen |
| PATCH | Részleges módosítás | Nem | A módosítás műveletétől függ |
| DELETE | Erőforrás eltávolítása | Nem | Igen |

Például a „status legyen paused” ismételhető állapotbeállítás, a „retry_count növelése eggyel” ismétléskor újabb változtatás. Az idempotencia a szerződés és az implementáció közös tulajdonsága.

## Saját API-terv: job indítása

| Esemény | Választerv | Kliens teendője |
| --- | --- | --- |
| Új job erőforrás elkészült | 201, Location: /jobs/42 | Az erőforrás lekérdezhető |
| Feldolgozási kérés elfogadva, eredmény még nincs | 202, állapotkövető URL a szerződés szerint | Később státuszlekérdezés |
| Sikeres törlés válaszbody nélkül | 204 | Nincs JSON dekódolás |
| Hibás bemenet | 400 vagy dokumentált 422 | Bemenet javítása |
| Állapotütközés | 409 | Aktuális állapot alapján új döntés |
| Átmeneti túlterhelés | 503 | Korlátozott, biztonságos retry mérlegelése |

Ez saját tervezési példa, nem univerzális státuszkód-térkép. Egy erőforrás már létrejöhet, miközben az általa leírt háttérmunka még fut. A 202 önmagában nem tartóssági vagy sikeres befejezési garancia.

## Idempotency key: mit kell valóban megoldani?

Egy POST /jobs kérés timeoutja után a kliens nem tudja biztosan, hogy létrejött-e a job. Ha ugyanazzal a kulccsal ismétli, a szervernek össze kell kapcsolnia a kulcsot, a hívó/tenant hatókörét, a kérés tartalmát és a korábbi eredményt. Ugyanaz a kulcs eltérő tartalommal legyen dokumentált konfliktus, ne véletlenül másik job.

Két egyidejű kérésnél a „megnézem egy dictben, majd létrehozom” nem atomikus. Tartós egyediség és tranzakciós vagy más összehangolt állapotkezelés kell; tisztázni kell a folyamatban lévő kérés, a kulcs lejárata és a szerverösszeomlás viselkedését. Egy processz memóriája nem véd több worker és újraindítás ellen.

## Tipikus hibák

GET endpointtal állapotot változtatni veszélyes: kliens, crawler vagy előtöltés is meghívhatja. Az összes POST vak újrapróbálása duplikált mellékhatást okozhat. A minden hibára 200-at adó API pedig elrejti a hibát az általános kliens- és megfigyelési eszközök elől.

## Interview questions

**Can an idempotent request return different responses?**

Válaszvázlat: Igen; a szándékolt állapotváltozás ismételhetősége számít. Az első és második DELETE státusza eltérhet.

**What does a timeout tell you about a POST request?**

Válaszvázlat: A kliens nem kapott időben eredményt; a szerver ettől még végrehajthatta. Retry előtt idempotenciát és eredmény-visszakeresést kell tervezni.

## Önellenőrzés

- [ ] Elmagyarázom a 201 és 202 választását a job API-n.
- [ ] Nem keverem az idempotenciát az azonos válaszbodyval.

## Kapcsolódó gyakorlat

[H01 – önálló feladat](../../../exercises/senior-python/08-http-and-backend.md#h01)

## Forrás és továbbolvasás

[HTTP semantics](https://www.rfc-editor.org/rfc/rfc9110.html)

[PATCH method](https://www.rfc-editor.org/rfc/rfc5789.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
