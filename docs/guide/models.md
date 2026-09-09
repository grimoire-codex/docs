# 3D Models

Grimoire browses STL files and other printable models alongside your books, maps, tokens, and audio, with rendered previews and an interactive 3D viewer in the browser. It is built for miniature collections: the kind of library that arrives as a monthly release of a few hundred presupported figures.

## Folder layout

Add a `models/` folder at the root of your library, organized by creator, release, or whatever grouping you use:

```
models/
└── Cosmere Minis/          ← shown as a group header in the model browser
    ├── agent_presupported.stl
    ├── agent_unsupported.stl
    └── Brightlady_base_presupported.stl
```

The folder name is used as a group header, exactly like maps, tokens, and audio.

## Supported formats

Formats differ in what Grimoire can do with them, because a 3D file can be a bare mesh, a container that references other files, or a sliced job for one specific printer.

| Format | Preview | 3D viewer | Notes |
| --- | --- | --- | --- |
| `.stl` | Yes | Yes | The common case for miniatures |
| `.3mf` | — | Yes | Container format |
| `.glb` | — | Yes | Self-contained glTF |
| `.ply` | — | Yes | Mesh format |
| `.obj` | — | — | References a sibling `.mtl` and textures by name |
| `.gltf` | — | — | Non-binary glTF; references an external `.bin` and textures |
| `.lys`, `.ctb`, `.cbddlp`, `.pwmx`, `.photon` | — | — | Sliced printer output, not geometry |

Everything listed is indexed, browsable, taggable, and downloadable. The columns only describe whether Grimoire can *draw* the file.

`.obj` and `.gltf` are deliberately excluded from the viewer despite being real geometry: neither is a single self-contained file, so displaying one on its own gives an untextured or broken result. `.glb` is the self-contained spelling of glTF and works.

Sliced files (`.ctb` and friends) are an image stack for one printer, not a model. They are stored and served so they stay alongside the mesh they came from, but there is nothing meaningful to render.

Archives (`.zip`, `.rar`, `.7z`, `.tar` and its compressed spellings) in the models tree are registered too, since releases are often distributed zipped. They are treated as opaque: no preview, no viewer, download only.

## Previews

`.stl` files get a rendered preview, so a model grid looks like a model grid rather than a wall of identical icons. Grimoire rasterises the mesh itself in software — no GPU, no extra dependencies, nothing to install. Other formats show a placeholder.

Previews are drawn in the model's **print orientation**: Z up, the way it sits on the build plate. A miniature stands on its base rather than lying on its side.

Rendering cost tracks the triangle count — a couple of seconds for a typical figure, longer for a photogrammetry scan. The mesh is streamed rather than loaded into memory, so even a 14-million-triangle model renders in a flat few dozen megabytes. Models up to 20 million triangles get a preview.

Only the heaviest meshes are **queued rather than rendered during the scan**, the same bargain [OCR](/guide/ocr) strikes for scanned books: the library walk finishes at full speed, and those previews are drawn afterwards in a **Rendering model previews** phase you can watch in **Settings → Maintenance**. A stop or a restart leaves the rest queued, and they resume next time.

## Viewing a model

Opening a model renders it in an interactive 3D viewer — drag to orbit, scroll to zoom, with a wireframe toggle and a reset-view button. The viewer is loaded on demand, so it costs nothing until you open a model.

A mesh over **256 MB** is not loaded automatically, since that is past what a browser tab renders comfortably. Grimoire warns you that it may be slow or unresponsive and lets you load it anyway if you want it, or download it instead. Formats with no viewer (sliced files, `.obj`, `.gltf`) offer a download directly.

## Presupported vs unsupported

Resin miniatures usually ship twice — once with printing supports attached and once without. Grimoire reads this from the file name or the folder above it and badges each model accordingly.

Recognised spellings include:

- **Presupported:** `presupported`, `presup`, `supported`, `supports`, `sup`
- **Unsupported:** `unsupported`, `unsup`, `no supports`, `raw`

Separators are flexible, so `Pre_Supported/`, `goblin-presup.stl`, and `no supports/` all work. An "unsupported" spelling always wins over "supported", so `agent_unsupported.stl` is read correctly rather than matching on the `supported` substring inside it.

A model Grimoire cannot classify is left unmarked rather than guessed at. You can always set it yourself from the model's detail page — the **Supports** row there cycles presupported → unsupported → unknown.

Pair the two copies with the **presupported** and **unsupported** version kinds to collapse them into a single library entry instead of two near-identical rows.

## Tagging

Drop a `tags.json` file into any `models/` folder or subfolder to auto-apply tags on scan, just like maps, tokens, and audio. See [Tags](/guide/tags).

Folder tags can also be set from the UI, and applied to many folders at once.

## Search

Models are matched in global search by filename, folder path, and tag. See [Search](/guide/search).
