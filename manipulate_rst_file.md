V **PyMAPDL** se `.rst` soubor (result file z ANSYSu) ukládá automaticky při řešení a potom ho můžeš znovu načíst buď přes MAPDL postprocessing (`POST1`), nebo přes PyMAPDL reader (`ansys.mapdl.reader` / `pymapdl-reader`). Níže jsou oba běžné postupy.

---

# 1️⃣ Uložení `.rst` souboru v PyMAPDL

`.rst` soubor se vytvoří automaticky po `solve()`. Stačí nastavit jméno jobu a adresář.

```python
from ansys.mapdl.core import launch_mapdl

mapdl = launch_mapdl()

# Nastavení jména úlohy
mapdl.jobname = "moje_ulohа"

# Řešení
mapdl.run("/SOLU")
mapdl.solve()
mapdl.finish()
```

Po výpočtu vznikne soubor:

```
moje_uloha.rst
```

Umístění zjistíš takto:

```python
print(mapdl.directory)
```

Nebo můžeš soubor zkopírovat:

```python
mapdl.download("moje_uloha.rst")
```

---

# 2️⃣ Načtení `.rst` v PyMAPDL (POST1)

Pokud chceš `.rst` načíst zpět do MAPDL:

```python
mapdl.clear()
mapdl.resume("moje_uloha", "db")  # pokud máš i db soubor

mapdl.post1()
mapdl.file("moje_uloha", "rst")
mapdl.set(1)

# např. vypsání posunů
disp = mapdl.post_processing.nodal_displacement("X")
print(disp)
```

Bez `.db` to jde taky, ale omezeně.

---

# 3️⃣ Načtení `.rst` přímo v Pythonu (doporučeno)

Nejčastěji se používá reader:

```python
from ansys.mapdl import reader as pymapdl_reader

rst = pymapdl_reader.read_binary("moje_uloha.rst")

print(rst.nsets)      # počet kroků
print(rst.nnodes)     # počet uzlů
```

Například posuny:

```python
nnum, disp = rst.nodal_displacement(0)
print(disp)
```

Napětí:

```python
nnum, stress = rst.nodal_stress(0)
```

---

# 4️⃣ Uložení výsledků jinam

Například do numpy:

```python
import numpy as np

nnum, disp = rst.nodal_displacement(0)
np.save("posuny.npy", disp)
```

---

# 5️⃣ Kontrola že `.rst` existuje

```python
import os
print(os.listdir(mapdl.directory))
```

---

# Shrnutí

* `solve()` → vytvoří `.rst`
* `mapdl.file("jobname","rst")` → načte do POST1
* `reader.read_binary("file.rst")` → načte do Pythonu

---

Jestli chceš, můžu ukázat:

* načtení více loadstepů
* animaci z `.rst`
* export do CSV nebo VTK.

---

Načtení více **loadstepů / časových kroků** z `.rst` v PyMAPDL jde nejčastěji přes reader (`ansys.mapdl.reader`), protože je to přehlednější než `POST1`.

Níže je kompletní příklad.

---

# 1️⃣ Načtení `.rst` souboru

```python
from ansys.mapdl import reader as pymapdl_reader

rst = pymapdl_reader.read_binary("moje_uloha.rst")

print("Počet kroků:", rst.nsets)
```

Každý `set` odpovídá jednomu loadstepu nebo substepu.

---

# 2️⃣ Vypsání všech loadstepů

```python
for i in range(rst.nsets):
    info = rst.set_info(i)
    print(f"Set {i}: čas = {info['time']}")
```

Například dostaneš:

```
Set 0: čas = 0.0
Set 1: čas = 0.1
Set 2: čas = 0.2
```

---

# 3️⃣ Načtení posunů ze všech loadstepů

```python
all_disp = []

for i in range(rst.nsets):
    nnum, disp = rst.nodal_displacement(i)
    all_disp.append(disp)

print("Počet kroků:", len(all_disp))
print("Tvar:", all_disp[0].shape)  # (počet_uzlů, 3)
```

---

# 4️⃣ Načtení napětí ze všech loadstepů

```python
all_stress = []

for i in range(rst.nsets):
    nnum, stress = rst.nodal_stress(i)
    all_stress.append(stress)
```

---

# 5️⃣ Načtení konkrétního loadstepu

Například třetí krok:

```python
nnum, disp = rst.nodal_displacement(2)
```

---

# 6️⃣ Uložení všech kroků do jednoho pole

```python
import numpy as np

disp_array = np.array(all_disp)
print(disp_array.shape)
```

Výsledek:

```
(počet_kroků, počet_uzlů, 3)
```

---

# 7️⃣ Alternativa přes MAPDL POST1

Pokud chceš načítat loadstepy přímo v MAPDL:

```python
mapdl.post1()
mapdl.file("moje_uloha", "rst")

nsets = mapdl.post_processing.nsets

for i in range(1, nsets+1):
    mapdl.set(i)
    disp = mapdl.post_processing.nodal_displacement("X")
    print(f"Krok {i} načten")
```

Pozor:

* POST1 indexuje od **1**
* reader indexuje od **0**

---

# Shrnutí

* `rst.nsets` → počet loadstepů
* `rst.nodal_displacement(i)` → i-tý krok
* `rst.set_info(i)` → čas/loadstep info

---

Jestli chceš, můžu ukázat:

* jak zjistit loadstep vs substep
* jak načíst jen poslední krok
* jak udělat animaci z loadstepů.
