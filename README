
Spuštěni
```
from ansys.mapdl.core import launch_mapdl
mapdl = launch_mapdl(transport_mode="insecure")
```

Načtení a spuštění souboru
```
# read file
with open("geometry.mac") as f:
  data = f.read()
mapdl.input_strings(data);
```
Zobrazit mesh jako v ANSYS
```
mapdl.view(1,1,1,1)
mapdl.run("eplot")
```
Zobrazit mesh v pyVista
```
mapdl.eplot()
```
