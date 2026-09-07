# pytest: fixture-ök, parametrizálás és exceptionök

## Mit kell tudnod?

- Átlátható fixture-életciklust és izolált állapotot kialakítani.
- Határértékeket parametrizálni beszédes esetnevekkel.
- Pontos kivételt és hibát okozó műveletet ellenőrizni.

## Magyarázat

A fixture a teszt függőségeit készíti elő. Az alapértelmezett function scope minden tesztesethez külön példányt biztosít. A module/session scope drága, megosztható erőforrásoknál hasznos, de egy módosítható közös állapot sorrendfüggő hibát hozhat létre. A függőségi gráf legyen kicsi: egy mindent előkészítő autouse fixture elrejti, mit használ a teszt.

A yield előtti rész a setup, az utána következő a teardown. Ha a fixture már a yield előtt hibázik, a saját yield utáni része nem fut le; a korábban sikeresen felállt fixture-ök takarítása ettől még megtörténik. Ezért a megszerzett erőforrást rögtön védd finally/context manager segítségével.

## Kódpélda: határérték és fájlerőforrás

Futtatás: a teljes blokk `test_example.py` fájlként, majd `python -m pytest -q test_example.py`.

```python
import pytest

def parse_limit(text: str) -> int:
    limit = int(text)
    if not 1 <= limit <= 100:
        raise ValueError('limit out of range')
    return limit

@pytest.mark.parametrize('text, expected', [('1', 1), ('100', 100), (' 7 ', 7)],
                         ids=['minimum', 'maximum', 'whitespace'])
def test_valid_limit(text, expected):
    assert parse_limit(text) == expected

@pytest.mark.parametrize('text', ['0', '101', '-1'])
def test_limit_out_of_range(text):
    with pytest.raises(ValueError, match='^limit out of range$'):
        parse_limit(text)

def test_malformed_limit():
    with pytest.raises(ValueError):
        parse_limit('many')

@pytest.fixture
def output_file(tmp_path):
    with (tmp_path / 'output.txt').open('w+', encoding='utf-8') as stream:
        yield stream

def test_file_is_available(output_file):
    output_file.write('ready')
    output_file.seek(0)
    assert output_file.read() == 'ready'
```

A parametrizálás itt hét parseresetet hoz létre; a fájlteszttel együtt nyolc teszt fut. A fixture with blokkja a teszt hibája esetén is lezárja a fájlt. A beépített tmp_path elkülönített útvonalat ad; nem feltételezzük, hogy a könyvtár azonnal eltűnik a teszt végén.

## Senior döntések és tipikus hibák

A `pytest.raises(Exception)` túl tág: egy NameError is zölddé teheti. A raises blokkban csak a hibára várt művelet legyen, az eredményellenőrzés utána következzen. A match reguláris kifejezés, nem egyszerű szövegegyenlőség.

A beépített caplog a naplót, capsys a standard kimenetet, monkeypatch a környezet vagy attribútum ideiglenes változtatását segít ellenőrizni. A változtatás visszaállítása nem tesz egy globális állapotot biztonságossá párhuzamos használatra.

## Interview questions

**When would you use a session-scoped fixture?**

Válaszvázlat: Drága, tudatosan megosztható erőforrásnál, külön tesztadat-izolációval. A szélesebb scope nem puszta gyorsítási kapcsoló.

**What happens if fixture setup fails before yield?**

Válaszvázlat: A fixture yield utáni teardownja kimarad; a már felállt függőségek takarítása lefut. Részleges setupnál korán kell védeni az erőforrást.

## Önellenőrzés

- [ ] Határérték és hibás formátum is szerepel.
- [ ] A setup részleges hibájára is gondolok.

## Kapcsolódó gyakorlat

[Q02 – önálló feladat](../../../exercises/senior-python/07-testing-and-quality.md#q02)

## Forrás és továbbolvasás

[pytest fixtures](https://docs.pytest.org/en/stable/how-to/fixtures.html)

[Parametrization](https://docs.pytest.org/en/stable/how-to/parametrize.html)

[Assertions and exceptions](https://docs.pytest.org/en/stable/how-to/assert.html)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
