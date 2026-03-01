
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

# Kresleni

Zobrazit mesh jako v ANSYS
```
mapdl.view(1,1,1,1)
mapdl.run("eplot")
```
Zobrazit mesh v pyVista
```
mapdl.eplot()
```

Vykresleni posunuti
```
mapdl.post1()  
mapdl.view(1,"",1)
mapdl.set("last")
mapdl.plnsol("uz") ! posunuti v jedne ose
mapdl.show("uz.png") ! ulozeni
mapdl.plnsol("epto","int") ! celkovy ekvivalentni strain
mapdl.plnsol("epto","x") ! jedna komponenta strain tenzoru
```

Vykrelseni v pyMAPDL
```
mapdl.post1()
mapdl.post_processing.plot_nodal_elastic_eqv_strain(savefig="strain.png")
mapdl.post_processing.plot_nodal_elastic_component_strain("xz",savefig="strain.png")
```


# Komponenty

Vypis komponent
```
mapdl.cmsel("all")
mapdl.components
```

Vykresleni komponent
```
areas = ["spodni_telisko", "horni_telisko", "horni_podpora", "leva_spodni_podpora", "prava_spodni_podpora", "spodni_podpory"]
for i,a in enumerate(areas[1:]):
    mapdl.cmsel("s",areas[0])
    mapdl.cmsel("a",a)
    mapdl.view(1,1,1,1)
    mapdl.run("aplot")
```

