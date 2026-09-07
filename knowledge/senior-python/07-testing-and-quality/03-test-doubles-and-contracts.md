# Fake, mock és szerződéstesztek

## Mit kell tudnod?

- A test double típusát a vizsgált kockázathoz választani.
- Autospec használatával és helyes patch-célponttal csökkenteni a hamis biztonságot.
- Közös szerződéssel ellenőrizni a fake és a valódi adapter egyezését.

## Magyarázat

A stub előre megadott választ szolgáltat; a fake működő, egyszerűsített implementáció; a spy rögzíti a történteket; a mockkal a szükséges interakciót is ellenőrizheted. Az elnevezések keveredhetnek, ezért interjún inkább mondd el, mi valódi a tesztben és mi helyettesített.

Egy in-memory repository jó a felhasználási esethez. Nem modellezi automatikusan az adatbázis izolációját, egyediségét vagy tranzakcióját. A fake is eltérhet a szerződéstől, ezért ugyanazokat az adapterteszteket érdemes később rajta és a valódi adapteren is futtatni.

## Kódpélda: csak a szerződés szerint fontos hívást ellenőrizzük

A teljes blokk pytest-tesztmodul.

```python
from unittest.mock import create_autospec
import pytest

class Mailer:
    def send(self, address: str, subject: str) -> None:
        raise NotImplementedError

def notify(address: str, mailer: Mailer) -> None:
    if not address:
        raise ValueError('address required')
    mailer.send(address, 'Job finished')

def test_notification_is_sent_once():
    mailer = create_autospec(Mailer, instance=True, spec_set=True)
    notify('worker@example.test', mailer)
    mailer.send.assert_called_once_with('worker@example.test', 'Job finished')

def test_invalid_address_has_no_side_effect():
    mailer = create_autospec(Mailer, instance=True, spec_set=True)
    with pytest.raises(ValueError, match='address required'):
        notify('', mailer)
    mailer.send.assert_not_called()

def test_delivery_failure_is_not_swallowed():
    mailer = create_autospec(Mailer, instance=True, spec_set=True)
    failure = OSError('delivery unavailable')
    mailer.send.side_effect = failure
    with pytest.raises(OSError) as caught:
        notify('worker@example.test', mailer)
    assert caught.value is failure
```

Az autospec a valódi hívási szignatúrát követi, a spec_set nem enged tetszőleges új attribútumot. Ettől még nem történik e-mail-küldés, és a típusannotációk runtime betartását sem bizonyítja a mock.

## Patch és szerződések

Ott patch-elj, ahol a kód feloldja a nevet. Ha a service modul `from gateway import send` importot használ, a service.send nevet cseréled. A gateway.send utólagos cseréje nem módosítja a már importált referenciát. Egyszerű esetben a példabeli paraméteres dependency injection érthetőbb a patch-nél.

Egy repository-szerződés tesztmátrixa tartalmazzon hiányzó rekordot, mentés utáni visszaolvasást és dokumentált duplikációkezelést. A fake és a valódi adapter azonos eredményt vagy hibakategóriát adjon. A valódi adatbázisos futtatás továbbra is külön integrációs teszt.

## Tipikus hibák

Ne másold a teljes belső hívási láncot mock-assertionökbe: egy helyes refaktorálás is eltörné. A külső mellékhatás elmaradása viszont érdemi szerződés lehet. A mindenre értéket adó korlátlan Mock elfedhet egy elírt metódusnevet vagy hiányzó awaitet is; async függőséghez szükség esetén AsyncMock és await-ellenőrzés kell.

## Interview questions

**When is a fake preferable to a mock?**

Válaszvázlat: Állapotátmenetek és több híváson átívelő viselkedés esetén olvashatóbb lehet; a fake egyezését külön szerződésteszttel kell védeni.

**Where should you patch an imported dependency?**

Válaszvázlat: A használó modul névfeloldási helyén. Magyarázd el a from-import által létrehozott referencia szerepét.

## Önellenőrzés

- [ ] Nem ellenőrzök lényegtelen belső hívássorrendet.
- [ ] A fake korlátait és a valódi integrációs ellenőrzést külön megnevezem.

## Kapcsolódó gyakorlat

[Q03 – önálló feladat](../../../exercises/senior-python/07-testing-and-quality.md#q03)

## Forrás és továbbolvasás

[unittest.mock](https://docs.python.org/3.12/library/unittest.mock.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
