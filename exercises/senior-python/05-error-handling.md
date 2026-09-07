# Hibakezelés és erőforrás-kezelés – önálló feladatok

Állapot: feladatkiírás kész; megoldás és review még nincs. Kész megoldás nincs mellékelve. Standard library elegendő. A megoldások és tesztek később az `05-error-handling/` almappába kerülhetnek.

[Tananyag](../../knowledge/senior-python/05-error-handling/README.md)

## E01

**Cél:** keskeny hibahatár és részleges feldolgozás.

Írj `process_lines(lines, save)` függvényt. Az input stringek egyszer bejárható iterable-je. Minden sorból `int(line.strip())` készül; parse ValueError esetén a sor hibás, feldolgozás folytatódik. Minden sikeres értékre egyszer `save(value)` hívás történik. Az eredmény `(saved_count, invalid_count)`. A save minden hibája változatlanul továbbterjed, és az input további elemeit nem szabad fogyasztani.

Példa: `['1', 'bad', ' 2 ']` → mentett értékek 1, 2; eredmény `(2, 1)`.

- [ ] Üres input `(0, 0)`, minden hibás sor külön számít.
- [ ] Save által dobott ValueError sem minősül parse-hibának.
- [ ] Save-hiba után nincs újabb mentés és nincs további inputfogyasztás.
- [ ] Nincs `except BaseException` és nincs néma általános elnyelés.

**Interview:** Why is the save call outside the parser's try block?

## E02

**Cél:** hibafordítás és chaining.

Írj `load_worker_count(read_text)` függvényt. A read_text argumentum nélküli callable stringet ad. OSError esetén saját `ConfigUnavailable` hibát dobj eredeti okkal. A string intté alakításának hibája vagy 1-nél kisebb érték saját `InvalidConfig` legyen. Mindkettő `ConfigError`-ból származzon. Hibás tartalom esetén `field='workers'` adatot tárolj; ne tartalmazza a teljes nyers inputot az üzenet.

- [ ] Mindkét technikai hiba `__cause__` mezője az eredeti exception objektum.
- [ ] A tartományhibának nem kell mesterséges technikai okot gyártani.
- [ ] Más read_text-hiba (például TypeError) változatlanul továbbterjed.
- [ ] Érvényes szövegre int eredmény, pontosan egy olvasás.

**Interview:** Which parts of your error are stable machine-readable data?

## E03

**Cél:** részleges erőforrás-megszerzés takarítása.

Írj `copy_text(open_source, open_target)` függvényt. A factory-k context managert adnak; a source iterálható string sorokat, a target `write(str)` műveletet biztosít. Először source, utána target nyíljon; a sorok változatlanul másolódjanak. Mindkét erőforrás a függvény tulajdona. A cleanupok a feladatban nem dobnak hibát.

- [ ] Normál esetben target záródik előbb, majd source.
- [ ] Target-megnyitás hibája után source lezárul.
- [ ] Írási hiba után mindkettő lezárul, az eredeti exception jut a hívóhoz.
- [ ] Fake erőforrásokkal bizonyítod a sorrendet; nem kell valódi fájlokat módosítani.

**Interview:** What additional policy is needed if cleanup itself can fail?

## E04

**Cél:** retry-szerződés és tesztelhetőség.

Írj `read_with_retry(operation, pause, attempts)` függvényt. Csak saját `TransientReadError` esetén próbálkozzon újra. Az attempts pozitív int, bool nélkül. Várakozások: 0.1, 0.2, 0.4, majd legfeljebb 1.0 másodperc; ezt az injektált pause kapja, tesztben valódi alvás nélkül. Az utolsó sikertelen próbálkozás után nincs pause; az utolsó exception objektum változatlanul továbbterjed.

- [ ] Elsőre siker esetén egy hívás, nulla pause.
- [ ] Két átmeneti hiba, majd siker esetén három hívás és két pause.
- [ ] Kimerülés, nem retryzható hiba és hibás attempts is tesztelt.
- [ ] Nem int attempts TypeError, nem pozitív érték ValueError.
- [ ] Írásos válaszban indoklod, miért nem használható vakon fizetés vagy üzenetküldés megismétlésére.

**Interview:** What does a timeout tell you about a remote write's outcome?

## E05

**Cél:** diagnosztika és elmaradó mellékhatás.

Készíts `run_job(job_id, operation, logger)` határfüggvényt. Az operation argumentum nélkül fut. Siker esetén eredménye változatlanul visszatér. Exception esetén pontosan egy error-log keletkezik job_id-val és exceptioninformációval, majd ugyanaz az exception objektum továbbterjed. A job_id a feladatban biztonságos, nem üres string; a naplózó nem hibázik.

- [ ] Sikerre nincs error-log, és nincs duplikált operation-hívás.
- [ ] Hibára ugyanaz az exception és pontosan egy logrekord.
- [ ] KeyboardInterrupt nem minősül normál jobhibának és nem nyelődik el.
- [ ] Recording handlerrel vagy fake loggerrel tesztelsz; nem a teljes traceback szövegét hasonlítod.

**Interview:** Where would you put correlation IDs and where would you avoid them?
