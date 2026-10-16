# Flujo Blender → FiveM (Legacy RP)

Guía de trabajo para llevar peds, props y MLOs desde Blender hasta un servidor FiveM usando las herramientas incluidas en este repositorio.

| Carpeta | Qué es | Para qué la usamos |
|---|---|---|
| `Sollumz-main/` | Add-on de Blender (requiere **Blender 4.2+**) | Importar/exportar YDR, YDD, YFT, YBN, YTYP, YMAP |
| `VichoTools-main/` | Add-on complementario de Sollumz (Sollumz 2.5+, .NET 8 x64) | Convertir texturas a DDS y empaquetarlas en YTD, exportar instancias de MLO a `.ymap` |
| `blender_io_GTA5Ped-master/` | Importador/exportador antiguo de mallas de ped | Solo referencia; para trabajo nuevo usar Sollumz |
| `fivem-master/` | Código fuente de Cfx (FiveM/FXServer) | Referencia de cómo funciona el streaming |

> Los números de esta guía son **recomendaciones de rendimiento** para un servidor RP con muchos jugadores, no límites duros del motor. Si un asset los supera, que sea por una razón concreta.

---

## 1. Preparar Blender

1. Instalar **Blender 4.2 LTS** o superior (Sollumz exige `blender_version_min = 4.2.0`).
2. Instalar Sollumz: *Edit → Preferences → Get Extensions → Install from Disk* y elegir el `.zip` de `Sollumz-main`.
3. Instalar VichoTools igual que Sollumz. Necesita el **runtime .NET 8 x64** en Windows para la conversión DDS/YTD.
4. Escena en **metros**, escala de unidad `1.0`. GTA V usa Z hacia arriba, igual que Blender, así que no hay que rotar nada.
5. (Recomendado) Tener **CodeWalker** a mano para extraer assets originales de referencia y revisar el resultado.

## 2. Modelado

Reglas comunes a todo tipo de asset:

- **Escala real**: una puerta mide unos 2,1 m, un ped unos 1,8 m. Compara con un asset original importado.
- **Pivote/origen**: en props, en la base del objeto, centrado (así se apoya en el suelo al colocarlo). En MLOs, el origen del interior es el punto de anclaje en el mapa.
- **Transformaciones aplicadas** (*Ctrl+A → All Transforms*) antes de convertir a objeto Sollumz.
- **Normales limpias**: sin caras duplicadas ni n-gons raros; triangula o deja quads, Sollumz triangula al exportar.
- **UVs**: sin solapamientos en texturas únicas; densidad de texel parecida entre piezas del mismo asset.
- **Materiales**: crea los materiales con el **Shader de Sollumz** (*Sollumz → Material → Create Shader Material*). Los materiales Principled BSDF normales se pueden convertir, pero revisa el resultado.
- **Vertex colors**: muchos shaders de GTA los usan (ambient occlusion, tinte, viento). Si no los necesitas, deja la capa en blanco en vez de borrarla.

### 2.1 Peds y ropa

- Parte siempre del **esqueleto original** (`mp_m_freemode_01` / `mp_f_freemode_01` o el ped base) importado con Sollumz. No crees huesos nuevos.
- Máximo **4 influencias de hueso por vértice**, pesos normalizados.
- Cada componente (torso, piernas, zapatos…) se exporta como **YDD** separado, con su textura en un YTD con el mismo nombre base.
- Nombres de componentes al estilo GTA: `jbib_000_u.ydd`, `jbib_diff_000_a_uni.ytd`, etc.

### 2.2 Props

- Un objeto Sollumz **Drawable** por prop → exporta **YDR**. Si tiene partes que se rompen o se mueven, **Fragment** → **YFT**.
- Añade **colisión** (Bound Composite → YBN o embebida en el YDR). Usa primitivas (caja, cilindro, esfera) o una malla de colisión muy simple; nunca la malla visual.
- Registra el prop en un **YTYP** (archetype) con su `lodDist` y la textura/YTD asociada.

### 2.3 MLOs (interiores)

- Estructura en Sollumz: **YTYP → Archetype tipo MLO → Rooms, Portals y Entity Sets**.
- Cada habitación con su **bounding box** ajustado; los **portales** deben coincidir exactamente con los huecos de puertas y ventanas, o habrá objetos que desaparecen.
- Separa el casco (paredes, suelo, techo) de los muebles: el casco como un YDR, los muebles como entidades del MLO reutilizando props.
- Colisión del casco en un YBN propio.
- Con VichoTools, exporta la **instancia del MLO como `.ymap`** para colocarla en el mundo.

## 3. Límites recomendados

### 3.1 Polígonos (triángulos) por LOD

| Tipo de asset | High (LOD0) | Med (LOD1) | Low (LOD2) | Very Low (LOD3) |
|---|---|---|---|---|
| Ped completo (todas las piezas visibles) | 30k – 60k | ~50 % | ~25 % | — |
| Pieza de ropa (YDD) | 2k – 10k | ~50 % | ~25 % | — |
| Prop pequeño (botella, silla) | 300 – 3k | ~50 % | ~20 % | opcional |
| Prop mediano (mueble grande, máquina) | 3k – 15k | ~50 % | ~25 % | ~10 % |
| Prop grande (vehículo estático, estructura) | 15k – 40k | ~50 % | ~25 % | ~10 % |
| Casco de MLO (paredes/suelo/techo) | 20k – 80k | — | — | — |
| MLO completo visible a la vez (casco + muebles) | < 300k | — | — | — |

### 3.2 LODs y distancias

- **Props**: mínimo High + Low. Distancias orientativas: High 0–30 m, Med 30–60 m, Low 60–120 m, `lodDist` del archetype 100–200 m según tamaño. Un prop pequeño de interior puede tener `lodDist` de 50 m.
- **Peds**: High + Med como mínimo para la ropa; GTA cambia de LOD muy cerca, así que el Med debe verse bien a 10–15 m.
- **MLOs**: los interiores no usan LODs de distancia igual que el exterior; se apoyan en rooms y portales para no dibujar lo que no se ve. El exterior del edificio (si lo haces) sí necesita LOD/SLOD en el YMAP.
- En Sollumz, los LODs se gestionan desde el panel **LOD** del objeto: cada nivel guarda su propia malla.

### 3.3 Texturas

| Uso | Tamaño recomendado | Máximo razonable |
|---|---|---|
| Ropa de ped (difusa) | 1024×1024 | 2048×2048 |
| Normal / specular de ped | 512 – 1024 | 2048 |
| Prop pequeño | 256 – 512 | 1024 |
| Prop mediano/grande | 1024 | 2048 |
| MLO (texturas tileables de pared/suelo) | 512 – 1024 | 2048 |

- Siempre **potencia de 2** (256, 512, 1024, 2048). VichoTools redondea automáticamente a la potencia de 2 más cercana.
- Formatos DDS: **DXT1** para texturas opacas, **DXT5** si tienen alfa, **BGRA sin compresión y sin mipmaps** solo para paletas de tinte. VichoTools elige DXT1/DXT5 detectando la transparencia.
- Mipmaps **siempre activados** (menos en paletas de tinte).
- Mantén cada **YTD por debajo de 16 MB**; FiveM avisa de *oversized assets* por encima de ese tamaño y causan tirones al cargar. Lo ideal para RP es < 8 MB por YTD.
- En MLOs, **comparte texturas** entre muebles en un mismo YTD en lugar de un YTD por objeto.

## 4. Texturas DDS/YTD con VichoTools

1. Selecciona los objetos (o la colección) con sus materiales Sollumz ya asignados.
2. En el panel de **VichoTools → Textures**, crea un YTD y pulsa **auto-fill / añadir mallas**: detecta las imágenes de los materiales.
3. Activa **Advanced Mode** si quieres limitar el tamaño:
   - *Max Pixel*: tamaño máximo (p. ej. 1024).
   - *Half Texture Size*: reduce a la mitad para versiones ligeras.
4. Elige la calidad (`Normal` para pruebas, `Production` para la versión final).
5. Exporta: VichoTools convierte a DDS y genera el `.ytd` en la ruta indicada.
6. Marca como **Is Tint?** solo las paletas de tinte.

El nombre del YTD debe coincidir con el que referencia el asset (en props, el *Texture Dictionary* del archetype en el YTYP; en ropa, el nombre del componente).

## 5. Exportar con Sollumz

1. Selecciona los objetos Sollumz raíz (Drawable, Drawable Dictionary, Fragment, Bound Composite, YTYP).
2. *File → Export → Sollumz* (o el botón de exportar del panel Sollumz).
3. Comprueba en las opciones que se exporten los LODs que has creado.
4. Resultado típico:
   - Prop: `mi_prop.ydr`, `mi_prop.ytd`, `mi_prop.ybn` (si va aparte), `mi_props.ytyp`.
   - Ropa: `jbib_000_u.ydd`, `jbib_diff_000_a_uni.ytd`.
   - MLO: `mi_mlo.ytyp`, `mi_mlo_shell.ydr`, `mi_mlo_shell.ybn`, YTDs, y el `.ymap` de la instancia (VichoTools).
5. Abre el resultado en **CodeWalker** antes de subirlo al servidor: es mucho más rápido detectar errores ahí que reiniciando FiveM.

## 6. Stream en FiveM

Estructura de un recurso:

```
resources/[assets]/legacy_mi_asset/
├── fxmanifest.lua
└── stream/
    ├── mi_prop.ydr
    ├── mi_prop.ytd
    ├── mi_prop.ybn
    ├── mi_props.ytyp
    └── mi_mlo.ymap
```

Todo lo que esté en `stream/` se transmite a los clientes automáticamente.

### 6.1 `fxmanifest.lua` para props y MLOs

```lua
fx_version 'cerulean'
game 'gta5'

this_is_a_map 'yes'

data_file 'DLC_ITYP_REQUEST' 'stream/*.ytyp'
```

- `this_is_a_map 'yes'` es necesario para recursos con YMAPs/MLOs.
- `DLC_ITYP_REQUEST` registra los YTYP para que el juego conozca los archetypes (props nuevos e interiores).

### 6.2 `fxmanifest.lua` para ropa de ped

```lua
fx_version 'cerulean'
game 'gta5'
```

- Para **reemplazar** una prenda original basta con este manifest y los archivos en `stream/` con el prefijo del modelo, p. ej. `mp_m_freemode_01^jbib_000_u.ydd` y `mp_m_freemode_01^jbib_diff_000_a_uni.ytd`.
- Para ropa **addon** (sin reemplazar nada) genera el pack con una herramienta de packs de ropa (p. ej. Durty Cloth Tool). Esa herramienta crea los `.ymt`/`.meta`, los nombres correctos y su propio `fxmanifest.lua`; úsalo tal cual.

### 6.3 Activar y probar

1. Añade `ensure legacy_mi_asset` en `server.cfg` (después de los recursos de mapa base).
2. Reinicia el recurso con `restart legacy_mi_asset` o el servidor completo si es un MLO nuevo.
3. Revisa la consola del cliente (F8) por avisos de *oversized asset* o archetypes que no cargan.
4. Usa `resmon` en el cliente para vigilar el impacto en memoria.

## 7. Checklist antes de subir

- [ ] Escala, pivote y transformaciones aplicadas.
- [ ] Triángulos dentro de los rangos de la tabla 3.1.
- [ ] LODs creados y probados a distancia.
- [ ] Colisión simple (no la malla visual).
- [ ] Texturas en potencia de 2, DXT1/DXT5 con mipmaps, YTD < 16 MB.
- [ ] Nombres de YTD coinciden con los referenciados.
- [ ] Revisado en CodeWalker.
- [ ] `fxmanifest.lua` correcto y recurso añadido al `server.cfg`.
- [ ] Probado en el servidor sin avisos en F8.
