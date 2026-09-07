# Lapozás, API-verziózás és kompatibilitás

## Mit kell tudnod?

- Determinista rendezéssel és egyedi tie-breakerrel lapozni.
- Megérteni az offset és keyset megoldások változó adatok melletti korlátait.
- Visszafelé kompatibilitást a kliens tényleges szerződése szerint értékelni.

## Magyarázat

Offset lapozásnál az első N sort átugrod. Egyszerű, de nagy offset drága lehet, és közben beszúrt/törölt sorok miatt eltolódhatnak az oldalak. Keyset lapozásnál az utolsó rendezési kulcs után folytatod. Ha created_at szerint rendezel, önmagában az időbélyeg nem feltétlenül egyedi; id tie-breaker is kell.

## Kódpélda: folytatás összetett kulcs után

Ez kis in-memory modell, nem SQL-adapter és nem teljes cursor-API. A rendezési kulcsok az adott bejárás alatt változatlannak tekintettek.

```python
from dataclasses import dataclass

@dataclass(frozen=True, order=True)
class Row:
    created_at: int
    id: int

def page(rows: list[Row], after: tuple[int, int] | None, limit: int):
    if not 1 <= limit <= 100:
        raise ValueError('invalid limit')
    candidates = [r for r in sorted(rows)
                  if after is None or (r.created_at, r.id) > after]
    batch = candidates[:limit]
    has_more = len(candidates) > limit
    cursor = (batch[-1].created_at, batch[-1].id) if batch and has_more else None
    return batch, cursor

rows = [Row(10, 2), Row(11, 3), Row(10, 1)]
first, cursor = page(rows, None, 2)
assert [r.id for r in first] == [1, 2]
assert cursor == (10, 2)
second, cursor = page(rows, cursor, 2)
assert [r.id for r in second] == [3]
assert cursor is None
assert page([], None, 2) == ([], None)
```

Éles adatbázisban a szűrést, rendezést és limit + 1 lekérést az adapter végezze megfelelő indexszel. A fenti sorted és teljes lista ezért csak a szemantikát mutatja, nagy adatra nem hatékony megoldás.

## Cursor és konzisztencia

A külső cursor legyen opaque a kliensnek; a kliens továbbküldi, nem szerkeszti. A szerver ellenőrizze a formátumát és azt, hogy a szűréshez, rendezéshez és jogosultsági hatókörhöz illeszkedik-e. A base64 kódolás nem aláírás és nem titkosítás. A cursorban lévő tenant nem jogosít hozzáférésre.

Keyset mellett sem kapsz automatikusan snapshotot. Az utolsó kulcs elé beszúrt sor kimaradhat az aktuális bejárásból, a mögé beszúrt megjelenhet, a változó rendezési kulcs pedig ismétlést vagy kihagyást okozhat. Ha pontos export kell egy időpontra, külön snapshot-/verzióstratégiát tervezz.

## Kompatibilitási döntések

| Változtatás | Kockázat |
| --- | --- |
| Mező átnevezése vagy eltávolítása | A meglévő kliens olvasása eltörhet |
| Új kötelező bemeneti mező | Régi kérések érvénytelenné válhatnak |
| Új opcionális válaszmező | Toleráns kliensnél működhet, szigorú parsernél nem biztos |
| Új enumérték | Kimerítő elágazású kliens eltörhet |
| Új alapértelmezett rendezés | Lapozás és üzleti megjelenítés változhat |

Az URL-ben lévő v2 csak egy verziózási eszköz. Kell támogatási időszak, migrációs út és szerződésteszt a régi kliensre. A „csak hozzáadtam” nem univerzális kompatibilitási bizonyíték.

## Interview questions

**Why does cursor pagination need a unique ordering key?**

Válaszvázlat: Az azonos időbélyegű rekordok között egyedi tie-breaker nélkül nem egyértelmű a folytatás.

**Does keyset pagination provide snapshot consistency?**

Válaszvázlat: Nem; konkurens beszúrás és a rendezési kulcs változása befolyásolja a bejárást. A snapshot külön garancia.

## Önellenőrzés

- [ ] Azonos időbélyegű sorokkal is ellenőrzöm a lapozást.
- [ ] Új enumértéknél is vizsgálom a régi klienst.

## Kapcsolódó gyakorlat

[H06 – önálló feladat](../../../exercises/senior-python/08-http-and-backend.md#h06)

## Forrás és továbbolvasás

[Google API pagination guidance](https://google.aip.dev/158)

[Google API compatibility guidance](https://google.aip.dev/180)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
