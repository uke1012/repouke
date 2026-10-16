# repouke · assets de Legacy RP

Repositorio de assets para el servidor FiveM de Legacy RP: recursos listos para `resources/` del servidor y los `.blend` de origen.

Las herramientas viven en sus propios repos: [Sollumz](https://github.com/uke1012/Sollumz) y [VichoTools](https://github.com/uke1012/VichoTools).

## Estructura

```
resources/
  [legacy]/            # copia esta carpeta a resources/ del servidor
    legacy_props/      # props: .ydr, .ytd, .ytyp
    legacy_mlo/        # MLOs: .ymap, .ytyp, .ydr, .ytd, .ybn
    legacy_peds/       # peds: .ydd, .yft, .ytd, .ymt
      fxmanifest.lua
      stream/
source/
  blend/{props,mlo,peds}/   # archivos .blend de trabajo (Sollumz)
  textures/                 # texturas fuente .dds
```

Cada recurso es una carpeta con su `fxmanifest.lua` y una carpeta `stream/`. Para un recurso nuevo, copia uno existente, renómbralo y añade `ensure nombre_recurso` (o `ensure [legacy]`) en `server.cfg`.

## Git LFS

`.blend`, `.dds`, `.ytd` y `.ydr` se guardan con Git LFS (ver `.gitattributes`). Antes de clonar o subir assets:

```
git lfs install
git clone https://github.com/uke1012/repouke
```

Para añadir otra extensión pesada: `git lfs track "*.yft"` y haz commit de `.gitattributes`.
