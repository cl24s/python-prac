# TypedDict, dataclass és runtime validáció

## Mit kell tudnod?

- Külső adat formáját és belső értékmodelljét külön kezelni.
- A hiányzó kulcsot és a None-értéket megkülönböztetni.
- Megválasztani, hol legyen statikus leírás, parse-olás és üzleti validáció.

## Modellválasztás

| Eszköz | Mire való? | Mit nem végez automatikusan? |
| --- | --- | --- |
| `TypedDict` | Dict kulcsainak és értékeinek statikus leírása | Runtime kulcs- és értékellenőrzés |
| `dataclass` | Névvel ellátott belső adatobjektum | Teljes típusvalidáció, deep immutability |
| Egyszerű domainosztály | Állapot és üzleti műveletek együtt | Külső JSON automatikus feldolgozása |
| Pydantic modell | Deklarált runtime parse/validáció | Minden üzleti invariáns vagy jogosultság |

A TypedDict runtime közönséges dict. A `NotRequired[str]` azt jelenti, hogy a kulcs hiányozhat; a `str | None` azt, hogy az érték lehet None. A kettő kombinálható, de eltérő szemantikát ír le.

## Kódpélda: alak és értékmodell elválasztása

```python
from collections.abc import Mapping
from dataclasses import dataclass
from typing import NotRequired, TypedDict

class JobPayload(TypedDict):
    name: str
    retries: NotRequired[int]

@dataclass(frozen=True)
class JobRequest:
    name: str
    retries: int

    def __post_init__(self) -> None:
        if not self.name.strip() or self.retries < 0:
            raise ValueError('invalid job request')

def parse_request(raw: Mapping[str, object]) -> JobRequest:
    name = raw.get('name')
    retries = raw.get('retries', 0)
    if not isinstance(name, str):
        raise TypeError('name must be text')
    if isinstance(retries, bool) or not isinstance(retries, int):
        raise TypeError('retries must be an integer')
    return JobRequest(name=name.strip(), retries=retries)

payload: JobPayload = {'name': 'report'}
assert 'retries' not in payload
request = parse_request({'name': ' report ', 'retries': 2})
assert request == JobRequest('report', 2)
try:
    parse_request({'name': 'report', 'retries': '2'})
except TypeError:
    pass
else:
    raise AssertionError('Numeric text is not accepted by this parser')
```

A parser itt tudatosan nem konvertál numerikus stringet intté. A többletkulcsokat figyelmen kívül hagyja; ha tiltani kell, ezt külön ellenőrizni kell. A parser Mapping bemenetet feltételez: egy tetszőleges JSON-gyökér listáját a külső rétegnek még ellenőriznie kell.

## Rövid Pydantic-kitekintés

Az alábbi önálló blokk Pydantic 2.x-et igényel. A FastAPI-fejezet később részletesen feldolgozza a modelleket. A `strict=True` itt elkerüli a példában a stringből számmá konvertálást; az `extra='forbid'` a váratlan mezőt tiltja.

```python
from pydantic import BaseModel, ConfigDict, Field, ValidationError

class RetryInput(BaseModel):
    model_config = ConfigDict(strict=True, extra='forbid')
    retries: int = Field(ge=0)

assert RetryInput.model_validate({'retries': 2}).retries == 2
for raw in [{'retries': '2'}, {'retries': -1}, {'retries': 2, 'debug': True}]:
    try:
        RetryInput.model_validate(raw)
    except ValidationError:
        pass
    else:
        raise AssertionError('Invalid payload must be rejected')
```

A parse sikeressége nem ad jogot műveletvégzésre. A felhasználó jogosultsága, a rendszer kapacitása és az állapotátmenet külön szabály. A modell további mutálhatóságát és az assignment-validációt szintén tudatosan kell beállítani; a konstruktor egyszeri ellenőrzése nem feltétlenül tart örökké.

## Tipikus hibák és senior szempontok

- **Hiba:** JSON-t `cast(JobPayload, raw)`-val „validálsz”. **Javítás:** tényleges parser vagy validátor.
- **Hiba:** minden réteg ugyanazt a külső modellt használja. **Javítás:** ahol más a felelősség és a szerződés, ott indokolt belső modellre fordítani.
- Ne hozz létre azonos modelleket csak a rétegszám kedvéért; az eltérő szabály vagy független változás indokolja a másolatot.

## Interview questions

**How does TypedDict differ from a runtime validation model?**

Válaszvázlat: statikus dict-forma versus tényleges parse és validáció; runtime a TypedDict nem véd meg a hibás inputtól.

**Where should business invariants be enforced?**

Válaszvázlat: a megfelelő domain/use-case határon, minden releváns módosító úton; a JSON-séma és típus önmagában kevés.

## Önellenőrzés

- [ ] Megkülönböztetem a hiányzó retries kulcsot a None-tól.
- [ ] Dokumentálom a konverzió és extra kulcsok szabályát.
- [ ] Nem használok castot inputvalidáció helyett.


## Kapcsolódó gyakorlat

[T05 – önálló feladat](../../../exercises/senior-python/04-types-and-interfaces.md#t05)

## Forrás és továbbolvasás

[TypedDict specification](https://typing.python.org/en/latest/spec/typeddict.html) · [Pydantic models](https://docs.pydantic.dev/latest/concepts/models/)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
