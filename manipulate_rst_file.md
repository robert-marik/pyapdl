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
