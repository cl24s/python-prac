# Hitelesítési dependency és objektumszintű authorization

## Mit kell tudnod?

- A bearer token kiolvasását elkülöníteni a token ellenőrzésétől.
- Hitelesített principalból jogosultsági döntést hozni.
- Tesztben is ellenőrizni a 401, 403 és tenant-határokat.

## Magyarázat

A FastAPI security segédei a credential kiolvasását és az OpenAPI security leírását segítik. Egy OAuth2PasswordBearer vagy HTTPBearer önmagában nem ellenőrzi a JWT aláírását, issuerét vagy audience-ét. Ehhez külön, ellenőrzött verifier kell, a használt identitásszolgáltató szerződése szerint.

A hitelesítés eredménye egy megbízható principal. A request body role vagy tenant mezője nem válik ettől megbízhatóvá. A policy azt vizsgálja, hogy ez a principal az adott erőforráson elvégezheti-e a műveletet.

## Kódpélda: demonstrációs verifier és valódi HTTP-határteszt

A token-tábla kizárólag tesztadat. Nem JWT-megoldás, nem éles credential-kezelés.

```python
from dataclasses import dataclass
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer
from fastapi.testclient import TestClient

@dataclass(frozen=True)
class Principal:
    user_id: str
    tenant_id: str

app = FastAPI()
bearer = HTTPBearer(auto_error=False)
TOKENS = {'demo-owner': Principal('alice', 'a'),
          'demo-other': Principal('bob', 'a'),
          'demo-tenant': Principal('alice', 'b')}

def current_user(credentials: Annotated[HTTPAuthorizationCredentials | None,
                                        Depends(bearer)]):
    principal = TOKENS.get(credentials.credentials) if credentials else None
    if principal is None:
        raise HTTPException(401, 'Invalid credentials', headers={'WWW-Authenticate': 'Bearer'})
    return principal

@app.get('/jobs/1')
def read_job(principal: Annotated[Principal, Depends(current_user)]):
    if (principal.tenant_id, principal.user_id) != ('a', 'alice'):
        raise HTTPException(403, 'Access denied')
    return {'id': 1}

def test_authentication_and_policy():
    with TestClient(app) as client:
        response = client.get('/jobs/1')
        assert response.status_code == 401
        assert response.headers['WWW-Authenticate'] == 'Bearer'
        for token in ['demo-other', 'demo-tenant']:
            assert client.get('/jobs/1', headers={'Authorization': f'Bearer {token}'}).status_code == 403
        assert client.get('/jobs/1', headers={'Authorization': 'Bearer invalid'}).status_code == 401
        assert client.get('/jobs/1', headers={'Authorization': 'Bearer demo-owner'}).json() == {'id': 1}
```

A resource ebben a demóban rögzített. Éles repositoryban a tenant-szűrés a lekérdezésbe is kerüljön, de az adott művelet policyje továbbra is szükséges. A listaendpoint ellenőrzése lapozás előtt történjen.

## Senior döntések és tipikus hibák

A JWT-verifier megbízható kulcsból és megengedett algoritmusból induljon, ellenőrizze a szükséges claim-eket, lejáratot és kulcsrotációt. Az OpenAPI-ban megjelenő lakat nem bizonyítja ezt a működést. A scope deklaráció sem automatikusan üzleti jogosultság: a szükséges és tényleges jogokat össze kell vetni.

Nem böngészős kliens ellen a CORS nem védelem. Cookie-session esetén CSRF, bearer token esetén a tárolási és továbbítási út is releváns. A security dependency override hasznos domain/HTTP-teszthez, de ha minden tesztben felülírod, a valódi auth-bekötést sosem teszteled.

## Interview questions

**Does HTTPBearer validate a JWT?**

Válaszvázlat: Nem; credentialt olvas ki és security-sémát ír le. Külön verifier ellenőrzi a token hitelességét és claimjeit.

**What is lost if every test overrides authentication?**

Válaszvázlat: A valódi hitelesítési határ, hibás tokenek és policy-bekötés ellenőrzése; ezekhez külön teszt kell.

## Önellenőrzés

- [ ] A példabeli tokeneket tesztadatként kezelem.
- [ ] A más tenantba tartozó azonos user ID sem kap hozzáférést.

## Kapcsolódó gyakorlat

[F07 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f07)

## Forrás és továbbolvasás

[FastAPI security tools](https://fastapi.tiangolo.com/reference/security/)

[OAuth2 scopes](https://fastapi.tiangolo.com/advanced/security/oauth2-scopes/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
