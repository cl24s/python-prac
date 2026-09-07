# SSRF, jogosultsági határok és érzékeny adatok

## Mit kell tudnod?

- Felismerni, amikor a szerver hálózati jogosultsága külső inputon át elérhető.
- Többrétegű URL-letöltési védelmet tervezni.
- A publikus választ az internális hibaadatoktól külön kialakítani.

## Magyarázat

SSRF-nél a támadó a szervert használja kérésindításra. A szerver elérhet belső szolgáltatást vagy cloud metadata végpontot, amely a támadótól közvetlenül nem elérhető. Az endpoint checker és a webhook-funkció ezért hálózati bizalmi határ, nem egyszerű URL-validáció.

Ha az üzleti funkció megengedi, a kliens célazonosítót válasszon, amelyet a szerver előre jóváhagyott célra képez. Tetszőleges URL esetén a séma, port, hostname, összes feloldott IP és minden redirect célja vizsgálandó. Egy DNS-ellenőrzés utáni második feloldás más IP-t adhat: a validált cím és a tényleges kapcsolat összekötése a transport feladata. A hálózati egress tiltás külön védelmi réteg.

## Konkrét fenyegetési áttekintés

| Input / esemény | Lehetséges baj | Tervezési válasz |
| --- | --- | --- |
| Nyilvános URL belső címre redirectel | Belső szolgáltatás elérése | Redirect tiltása vagy minden hop teljes ellenőrzése |
| Host több IP-re oldódik | Tiltott cím is választható | Összes cím és tényleges kapcsolati cél ellenőrzése |
| Nagy, tömörített válasz | Memória- és időkeret kimerítése | Stream, kibontott méret és teljes deadline korlát |
| URL-be írt credential | Secret logba vagy továbbküldésbe kerül | Userinfo tiltása, headerátadás explicit szabálya |
| Belső kivétel visszaadása | DB-cím vagy adat kiszivárgása | Külön publikus hibaazonosító és privát diagnosztika |

Nem adunk félkész „biztonságos URL” függvényt: a szövegellenőrzés önmagában nem oldja meg a DNS/transport/redirect problémát. A feladatban a rétegek szerződését kell megtervezni és fake transporttal vizsgálni; valódi belső címet nem kell lekérni.

## Kódpélda: elkülönített publikus hibaválasz

```python
from dataclasses import dataclass

@dataclass
class Failure:
    code: str
    internal_detail: str

PUBLIC_MESSAGES = {'dependency_unavailable': 'Please try again later.'}

def public_error(failure: Failure, incident_id: str):
    if failure.code in PUBLIC_MESSAGES:
        code = failure.code
    else:
        code = 'internal_error'
    return {'error': code,
            'message': PUBLIC_MESSAGES.get(code, 'Unexpected error.'),
            'incident_id': incident_id}

def test_internal_details_stay_out_of_response():
    failure = Failure('dependency_unavailable', 'password=example-only host=private-db')
    response = public_error(failure, 'incident-1')
    assert response == {'error': 'dependency_unavailable',
                        'message': 'Please try again later.', 'incident_id': 'incident-1'}
    assert 'example-only' not in str(response)

def test_unknown_code_is_not_echoed():
    assert public_error(Failure('private-table-name', 'detail'), 'i2')['error'] == 'internal_error'
```

Az incident ID-t élesben a szerver generálja; a példa rögzített értéket kap. Az internal_detail itt mesterséges adat: élesben ezt sem szabad ellenőrizetlenül logolni. Explicit mezőlista, redakció, hozzáférési szabály és megőrzési idő kell.

## Senior döntések és tipikus hibák

A request body teljes naplózása kényelmes debug, de jelszót és személyes adatot gyűjthet. Inkább eseményt és szükséges, minimális kontextust rögzíts. A debug hibaválasz élesben legyen kikapcsolva. A jogosultságot minden objektumműveletnél, a hitelesített tenant alapján ellenőrizd; a kliens által beküldött tenant ID nem jogosultsági bizonyíték.

Egy gateway vagy WAF nem helyettesíti az alkalmazás objektumszintű ellenőrzését. A security review kérdése: melyik komponensben válik a nem megbízható adat hálózati, fájl-, DB- vagy végrehajtási jogosultsággá?

## Interview questions

**Why is checking a URL string insufficient against SSRF?**

Válaszvázlat: A DNS, redirect és tényleges kapcsolat más címet érhet el. A transport és egress szabályoknak is ugyanazt a határt kell védeniük.

**How do you keep useful diagnostics without leaking secrets?**

Válaszvázlat: Publikus hibakód és szerveroldali korreláció, explicit logmezők, minimális adat és korlátozott hozzáférés.

## Önellenőrzés

- [ ] Végig tudom követni a validált címet a kapcsolatig.
- [ ] A belső kivétel nem lesz közvetlen HTTP-válasz.

## Kapcsolódó gyakorlat

[OP07 – önálló feladat](../../../exercises/senior-python/13-packaging-security-and-operations.md#op07)

## Forrás és továbbolvasás

[OWASP SSRF prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)

[OWASP logging](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
