# Authentication, authorization és erőforrás-hozzáférés

## Mit kell tudnod?

- A hitelesített identitást különválasztani a műveleti jogosultságtól.
- Tenant- és objektumszintű ellenőrzést minden érintett útvonalon végezni.
- A token, session és böngészős védelem szerepét helyesen elhelyezni.

## Magyarázat

Authentication: milyen ellenőrzött identitás képviseli a kérést? Authorization: ez az identitás elvégezheti-e ezt a műveletet ezen az erőforráson? Egy sikeresen ellenőrzött token nem ad automatikus hozzáférést minden jobhoz.

A bearer token birtokosa használni tudja a tokent, ezért bizalmas adat. Az átviteli és naplózási útvonalnak ennek megfelelően kell kezelnie. JWT-nél a dekódolás önmagában nem ellenőrzés: megbízható kulcs, megengedett algoritmus, issuer, audience és érvényességi feltételek szükségesek, a választott token-szerződés szerint. Éles validálást kipróbált könyvtár végezzen; itt nem írunk saját JWT-verifikátort.

## Kódpélda: tiszta objektumszintű policy

A principal ebben a példában már hitelesített, megbízható adat. Nem a request bodyból építjük; az admin szerepet sem a kliens bemondása alapján fogadjuk el.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Principal:
    user_id: str
    tenant_id: str
    roles: frozenset[str]

@dataclass(frozen=True)
class Job:
    owner_id: str
    tenant_id: str

def may_read(principal: Principal, job: Job) -> bool:
    if principal.tenant_id != job.tenant_id:
        return False
    return principal.user_id == job.owner_id or 'tenant_admin' in principal.roles

job = Job('alice', 'tenant-a')
assert may_read(Principal('alice', 'tenant-a', frozenset()), job)
assert not may_read(Principal('bob', 'tenant-a', frozenset()), job)
assert may_read(Principal('bob', 'tenant-a', frozenset({'tenant_admin'})), job)
assert not may_read(Principal('bob', 'tenant-b', frozenset({'tenant_admin'})), job)
```

A tenant határ még az admin szerepnél is megmarad. Ez saját demonstrációs policy; egy valódi rendszer szerepei és delegációi a követelményből következnek. Lista- és keresőendpointnál a jogosultsági szűrés a lekérdezésbe is kerüljön, lapozás előtt, különben rekordok és darabszámok szivároghatnak vagy hiányos oldalak keletkezhetnek.

## Válasz és böngészős kontextus

Hiányzó/érvénytelen hitelesítésnél tipikusan 401 és a használt séma szerinti WWW-Authenticate challenge a válasz. Hitelesített, de tiltott hozzáférésnél 403; egy API szándékosan 404-et is választhat a létezés elrejtésére. Ez legyen következetes szerződés, és a belső napló továbbra is különböztesse meg az okokat.

Cookie-alapú sessionnél a böngésző automatikus credential-küldése miatt CSRF-védelemre is gondolni kell. A CORS böngészős eredetszabály, nem backend authorization: egy nem böngészős kliens nem lesz tőle letiltva. Az XSS és a token tárolási helye külön fenyegetés; egyik credential-forma sem általánosan sérthetetlen.

## Tipikus hibák

Ne tekintsd titoknak az objektumazonosítót, amely önmagában véd a hozzáféréstől. A nehezen kitalálható UUID sem helyettesíti a policyt. Ne bízz a requestben küldött tenant_id vagy role értékben. A policy tesztmátrixa tartalmazzon saját objektumot, másik tulajdonost, helyi admint és másik tenant adminját is.

## Interview questions

**Why is successful authentication insufficient?**

Válaszvázlat: A hitelesített személy jogosultságát a műveletre és az adott objektumra is ellenőrizni kell, tenant-határral együtt.

**Is CORS an authorization mechanism?**

Válaszvázlat: Nem; böngészős eredetszabály. A backend minden kliensnél önállóan ellenőrzi a jogosultságot.

## Önellenőrzés

- [ ] Másik tenant adminját is elutasítom a saját policyben.
- [ ] A token dekódolását nem nevezem hitelesítésnek.

## Kapcsolódó gyakorlat

[H05 – önálló feladat](../../../exercises/senior-python/08-http-and-backend.md#h05)

## Forrás és továbbolvasás

[Bearer token usage](https://www.rfc-editor.org/rfc/rfc6750.html)

[OWASP authorization guidance](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
