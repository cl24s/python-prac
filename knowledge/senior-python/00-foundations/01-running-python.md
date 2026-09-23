# Környezet és futtatás

## Mit kell tudnod?

Tudd megmondani, melyik Python fut, honnan indul a program és hová telepítesz csomagot. Az első feladatokhoz nem kell Docker, webframework vagy cloud.

## Minimális indulás

A gyakorlati cél CPython 3.12 vagy újabb, hagyományos build. A korábbi fejezetek ellenőrzése 3.12-n történt; a tényleges saját verziódat rögzítsd. A példákhoz nem kell minden új nyelvi funkciót bekapcsolni vagy a legfrissebb prerelease-re váltani.

Windows PowerShellben, a repó gyökerében:

```powershell
python --version
python -m venv .venv
.\.venv\Scripts\python.exe -c "import sys; print(sys.version); print(sys.executable)"
```

Ha a `python` parancs nem található, de a telepített launcher `py` parancsa működik, a környezet létrehozásához használható `py -m venv .venv`. Ha egyik sem működik, előbb működő Python-telepítés kell. Ne változtass találomra globális PATH- vagy biztonsági beállításokat.

Linuxon vagy WSL-ben:

```bash
python3 --version
python3 -m venv .venv
.venv/bin/python -c "import sys; print(sys.version); print(sys.executable)"
```

Windows és WSL között ne másold át a `.venv` mappát: mindkét környezetben külön hozd létre. Az aktiválás opcionális; itt közvetlenül a környezet interpreterét hívjuk, így nem kell PowerShell execution policyt módosítani.

## Az első script

Ezt a példát ideiglenes `scratch.py` fájlban futtasd, ne feladatmegoldásként:

```python
from pathlib import Path
import sys

print(f"Python: {sys.version.split()[0]}")
print(f"Interpreter: {sys.executable}")
print(f"Working directory: {Path.cwd()}")
```

Windows: `.\.venv\Scripts\python.exe scratch.py`; Linux/WSL: `.venv/bin/python scratch.py`. A relatív fájlút alapja jellemzően az aktuális munkakönyvtár, nem automatikusan a script helye. Az editorban is ugyanazt az interpretert válaszd.

## Tesztcsomag, amikor már szükséges

Az első tiszta Python-feladathoz nincs külső függőség. Pytest előtt Windows alatt `.\.venv\Scripts\python.exe -m pip install pytest`, Linux/WSL alatt `.venv/bin/python -m pip install pytest`. A verziót a megfelelő interpreterrel kiadott `-m pytest --version` mutatja.

Ez induló, nem lockolt környezet. Az első projektben rögzítjük a szükséges függőségeket és az újratelepítés módját; most ne töltsük fel előre az összes frameworköt. A `.venv`, cache-ek és valódi `.env` fájlok nem kerülhetnek Gitbe.

## Tipikus hibák és önellenőrzés

- A `pip` másik Pythonhoz tartozik: használd ugyanannak az interpreternek a `-m pip` parancsát.
- A terminál és az editor más környezetet használ: nézd meg a `sys.executable` értéket.
- A program máshol keresi a fájlt: ellenőrizd a munkakönyvtárat.
- Önellenőrzés: üres fájlból elindítok egy scriptet, megmutatom az interpretert és értem a hibaüzenetet, ha rossz fájlnevet adok.

**Explain:** How do you check which Python interpreter runs your program?

[venv dokumentáció](https://docs.python.org/3/library/venv.html) · [Vissza](README.md)
