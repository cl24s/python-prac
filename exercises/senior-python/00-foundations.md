# Gyakorlati alapozás — L01–L08

Állapot: kidolgozott kiírások; felhasználói megoldás és review nincs rögzítve. Az L prefix nem ütközik a meglévő FastAPI F01–F11 azonosítókkal. Kész megoldások nincsenek mellékelve; az L05 kódrészlete szándékosan hibás bemenet a hibakereséshez.

[Tananyag](../../knowledge/senior-python/00-foundations/README.md) · [Aktuális sorrend](../../ROADMAP.md)

Kezdetben standard library elég; pytest a tesztekhez használható. A fájlfeladatok kizárólag saját ideiglenes, mesterséges adatokkal dolgozzanak. A kiírás által garantált bemenetekhez ne találj ki további rejtett validációs követelményt. A megoldások később `00-foundations/l01/`, `l02/` stb. almappába kerülhetnek.

## L01

**Cél:** kis függvény, feltételek, határértékek és return.

Írj `latency_band(ms)` függvényt. A bemenet garantáltan int, nem bool. Negatív értékre `ValueError` kell. 0–99 esetén `"fast"`, 100–499 esetén `"normal"`, 500-tól `"slow"` legyen az eredmény. A függvény ne írjon stdout-ra.

Példák: `latency_band(0)` → `"fast"`; `latency_band(100)` → `"normal"`; `latency_band(500)` → `"slow"`.

- [ ] Tesztelve: -1, 0, 99, 100, 499, 500.
- [ ] A függvény eredményt ad, nem csak kiír.
- [ ] Elmagyarázod az ágak sorrendjét.

**Explain:** What happens at the exact boundary value?

## L02

**Cél:** stringművelet, ciklus és új lista.

Írj `clean_names(names)` függvényt. A bemenet stringlista. Minden elem elejéről és végéről távolítsd el a whitespace-t, az így üres elemeket hagyd ki. A sorrend, kis-/nagybetű és duplikáció megmarad. A bemeneti lista nem módosul.

Példa: `[" api ", "", "  ", "Worker", "api"]` → `["api", "Worker", "api"]`.

- [ ] Üres lista és csak whitespace-elemek is tesztelve.
- [ ] Két egymás melletti kihagyandó elem sem marad bent.
- [ ] Új listát adsz; a bemenet változatlan.

**Explain:** Why is lowercasing not part of this function?

## L03

**Cél:** dictionary-aggregáció.

Írj `total_by_product(items)` függvényt. A bemenet `(product: str, quantity: int)` párok listája; a név garantáltan nem üres, a quantity pozitív int, nem bool. Eredmény: terméknév → mennyiségek összege. A kis- és nagybetű számít; a bemenet nem módosul. A dict kulcssorrendje nem része az elfogadási feltételeknek.

Példa: `[("apple", 2), ("pear", 1), ("apple", 3)]` → `{"apple": 5, "pear": 1}`.

- [ ] Üres lista üres dictet ad.
- [ ] Azonos termék többször összeadódik, nem felülíródik.
- [ ] Megfogalmazod, mit tárol a dict egy adott cikluslépés után.

**Explain:** Are you counting rows or adding quantities?

## L04

**Cél:** rendezés és formázás különválasztása az összesítéstől.

Írj `format_totals(totals)` függvényt. A bemenet nem üres stringkulcsokat és nem negatív intértékeket tartalmazó dict. Az eredmény stringlista, terméknév szerint növekvő, kis-/nagybetűt megkülönböztető rendezéssel: `"name: quantity"`. Az üres dict üres listát ad. Nem módosíthatod a bemenetet.

Példa: `{"pear": 1, "apple": 5}` → `["apple: 5", "pear: 1"]`.

- [ ] Eltérő beszúrási sorrend ugyanazt a kimenetet adja.
- [ ] Nulla mennyiség is szerepel.
- [ ] Az L03 eredményével összeköthető, de külön is tesztelhető.

**Explain:** Why keep calculation and formatting separate?

## L05

**Cél:** meglévő hibás kód javítása tesztekkel.

Az alábbi függvénynek az első olyan szót kell visszaadnia, amelynek hossza **legalább** `min_length`. Ha nincs ilyen, `None` az eredmény. A words stringlista, min_length garantáltan pozitív int; nincs bemenetmódosítás.

Szándékosan hibás kiinduló kód, nem mintamegoldás:

```python
def first_long_word(words, min_length):
    for word in words:
        if len(word) > min_length:
            return word
        return None
```

Példa: `["a", "boat", "river"]`, 4 → `"boat"`.

- [ ] Javítás előtt készíts tesztet a pontos határértékre és egy később található megfelelő szóra.
- [ ] Üres lista, nincs találat és az első szó megfelelő eset is tesztelve.
- [ ] Írd le a két hiba okát; ne csak egy teljesen más függvényt másolj be.
- [ ] Jegyezd fel a javítás előtti bukást és a javítás utáni teszteredményt.

**Explain:** Which return belongs inside the loop, and which belongs after it?

## L06

**Cél:** fájlkezelés és meglévő tiszta logika összekapcsolása.

Írj `read_names(path)` függvényt. A path string vagy `pathlib.Path`, UTF-8 szövegfájlra mutat. A fájl sorait az L02 szabálya szerint dolgozd fel: trim, üres sorok kihagyása, sorrend és ismétlődések megőrzése. A fájl ne módosuljon, a megnyitott erőforrás záródjon le. Hiányzó fájl és más I/O-hiba továbbterjed; ne adjon csendben üres listát.

Példa fájltartalom: `" api \n\nWorker\napi\n"` → `["api", "Worker", "api"]`.

- [ ] Üres fájl, ékezetes név, záró sortörés nélküli utolsó sor és hiányzó fájl tesztelve.
- [ ] Külön tesztelhető normalizálási logika marad.
- [ ] Ideiglenes fájlokkal tesztelsz; valódi konfigurációhoz nem nyúlsz.

**Explain:** Why is a missing file different from an empty file?

## L07

**Cél:** JSON-parszolás és alkalmazási validáció külön kezelése.

Írj `load_limits(path)` függvényt UTF-8 JSON-fájlhoz. A JSON gyökere object legyen, pontosan `timeout_seconds` és `max_jobs` mezőkkel. Mindkét érték int, bool nélkül. Timeout: 1–30; max_jobs: 1–1000. Eredmény az ellenőrzött két mezős dict. Hiányzó/extra mező, rossz gyökértípus, hibás érték vagy tartomány `ValueError`; a JSON formátumhibája `json.JSONDecodeError` maradhat. Az I/O-hibák továbbterjednek.

Példa: `{"timeout_seconds": 5, "max_jobs": 20}` → azonos tartalmú dict.

- [ ] Normál, mindkét alsó/felső határ, bool, hiányzó és extra mező tesztelve.
- [ ] Lista gyökér, hibás JSON és hiányzó fájl külön ellenőrzött.
- [ ] Hibás konfigurációra nem keletkezik hallgatólagos default.
- [ ] Nem használod az `assert`-et alkalmazási bemenetvalidációként.

**Explain:** Does valid JSON guarantee valid application configuration?

## L08

**Cél:** CSV-adatfeldolgozás, aggregáció és hibás rekordok számlálása.

Írj `summarize_orders(path)` függvényt UTF-8 CSV-fájlhoz. A CSV-szintaxis a feladatban garantáltan helyes; használj `csv` modult, ne kézi vessződarabolást. Az első logikai rekord pontosan `service,count` fejléc, ebben a sorrendben; üres fájl vagy más fejléc `ValueError`.

Minden további rekordnak pontosan két mezője kell legyen. A service trim után nem üres, kis-/nagybetűt megőrző string. A count mező strip után `int()`-tel alakítandó, és nem negatív. Hibás oszlopszám, üres service, hibás számszöveg vagy negatív count esetén növeld az invalid számlálót és folytasd. A nulla érvényes. I/O-hibát ne minősíts rossz rekordnak.

Eredmény: `{"totals": {service: összeg, ...}, "invalid": hibás_rekordok_száma}`. Csak fejlécet tartalmazó fájl → `{"totals": {}, "invalid": 0}`. A kimeneti dict sorrendjére nincs követelmény. A fájl ne módosuljon, és záródjon le.

Példa:

```csv
service,count
api,2
worker,1
api,3
,4
api,bad
```

Elvárt eredmény: `{"totals": {"api": 5, "worker": 1}, "invalid": 2}`.

- [ ] Ismétlődés, nulla, negatív szám, hibás mezőszám és csak fejléc külön tesztelve.
- [ ] Idézőjelezett, vesszőt tartalmazó service működik, például `"api,west",2`.
- [ ] Hiányzó fájl és rossz fejléc nem válik üres sikeres eredménnyé.
- [ ] Megindoklod, mi függ a sorok számától és mi az egyedi service-nevek számától.

**Explain:** What state still grows when the input is read one row at a time?

## Kész állapot

Az elfogadási feltételek teljesítése a konkrét feladat kész állapota. A kapcsolódó készség stabilitását egy későbbi, új feladatváltozaton ellenőrizzük. Az idő és a kapott segítség megfigyelés, nem automatikus minősítés.
