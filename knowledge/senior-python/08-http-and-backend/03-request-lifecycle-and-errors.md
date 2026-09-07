# Request-életciklus, middleware és hibaválaszok

## Mit kell tudnod?

- Rétegenként elhelyezni a validációt és az exception-fordítást.
- Middleware-sorrendet és streaming-korlátokat felismerni.
- Stabil, géppel feldolgozható hibaszerződést kialakítani.

## Magyarázat

Egy kérés útja tipikusan: proxy és szerver → middleware → routing és határvalidáció → felhasználási eset → adapter. A visszaút ugyanazon határokon át ad státuszt, headereket és bodyt. Ez szervezési minta, nem minden framework kötelező belső sorrendje.

A határvalidáció ellenőrzi például a mezőtípust és méretet. Az üzleti réteg dönti el, hogy az adott job újraindítható-e. Az adatbázis egyediségét végül a tároló is kényszerítse ki: a korábbi „létezik-e?” ellenőrzés nem véd konkurens beszúrás ellen.

Middleware-k sorrendje számít: a külső middleware a belépésnél előbb, a válasz feldolgozásánál később fut. A request ID-t a belső naplózás előtt kell előállítani vagy ellenőrzötten átvenni. A minden kérésbodyt teljesen beolvasó naplózó middleware memória- és adatvédelmi hibát okozhat, streamingnél a viselkedést is megváltoztatja.

## Kódpélda: kontrollált fordítás a HTTP-határon

Ez önálló Python-script; nem teljes webszerver. A domainhiba nem ismer HTTP-státuszt.

```python
import json

class JobConflict(Exception):
    pass

def error_response(error: Exception, request_id: str):
    if isinstance(error, JobConflict):
        status, title = 409, 'Conflict'
        detail = 'Job cannot be started in its current state.'
    else:
        status, title = 500, 'Internal Server Error'
        detail = 'The request could not be completed.'
    body = {'type': 'about:blank', 'title': title, 'status': status,
            'detail': detail, 'request_id': request_id}
    return status, {'Content-Type': 'application/problem+json'}, json.dumps(body)

status, headers, raw = error_response(JobConflict('private state'), 'req-17')
assert status == 409
assert headers['Content-Type'] == 'application/problem+json'
assert json.loads(raw)['request_id'] == 'req-17'
status, _, raw = error_response(RuntimeError('secret connection data'), 'req-18')
assert status == 500
assert 'secret connection data' not in raw
```

Az RFC 9457 Problem Details formátum stabil keretet ad a hiba típusának és részleteinek. A request_id itt saját extension mező. Az about:blank általános HTTP-hibát jelöl; saját üzleti hibatípushoz később dokumentált type URI adható. A kliens gépi döntése stabil típusra/kódra épüljön, ne a lokalizálható detail szövegre.

## Hibás utak és senior kompromisszumok

Az ismeretlen hiba 500 marad, a belső részlet szerveroldali diagnosztikába kerül. Ez a kis mapper nem naplóz: a valódi request boundary felelőssége egyszer rögzíteni az exceptiont a kontextussal. A correlation ID összeköt, de nem helyettesíti a teljes trace-t.

Ha a streaming válasz fejléce már kiment, nem tudsz egyszerűen új 500-as JSON-válaszra váltani. A kapcsolat és a stream lezárását, a kliens által észlelhető részleges választ és a naplózást külön kell tervezni. A kliens bontása sem garantálja, hogy a már elindított üzleti mellékhatás visszagördül.

Ne map-elj minden ValueErrort 400-ra: lehet belső programozási hiba is. A validáció helye és a dokumentált kivételtípus alapján fordíts.

## Interview questions

**Where should domain exceptions become HTTP responses?**

Válaszvázlat: A HTTP-határon; a domain ne függjön a framework válaszobjektumától vagy státuszkódjától.

**What if an exception occurs after response headers were sent?**

Válaszvázlat: Új státuszt általában már nem küldhetek; részleges stream és lezárás lesz, külön diagnosztikával és szerződéssel.

## Önellenőrzés

- [ ] Nem kerül belső exception-szöveg automatikusan a klienshez.
- [ ] Megkülönböztetem a mezővalidációt az üzleti és tárolói invariánstól.

## Kapcsolódó gyakorlat

[H03 – önálló feladat](../../../exercises/senior-python/08-http-and-backend.md#h03)

## Forrás és továbbolvasás

[Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)

[ASGI HTTP response messages](https://asgi.readthedocs.io/en/stable/specs/www.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
