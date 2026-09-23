# 00. Gyakorlati alapozás

Cél: áthidalni a rövid programok önálló megírása és a mélyebb Python-fejezetek közötti rést. Nem kell előre végigolvasni mindent. Az éppen gyakorolt készséghez tartozó fejezetet vedd elő.

| Fejezet | Mit gyakorolsz? | Kapcsolódás |
| --- | --- | --- |
| [01. Környezet és futtatás](01-running-python.md) | Interpreter, venv, script, futtatási könyvtár, csomagtelepítés. | A saját kód futtatása. |
| [02. Rövid programok írása](02-writing-small-programs.md) | Függvény, return, if, for, string, list, dict, set, alap type hintek. | L01–L04, D01, B01, D02. |
| [03. Tesztelés és hibakeresés](03-testing-and-debugging.md) | Elvárt eredmény, assert, pytest, traceback, debugger, regresszió. | L05, Q02; tesztek minden feladathoz. |
| [04. Fájlok és kis programok](04-files-and-programs.md) | Pathlib, UTF-8, with, JSON, CSV, modul és CLI határa. | L06–L08, majd logelemző. |

[Új feladatok — L01–L08](../../../exercises/senior-python/00-foundations.md) · [Adatszerkezet-feladatok](../../../exercises/senior-python/02-data-structures.md)

## Ellenőrzés és továbbhaladás

Egy helyes példaprogram megértése után írj saját megoldást és teszteket. Egy másik alkalommal ugyanazt a készséget új változaton ellenőrizzük. A továbblépést a [ROADMAP](../../../ROADMAP.md) kapui írják le, nem egy kötelező naptár.

A fájl-, hálózat- és adatbázis-műveleteket később adapterként választjuk el a tiszta feldolgozó függvényektől. Most ehhez nem kell hexagonális framework vagy nagy osztályhierarchia.

## Ellenőrzési hatókör

2026-09-23: helyi ellenőrzés Linuxon, CPython 3.13.5 és pytest 9.0.2 környezetben. A 4 fejezet mind a 10 Python-blokkja külön ideiglenes fájlból sikeresen futott: 9 script és 1 pytest-modul, utóbbiban 6 sikeres tesztesettel. Az L05 szándékos hibáit külön ellenőriztük, a javítást nem mellékeltük.

Az új/módosított dokumentumok relatív hivatkozásai útvonal- és használt feladatankor-ellenőrzést kaptak az új fájlok és a connectorban látott állományjegyzék alapján. Windows/PowerShell-futtatás, csomagtelepítés és interaktív debugger-menet nem történt. A korábbi 97 fejezetet nem futtattuk újra. A felhasználói feladatok nincsenek megoldva vagy teljesítettnek jelölve; ez nem teljes repository-audit.

## Hivatalos referencia

[Python tutorial](https://docs.python.org/3/tutorial/) — programozási tapasztalattal új Python-használóknak. Az alfejezeteket célzott referenciaként használd, ne újabb kötelező olvasási listaként.
