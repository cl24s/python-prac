# CI, ellenőrzési kapuk és artifactok továbbadása

## Mit kell tudnod?

- Külön kezelni a forráskód és a telepíthető csomag ellenőrzését.
- Reprodukálható, minimális jogosultságú pipeline-t tervezni.
- Ugyanazt az artifactot továbbvinni a környezetek között.

## Magyarázat

A CI egy állítás bizonyítékait gyűjti: ez a commit adott környezetben teljesítette a meghatározott ellenőrzéseket. A zöld státusz nem univerzális helyességi garancia. A gate csak azokat a hibákat fogja meg, amelyekre van érzékeny ellenőrzés.

| Lépés | Mire érzékeny? | Mire nem bizonyíték? |
| --- | --- | --- |
| Lint és formázás | Következetlenség, ismert kódminták | Üzleti helyesség |
| Típusellenőrzés | Statikusan leírt szerződések | Külső input runtime érvényessége |
| Unit teszt | Lokális viselkedés | Valódi DB/broker együttműködés |
| Integráció | Valós adapter és szolgáltatás | Teljes éles terhelés |
| Wheel smoke teszt | Telepíthetőség, import, entry point | Minden funkció működése |
| Dependency scan | Ismert sérülékenységek | Ismeretlen hiba hiánya |

## Konkrét pipeline-terv

A későbbi job API minden PR-jén tiszta checkoutból, rögzített Python- és eszközverziókkal indulunk. A gyors ellenőrzések és a szükséges integrációs tesztek után wheel készül. Egy új környezet csak ezt a wheelt és a rögzített runtime függőségeket kapja. Az import/indulás ellenőrzése a checkouton kívül fut, nehogy a forrásfa pótolja a hibás csomagot.

A kiadás a commit SHA-t, az artifact digestjét és a tesztjelentést összeköti. Staging és production ugyanazt az artifactot kapja, eltérő runtime konfigurációval. Az élesítés előtti újraépítés megváltoztathatja a függőségeket, így elveszne a staging teszt bizonyító ereje.

A projekt elkészülte után jellemző parancsok: `python -m pytest`, `python -m mypy src`, `ruff check .`, `python -m build`. Ezek itt folyamatpéldák; nincs kész CI-workflow és ebben a fejezetben nem futott build/lint/mypy. A parancsok csak telepített eszközökkel és valódi projekttel működnek.

## Hiba- és bizalmi határok

Egy PR forrása végrehajtható, nem megbízható input. Ne adj éles secretet a teszteknek; a publish és deploy külön jogosultságú folyamat legyen. A GitHub Actions actionjeinek teljes commit SHA-ra rögzítése csökkenti a mozgó tagek kockázatát, de frissítési folyamatot igényel. A token alapjogosultsága minimális legyen. Külső PR szövegét ne illeszd közvetlen shell-scriptbe.

A cache gyorsításra való. Kulcsa tartalmazza az érdemi környezeti jellemzőket és a függőségdeklaráció lenyomatát; cache miss esetén ugyanúgy helyes build kell. A megbízható release ne vegyen át ellenőrizetlen artifactot egy kevésbé megbízható workflow-ból.

Flaky tesztnél rögzítsd az eredeti hibát és az ismétlés eredményét. A korlátlan újrafuttatás elrejti a hibaarányt. Tudatos karanténhoz felelős és javítási határidő kell; nem tekinthető „minden teszt zöld” állapotnak.

## Interview questions

**Why promote the same artifact instead of rebuilding for production?**

Válaszvázlat: Így a stagingben vizsgált bájtokat telepítjük; az új build más függőséget vagy eszközverziót hozhat.

**What does a green pipeline actually prove?**

Válaszvázlat: A megnevezett ellenőrzések teljesültek egy konkrét környezetben. A hiányzó tesztszinteket és veszélyeket külön kell kezelni.

## Önellenőrzés

- [ ] El tudom különíteni a PR és release jogosultságait.
- [ ] Meg tudom mondani, mi történik cache nélkül.

## Kapcsolódó gyakorlat

[OP04 – önálló feladat](../../../exercises/senior-python/13-packaging-security-and-operations.md#op04)

## Forrás és továbbolvasás

[GitHub Actions secure use](https://docs.github.com/en/actions/reference/security/secure-use)

[PyPA build](https://build.pypa.io/en/stable/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
