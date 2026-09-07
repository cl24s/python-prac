# Éles futtatás, kapacitás, megfigyelés és graceful shutdown

## Mit kell tudnod?

- A szerver, replika és worker kapacitását közösen méretezni.
- Liveness, readiness és startup feltételt megkülönböztetni.
- Leállításkor a folyamatban lévő kérések és jobok sorsát rögzíteni.

## Magyarázat

FastAPI az ASGI-alkalmazás; a kapcsolatokat ASGI-szerver kezeli. A processzeket és újraindításukat deploymentkörnyezet vagy processzfelügyelet szervezi. Fejlesztői reload kényelmi funkció, nem éles workerstratégia. A szerver kiválasztása nem javítja ki az app deadlockját vagy korlátlan memórianövekedését.

Minden worker külön memóriát és általában saját poolokat kap. A példabeli kapacitásterv: 3 replika × 2 worker × (5 alapkapcsolat + 2 overflow) = legfeljebb 42 ilyen poolból engedett kapcsolat. Ez tervezési felső becslés; nem jelenti, hogy mind mindig nyitva van. A migrációs jobok, más szolgáltatások és adminforgalom kerete is kell.

## Timeoutok és overload

| Réteg | Saját döntés |
| --- | --- |
| Bejövő szerver | Kapcsolati és kéréskezelési korlátok |
| Reverse proxy | Upstream várakozás és bodykorlát |
| Alkalmazás | Teljes felhasználási eset deadline-ja |
| HTTP/DB-kliens | Kapcsolat, pool-várakozás, I/O vagy query timeout |
| Leállítás | Drain és végső processzleállítás időkerete |

Egy beállítás neve nem biztos, hogy teljes kérésidőt jelent: például a keep-alive timeout két kérés közti üres kapcsolatot érinthet. A saját deadline ne adjon minden retryra új teljes keretet. Overloadnál a korlátlan várakozósor csak későbbre tolja a hibát és növeli a memóriát.

## Saját leállítási forgatókönyv

1. Az instance kerüljön ki az új forgalom célpontjai közül; számolj a routingfrissítés késésével.
2. A szerver ne fogadjon korlátlanul új munkát; a meglévő kérés kapjon dokumentált befejezési időt.
3. A még futó munkát fejezd be vagy kooperatívan szakítsd meg; a kritikus job tartós állapota legyen visszakereshető.
4. Az aktív feladatok után zárd a DB/HTTP-erőforrásokat.
5. A külső processzleállítási keret legyen összhangban a szerver drain idejével.

Ez tervezési anyag, nem végrehajtott deployteszt. A SIGKILL vagy gépkiesés nem ad garantált cleanupot. A folyamat memóriájában tárolt háttérmunka elveszhet.

## Health és megfigyelés

Liveness: életképes-e a processz, indokolt-e újraindítani? Readiness: fogadhat-e most érdemi forgalmat? Startup: befejezte-e az induláshoz szükséges inicializálást? Minden DB-lassulásra bukó liveness tömeges újraindítással súlyosbíthatja a kiesést.

Mérd a request latencyt percentilisekkel, hibaarányt, in-flight kéréseket, pool-várakozást és függőséghibákat. Request ID alapján kövesd a hibát, de ne használj egyedi request ID-t metrika-labelként: túl nagy kardinalitást okoz. A token, teljes body és SQL-paraméter nem automatikusan naplózható adat.

## Senior döntések és tipikus hibák

Proxyfejléceket csak megbízható proxy felől fogadj el. A root_path alkalmazásalatti mountot írhat le; nem authorization. A több replika önmagában nem oldja meg a közös DB szűk keresztmetszetét.

Az induló projektben először legyen reprodukálható futtatási parancs, health szerződés, korlátos pool és mérhető kritikus út. A worker-számot reprezentatív terhelés alapján állítsd; univerzális „CPU × kettő” szabályt ne alkalmazz bizonyítás nélkül.

## Interview questions

**Why can increasing worker count reduce reliability?**

Válaszvázlat: Megsokszorozza a memóriát, poolokat és downstream terhelést; ugyanazt a DB-limitet hamarabb elérhetjük.

**What does graceful shutdown not guarantee?**

Válaszvázlat: Kényszerleállításkor nincs biztos cleanup, és a nem tartós background munka elveszhet.

## Önellenőrzés

- [ ] Kiszámolom a teljes deployment DB-kapcsolati keretét.
- [ ] Külön tudom indokolni a readiness és liveness feltételeit.

## Kapcsolódó gyakorlat

[F11 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f11)

## Forrás és továbbolvasás

[FastAPI deployment concepts](https://fastapi.tiangolo.com/deployment/concepts/)

[Uvicorn settings](https://uvicorn.dev/settings/)

[Behind a proxy](https://fastapi.tiangolo.com/advanced/behind-a-proxy/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
