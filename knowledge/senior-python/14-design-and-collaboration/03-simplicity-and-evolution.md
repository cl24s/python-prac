# Egyszerű megoldás, moduláris monolit és bővíthetőség

## Mit kell tudnod?

- A jelenlegi igényből kiindulva választani architektúrát.
- Megkülönböztetni a logikai modulhatárt a hálózati határtól.
- Konkrét változási jelhez kötni a későbbi szétválasztást.

## Magyarázat

Az egyszerűség a teljes rendszer működtetési és változtatási költségére vonatkozik. Egy rövid endpoint, amely közvetlenül öt külső szolgáltatást módosít, kódsorban egyszerű, hibakezelésben drága lehet. Ugyanígy egy kis csapatnak a tíz microservice több kiadási, tracing- és ügyeleti munkát hozhat, mint üzleti előnyt.

A moduláris monolit egyben települő alkalmazás világos belső felelősségekkel. A modulok saját publikus interfészen keresztül működnek együtt, és nem írják tetszőlegesen egymás adatait. Egy közös DB nem indokolja a korlátlan kereszthivatkozást. A hexagonális tervezés a külső technológiákhoz való kapcsolódás módja; monolitban és szolgáltatásban is használható.

## Esettanulmány: első job API

Saját feltételezések: kis fejlesztőcsapat, egy régió, mérsékelt és még nem pontosan mért forgalom, egyszerű jelentésgenerálás. Első döntésként egy moduláris API és ugyanabból a kódbázisból külön futtatott worker indokolt lehet. A két processztípus külön skálázható anélkül, hogy rögtön önálló üzleti szolgáltatásokat és teljesen külön kódbázisokat hoznánk létre.

| Választás | Előny ebben az esetben | Vállalt költség |
| --- | --- | --- |
| Egy kódbázis és közös release | Könnyebb szerződésváltoztatás | Összekapcsolt kiadás |
| Külön API és worker processz | Eltérő CPU/I/O kapacitás | Queue és üzenetséma kezelése |
| Modulonként kijelölt adatgazda | Átlátható módosítások | Határok betartatása review-val |
| Későbbi szolgáltatáskiválasztás | Valós igényből indulhat | A bontás későbbi migrációs munka |

A külön worker nem oldja meg automatikusan az idempotenciát és a schema compatibilityt. A 11. témakör garanciái már az első verzióban szükségesek, ha tartós aszinkron feldolgozást ígérünk.

## Mikor bontanád tovább?

Konkrét jel lehet az egymást blokkoló, rendszeres kiadási igény; eltérő adatbiztonsági határ; erősen különböző kapacitásprofil; vagy külön felelős csapat stabil üzleti szerződéssel. A „később nagyok leszünk” kevés. Előbb mérd, hogy a külön worker, jobb index vagy belső modulhatár megoldja-e a problémát.

Kiválasztás előtt írd le az új hálózati hibákat, timeoutot, retryt, API/üzenetverziózást, adatmigrációt és ügyeleti tulajdonost. Ha a szolgáltatások csak együtt deployolhatók, miközben hálózaton hívják egymást, a függetlenség előnye korlátozott marad.

## Tipikus hibák

Előre gyártott pluginrendszer egyetlen megvalósításhoz; minden entitáshoz önálló service; általános CRUD repository az eltérő üzleti műveletek helyett. Másik véglet a technológiai részletek szétkenése a kódban, amely valódi igénynél nagyon drágává teszi a cserét.

A jó bővítési pont a várható változás és egy már létező felelősséghatár találkozása. Ne implementálj három képzeletbeli tárolót, de a domainbe se égesd bele a HTTP és SQL fogalmait.

## Interview questions

**When would you start with a modular monolith?**

Válaszvázlat: Ha a csapat, terhelés és kiadási igény még nem indokol önálló szolgáltatásokat, de világos belső üzleti határok már kellenek.

**What evidence would justify extracting a service?**

Válaszvázlat: Mért skálázási különbség, független kiadás vagy bizalmi/csapathatár, a hálózati és operációs költségek vállalásával.

## Önellenőrzés

- [ ] A választást a forgatókönyvhöz kötöm, nem divathoz.
- [ ] Külön tudom mondani a logikai és deploymenthatárt.

## Kapcsolódó gyakorlat

[SD03 – önálló feladat](../../../exercises/senior-python/14-design-and-collaboration.md#sd03)

## Forrás és továbbolvasás

[Martin Fowler: Monolith First](https://martinfowler.com/bliki/MonolithFirst.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
