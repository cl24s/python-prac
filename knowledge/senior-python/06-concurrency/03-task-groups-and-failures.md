# TaskGroup, gather és több feladat hibája

## Mit kell tudnod?

- Az összetartozó taskok élettartamát közös határhoz kötni.
- Megérteni a fail-fast és a részleges eredmények közti döntést.
- Az ExceptionGroup releváns részét kezelni, a többit továbbengedni.

## Magyarázat

Egy lekérés több részfeladatot indíthat. Ha az egyik nélkül a teljes eredmény használhatatlan, indokolt lehet a többi megszakítása. Ha a részleges adatok is értékesek, külön eredmény- és hibajelzési szerződés kell. A választás üzleti döntés, nem csak annak kérdése, hogy melyik asyncio függvény rövidebb.

Python 3.11-től a TaskGroup strukturált határt ad. A csoport kijárata megvárja a saját taskokat; egy normál, nem cancellation jellegű taskhiba a még futó társak megszakítását indítja, majd a hibákat csoportban továbbíthatja. KeyboardInterrupt és SystemExit speciális eset. A cleanup így az összetett művelethez kötődik.

## Kódpélda: egy task hibája lezárja a társát

```python
import asyncio

async def main() -> None:
    started = asyncio.Event()
    cleaned = asyncio.Event()
    observed: list[str] = []

    async def waiting() -> None:
        try:
            started.set()
            await asyncio.Event().wait()
        finally:
            cleaned.set()

    async def failing() -> None:
        await started.wait()
        raise ValueError('invalid shard response')

    try:
        async with asyncio.TaskGroup() as group:
            pending = group.create_task(waiting())
            failed = group.create_task(failing())
    except* ValueError as errors:
        observed.extend(str(error) for error in errors.exceptions)

    assert observed == ['invalid shard response']
    assert cleaned.is_set()
    assert pending.cancelled()
    assert failed.done()
    assert isinstance(failed.exception(), ValueError)

if __name__ == '__main__':
    asyncio.run(main())
```

Az Event biztosítja, hogy a társ már a try/finally blokkjában legyen a hiba előtt. Az except* csak a ValueError részt kezeli. Más, nem kezelt hibatípus továbbterjedne. Nested ExceptionGroup esetén a kapott részfa is lehet csoport, ezért általános riportoláskor ne feltételezz mindig lapos listát.

## TaskGroup és gather összehasonlítása

| Eszköz | Normál használat | Hiba esetén |
| --- | --- | --- |
| TaskGroup | Közös életciklusú feladatok | Normál taskhiba leállítja a társakat, a kijárat megvárja őket |
| gather alapértelmezés | Eredmények input-sorrendben | Első exceptiont továbbítja, társakat ettől nem automatikusan törli |
| gather `return_exceptions=True` | Eredmény/hiba lista | A visszatérő elemeket külön kell értelmezni |

Ha a gather maga törlődik, a még be nem fejezett benne ütemezett awaitable-ek cancellationt kaphatnak. Ez nem ugyanaz, mint az egyik gyerek exceptionje. Már befejezett gather cancel-je sem állítja visszamenőleg le a továbbfutó feladatokat.

## Tipikus hibák és senior szempontok

- **Hiba:** a `return_exceptions=True` eredményeit sikeres adatként használod. **Javítás:** típus és állapot szerinti feldolgozás, explicit részleges eredmény.
- **Hiba:** a handler csak a hibaüzenetet naplózza, a több taskhibát elveszti. **Javítás:** őrizd meg a csoport szerkezetét és a feladatazonosítókat.
- TaskGroup nem adatbázis-tranzakció: egy már végrehajtott mellékhatást a társ cancellationje nem csinál vissza.
- Az except és except* nem keverhető ugyanazon try handlerlistájában; szükség esetén egymásba ágyazott határokat használj.

## Interview questions

**How does TaskGroup differ from gather when a child fails?**

Válaszvázlat: a TaskGroup kezeli a társak leállítását és megvárását; a gather alapértelmezésben az első hibát továbbítja, de nem ugyanezt a felügyeletet adja.

**Does cancelling sibling tasks roll back completed effects?**

Válaszvázlat: nem; a mellékhatás és a task-életciklus külön szerződés.

## Önellenőrzés

- [ ] Megmondom, mi lesz a társ taskokkal hiba után.
- [ ] Részleges eredménynél külön jelzem a hibás elemeket.
- [ ] A nem kezelt exceptioncsoport-rész továbbterjed.


## Kapcsolódó gyakorlat

[C03 – önálló feladat](../../../exercises/senior-python/06-concurrency.md#c03)

## Forrás és továbbolvasás

[Task groups and gather](https://docs.python.org/3.12/library/asyncio-task.html) · [ExceptionGroup](https://docs.python.org/3.12/library/exceptions.html#ExceptionGroup)

[Témakör áttekintése](README.md) · [Teljes tudástérkép](../README.md)
