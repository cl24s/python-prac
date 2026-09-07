# Pydantic v2: validáció, szerializáció és részleges frissítés

## Mit kell tudnod?

- A coercion és a strict validáció közti döntést megindokolni.
- Külön bemeneti és kimeneti modelleket használni.
- Elkülöníteni a hiányzó és explicit null mezőt PATCH esetén.

## Magyarázat

A type hint önmagában nem validál külső adatot. A Pydantic modell létrehozása már runtime ellenőrzést és a beállítások szerinti átalakítást végez. A szöveges "3" normál int mezővé alakítható, strict int esetén elutasítható. A döntés az API-szerződés része; az „intként jött ki” nem bizonyítja, hogy intként érkezett.

Bejövő modellben csak a kliens által állítható mezők legyenek. A tenant_id, belső státusz vagy admin szerep ne váljon véletlenül mass assignment bemenetté. Az extra='forbid' elírt vagy tiltott mezőt is láthatóvá tesz; más API-knál az extra mezők figyelmen kívül hagyása kompatibilitási döntés lehet.

## Kódpélda: explicit szabályok és PATCH-jelenlét

```python
import pytest
from pydantic import BaseModel, ConfigDict, Field, ValidationError

class CreateJob(BaseModel):
    model_config = ConfigDict(extra='forbid')
    name: str = Field(min_length=1, max_length=80)
    attempts: int = Field(default=1, ge=1, le=5, strict=True)

class JobPatch(BaseModel):
    model_config = ConfigDict(extra='forbid')
    description: str | None = None

def test_input_constraints():
    assert CreateJob(name='report').model_dump() == {'name': 'report', 'attempts': 1}
    for bad in ['2', True, 0, 6]:
        with pytest.raises(ValidationError):
            CreateJob(name='report', attempts=bad)
    with pytest.raises(ValidationError):
        CreateJob(name='report', tenant_id='other')

def test_patch_presence():
    omitted = JobPatch.model_validate({})
    cleared = JobPatch.model_validate({'description': None})
    assert omitted.model_dump(exclude_unset=True) == {}
    assert cleared.model_dump(exclude_unset=True) == {'description': None}
    assert cleared.model_fields_set == {'description'}
```

A példában a null törlési szándék, a hiányzó mező „ne változtasd”. Az exclude_none=True összemosná ezt a két esetet. A str | None annotáció alapérték nélkül lehet kötelező, de nullt elfogadó mező; az opcionális jelenlét és a nullable érték nem ugyanaz.

## Senior döntések és tipikus hibák

A model_validate Python-adatot ellenőriz, a model_dump strukturált adatot ad, a model_dump_json JSON-szöveget. JSON-kompatibilis Python-adathoz model_dump(mode='json') használható. Ne serializálj kétszer: egy JSON-string újabb JSON-válaszba téve idézőjelezett szöveg lesz objektum helyett.

Field validatorral helyi normalizálás vagy értékszabály, model validatorral több mező közti feltétel fejezhető ki. Ne rejts hálózati vagy adatbázis-hívást a modellvalidációba: az üzleti művelet külön réteg. A név min_length=1 szabálya önmagában nem tiltja a csak szóközökből álló nevet; ha ez követelmény, külön normalizálás és teszt kell.

Az ORM-attribútumokból épített modell from_attributes beállítása kényelmes lehet, de egy attribútum olvasása lazy SQL-t is indíthat. A szerializáció előtt tudd, mely adat van már betöltve.

## Interview questions

**What is the difference between omitted and null in a PATCH body?**

Válaszvázlat: Az elhagyott mező nem kér változást, az explicit null saját szerződés szerint törölhet. A model_fields_set/exclude_unset megőrzi a különbséget.

**Where should database-dependent validation live?**

Válaszvázlat: A felhasználási esetben és a tárolói constraintben, nem rejtett I/O-ként a Pydantic-validátorban.

## Önellenőrzés

- [ ] A bool/int és coercion szélső esetét tesztelem.
- [ ] A hiányzó és null értéket nem mosom össze.

## Kapcsolódó gyakorlat

[F02 – önálló feladat](../../../exercises/senior-python/09-fastapi.md#f02)

## Forrás és továbbolvasás

[Pydantic models](https://docs.pydantic.dev/latest/concepts/models/)

[Pydantic strict mode](https://docs.pydantic.dev/latest/concepts/strict_mode/)

[Pydantic serialization](https://docs.pydantic.dev/latest/concepts/serialization/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
