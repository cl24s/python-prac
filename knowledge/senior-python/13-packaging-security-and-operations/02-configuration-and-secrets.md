# Konfiguráció, secret-kezelés és indulási validáció

## Mit kell tudnod?

- A konfigurációt egyszer, ellenőrzött határon beolvasni.
- Secretet távol tartani a reprtől, logtól és hibaválasztól.
- Rotációt és hibás konfigurációt működési eseményként kezelni.

## Magyarázat

A konfiguráció az alkalmazás viselkedésének bemenete. Legyen dokumentált forrása és prioritása: például alapérték, konfigurációs fájl, környezeti felülírás. A titok hiányát ne helyettesítsd fejlesztői jelszóval éles környezetben. Az alkalmazás induláskor alakítsa típusos értékekké az adatokat, és még a forgalom fogadása előtt bukjon el hibás kötelező beállítással.

Ne olvass minden kérésben környezeti változókat. Egy explicit settings objektum könnyebben tesztelhető és világossá teszi, mikor lép életbe a módosítás. A dinamikus reload külön életciklus-probléma: mi történik a meglévő DB-kapcsolatokkal credentialrotáció után?

## Kódpélda: tiszta konfigurációs határ

```python
from dataclasses import dataclass, field
from collections.abc import Mapping
import math
import pytest

@dataclass(frozen=True)
class Settings:
    timeout: float
    api_key: str = field(repr=False)

def read_settings(values: Mapping[str, str]) -> Settings:
    key = values.get('API_KEY', '')
    if not key.strip():
        raise ValueError('API_KEY is required')
    try:
        timeout = float(values.get('TIMEOUT_SECONDS', '2'))
    except ValueError:
        raise ValueError('TIMEOUT_SECONDS must be numeric') from None
    if not math.isfinite(timeout) or not 0 < timeout <= 30:
        raise ValueError('TIMEOUT_SECONDS must be in (0, 30]')
    return Settings(timeout, key)

def test_valid_config_hides_key_in_repr():
    settings = read_settings({'API_KEY': 'example-only', 'TIMEOUT_SECONDS': '1.5'})
    assert settings.timeout == 1.5
    assert 'example-only' not in repr(settings)

@pytest.mark.parametrize('timeout', ['nan', 'inf', '0', '-1', '31', 'bad'])
def test_invalid_timeout(timeout):
    with pytest.raises(ValueError, match='TIMEOUT_SECONDS'):
        read_settings({'API_KEY': 'example-only', 'TIMEOUT_SECONDS': timeout})

def test_missing_key():
    with pytest.raises(ValueError, match='API_KEY is required'):
        read_settings({})
```

A példa csak a megadott mappinget olvassa; nincs globális környezetmódosítás vagy valódi secret. A `repr=False` egy véletlen naplózási utat zár le. Az `asdict`, a közvetlen attribútum-hozzáférés és egy hibás logger továbbra is kiolvashatja az értéket. A frozen dataclass sem titkosítás.

## Üzemeltetési döntések

A környezeti változó praktikus beadási mód, de nem secret-vault: processzdump, diagnosztikai kimenet vagy túl széles hozzáférés kiszivárogtathatja. Fájlból beadott secret esetén jogosultság, mount és rotációs stratégia kell. Secret manager esetén a hozzáférés rövid életű azonossághoz és minimális jogosultsághoz kötődjön; a lekérés hibájára is legyen indulási vagy megújítási szabály.

A rotációhoz átmenetileg két credential elfogadása vagy kapcsolatcsere is kellhet. Dokumentáld a lejárat előtti megújítást, a régi credential visszavonását és az ellenőrzést. A Gitből kitörölt secret a történetből és másolatokból nem tűnik el: kiszivárgáskor visszavonás szükséges, nem csak fájltörlés.

## Interview questions

**Where should configuration validation happen?**

Válaszvázlat: Az alkalmazás összeállításakor, forgalomfogadás előtt; explicit típuskonverzió és érthető, titokmentes hibák mellett.

**Does repr=False protect a secret?**

Válaszvázlat: Csak az automatikus reprből rejti el. Más szerializáló, dump és log továbbra is hozzáférhet.

## Önellenőrzés

- [ ] A hibás és hiányzó konfigurációra is van tervem.
- [ ] A rotációt a meglévő kapcsolatokra is végiggondolom.

## Kapcsolódó gyakorlat

[OP02 – önálló feladat](../../../exercises/senior-python/13-packaging-security-and-operations.md#op02)

## Forrás és továbbolvasás

[Python dataclasses](https://docs.python.org/3.12/library/dataclasses.html)

[OWASP secrets management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
