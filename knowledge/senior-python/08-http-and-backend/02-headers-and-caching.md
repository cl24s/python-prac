# Headerek, reprezentációk és feltételes kérések

## Mit kell tudnod?

- Elkülöníteni a Content-Type és Accept szerepét.
- ETaggel feltételes olvasást és elvesző módosítás elleni védelmet tervezni.
- Tudatos cache-szabályt adni személyre szabott válaszhoz.

## Magyarázat

A Content-Type a küldött reprezentáció típusát írja le, az Accept a fogadható típusokat jelzi. JSON esetén a sikeres szintaktikai dekódolás nem bizonyítja, hogy a szükséges mezők vagy üzleti értékek helyesek.

A Cache-Control no-store tiltja a válasz tárolását; a no-cache tárolást megengedhet, de újrafelhasználás előtt validálást kér. A private a megosztott cache-ek használatát korlátozza. A Vary azt jelzi, mely kérésfejlécek szerint különböznek a reprezentációk. A személyre szabott joblista cache-kulcsának hibája adatkeveredést okozhat.

## Saját tervezési példa: job állapotának olvasása és módosítása

1. GET /jobs/42 → 200, ETag: "job-42-v3", bodyban a harmadik verzió.
2. Új GET ugyanazzal az If-None-Match értékkel → változatlan állapotnál 304, új body nélkül.
3. Módosítás If-Match: "job-42-v3" feltétellel → csak az ismert verzió alapján engedélyezett.
4. Ha már v4 az állapot → 412; a kliens újraolvas és eldönti, megismételhető-e a változtatás.

Az ETag lehet opaque verzióazonosító; a kliens ne fejtse vissza üzleti adatként. Az If-Match összehasonlításához erős validator kell. A W/ előtagú gyenge ETag más szemantika, nem cserélhető be automatikusan írási konkurenciavédelemhez.

## Atomikusság az adapterben

A backendben a feltétel ellenőrzése és a módosítás legyen egy összehangolt művelet. Például UPDATE ... WHERE id = ... AND version = ... után a módosított sorok száma mutatja a sikert. Ha előbb külön lekérdezed a verziót és utána feltétel nélkül írsz, közben másik kérés is módosíthat; az HTTP-fejléc önmagában nem oldja meg a race conditiont.

## Headerkezelés és tipikus hibák

A fejlécnevek nem kis-/nagybetű-érzékenyek. Ne kezeld minden ismételt header értékét automatikusan egyszerű vesszős listaként: például a Set-Cookie külön szabályokat igényel. Token vagy session cookie ne kerüljön diagnosztikai dumpba.

A 304 nem hibás vagy üres JSON-dokumentum; a kliens a korábbi reprezentációját használja. A 204 sem JSON-bodyval visszatérő siker. A no-cache ne legyen a „sehol ne tárold” véletlen szinonimája.

Senior döntésként mondd ki, mit szeretnél: kevesebb letöltött adatot, ritkább backendhívást vagy írási konkurenciavédelmet. A három cél külön mechanizmust és eltérő teszteket kívánhat.

## Interview questions

**What is the difference between no-cache and no-store?**

Válaszvázlat: Az előbbi újrafelhasználás előtti validálást ír elő, az utóbbi a tárolást tiltja.

**Does checking If-Match in Python prevent lost updates?**

Válaszvázlat: Csak akkor, ha az állapotellenőrzés és az írás atomikusan kapcsolódik a tárolóban; külön read és write race conditiont hagyhat.

## Önellenőrzés

- [ ] A 304 és 204 választ nem próbálom JSON-ként olvasni.
- [ ] Megnevezem az állapotverzió atomikus ellenőrzésének helyét.

## Kapcsolódó gyakorlat

[H02 – önálló feladat](../../../exercises/senior-python/08-http-and-backend.md#h02)

## Forrás és továbbolvasás

[HTTP caching](https://www.rfc-editor.org/rfc/rfc9111.html)

[HTTP conditional requests](https://www.rfc-editor.org/rfc/rfc9110.html#name-conditional-requests)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
