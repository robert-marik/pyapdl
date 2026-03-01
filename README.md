# Zaklady

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

Ulozeni
```
mapdl.save("/home/marik/work/ansys/beam_with_holes/data.db")
```

Obnoveni
```
mapdl.restore("/home/marik/work/ansys/beam_with_holes/data.db")
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

```
mapdl.allsel()
mapdl.plnsol("EPEL","X")  # jedna komponenta elastickeho stresu
```

```
result = mapdl.result
result.plot_nodal_solution(
    0,
    "UY",
    show_edges=True,
    lighting=False,
    )
```

```
result.plot_principal_nodal_stress(
  0, 'SEQV', displacement_factor = 1,
  show_displacement=True,
  lighting=False,
  background='w',
  show_edges=True,
  text_color='k',
  add_text=True)
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

# Reseni

```
mapdl.solution()
mapdl.antype("STATIC")
output = mapdl.solve()
mapdl.finish()
```

# Mesh

```
mapdl.esize(0.001)

# tetrahedral elements for meshing the block
# mapdl.mshkey(0)
# mapdl.mshape(1, "3D")
# mapdl.et(0, "SOLID185")

# # cube meshing with SOLID185 elements
# # tetrahedral elements for meshing the block
mapdl.mshkey(1)
mapdl.mshape(0, "3D")
mapdl.et(1, "SOLID185")

mapdl.vmesh("ALL")
mapdl.eplot(savefig="mesh.png")

# # Select tetrahedral elements for meshing the block
# mapdl.mshkey(1)  # Tetrahedral elements
# mapdl.et(1, "SOLID185")

# mapdl.vmesh("ALL")

# mapdl.mshape(0,"3D")
# mapdl.mshkey(1)
# mapdl.vsweep("ALL")
# mapdl.allsel('all')
```

# Okrajove podminky

```
#Define loading. Fix x=0 face and apply a force in the x-direction in the
#opposite direction to the positive x-axis. The force is distributed uniformly on the face at x=a.
mapdl.nsel("S", "LOC", "X", a)
# Zatizeni silou
num_nodes_face = mapdl.get("NUM_NODES", "NODE", 0, "COUNT")
mapdl.f("ALL", "FX", force/num_nodes_face)
# Zatizeni posunem
# mapdl.d("ALL", "UX", -0.01)   # posun 1 cm

mapdl.nsel("S", "LOC", "X", 0)
mapdl.d("ALL", "ALL")
mapdl.allsel(mute=True)

# plot the geometry
mapdl.eplot(savefig="geom.png")
```

Okrajove podminky v APDL
```
! ==============================
! 1) Levá dolni hrana fixně (osa z dolu)
! ==============================

lsel,s,loc,x,0
lsel,r,loc,z,vyska
dl,all,,uz,0   ! žádný posun svisle
dl,all,,ux,0   ! žádný posun podel
dl,all,,uy,0   ! žádný posun do šířky
allsel

! ==============================
! 2) Pravá stěna – zákaz UX
! ==============================
asel,s,loc,x,delka
da,all,ux,0
allsel

! ==============================
! 3) Horní hrana pravé stěny – posun dolů
! ==============================
lsel,s,loc,x,delka
lsel,r,loc,z,0
dl,all,,uz,0.02
allsel
```

# Material

```
mapdl.units("SI")  # SI - International system (m, kg, s, K).
# define material properties
mapdl.mp("EX", 1, e_x)
mapdl.mp("EY", 1, e_y)
mapdl.mp("EZ", 1, e_z)
mapdl.mp("GXY", 1, g_xy)
mapdl.mp("GYZ", 1, g_yz)
mapdl.mp("GXZ", 1, g_xz)
mapdl.mp("PRXY", 1, pr_xy)
mapdl.mp("PRYZ", 1, pr_yz)
mapdl.mp("PRXZ", 1, pr_xz)
mapdl.mp("DENS", 1, density)
```

# Postprocessing

```

# select all nodes on x=a and get the x-displacement
result = mapdl.result
mapdl.nsel("S", "LOC", "X", a)
nsel = mapdl.mesh.nnum.tolist()
mapdl.allsel(mute=True)
num, ux = result.nodal_solution(0, "UX", nsel)
np.nanmin(ux[:,0]), np.nanmax(ux[:,0])
```

# Dalsi

Uložení ID posledního přidaného objemu
```
block, 0,length,0,width,0,height
*get,block_id,volu,0,num,max
```
