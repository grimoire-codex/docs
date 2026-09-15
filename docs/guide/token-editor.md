# Token Editor

Turn any picture into a VTT-ready token without leaving Grimoire — crop it to a circle, drop a frame on it, and either download the PNG or make it a character's portrait.

Nothing is written to your library and no new token is indexed. The image is composed in your browser and only leaves by one of two doors: a download, or a character portrait. That means the editor works on a read-only library and never disturbs a scan.

## Opening it

Three ways in, depending on where you start:

| From | How | Best for |
| --- | --- | --- |
| The tokens page | The **Token editor** button in the toolbar | A picture on your phone or desktop |
| A token's page | The **Token editor** button beside the download controls | Framing art already in your library |
| A campaign member | Click a character's portrait, then **Create a token…** | Making a character's own token |

The third route is the quickest, because it already knows which character the finished token is for — no picking afterwards.

## Choosing the art

Load an image by uploading a file, pasting from the clipboard with `Ctrl`/`Cmd`+`V`, dragging it onto the page, or browsing your token library. Browsing shows your tokens grouped by the folders you filed them in, each collapsible, rather than one flat wall of thumbnails — searching switches to a single result list, since matches spanning many folders read better that way.

Browsing deliberately offers the token library and nothing else. A book's thumbnail is a cover, a track's is album art, and a battlemap is scenery — none of them a character portrait, and tokens are already the collection of character images. Anything filed elsewhere is still one upload away.

If the picture is smaller than the output size you have chosen, the editor says so. A 140-pixel image blown up to a 1024-pixel token will look soft, and it is better to hear that before you drop it into your VTT.

## Positioning

| Input | Does |
| --- | --- |
| Drag | Move the art |
| Scroll | Zoom toward the pointer |
| `Shift` + scroll | Rotate |
| Arrow keys | Nudge one pixel (hold `Shift` for ten) |
| `+` / `-` | Zoom |
| `[` / `]` | Rotate 15° |
| `R` or double-click | Start over |

On a touchscreen, drag with one finger to move, and pinch or twist with two to zoom and rotate. A small twist is ignored so that pinching to zoom does not leave everything slightly tilted.

The dashed outline shows where the mask will cut, so you can see what survives even with no frame selected.

## Output

- **Shape** — decided by the frame you pick. Choose an ornate border and the art follows its real opening. For a plain round or square token with no visible border, pick the circle or square frame and set its colour to **None** — the shape still does the cropping, it just draws nothing. With no frame at all you get the full square image.
- **Size** — 140, 256, 512, or 1024 pixels. 140 is Roll20's native token size; 256 and 512 suit Foundry and most others; 1024 is for print or high-DPI displays.
- **Background** — transparent by default, since a token sits on a map. The colour swatches help when the art has awkward or ragged edges, and a custom picker is there if none of them fit.

The result is always a PNG, so transparency is preserved.

## Where it goes

**Download** saves the PNG to your device, ready for Roll20, Foundry, or anything else.

**Send to a campaign** offers whichever of two destinations you have the standing for:

| Destination | Who sees it | What happens |
| --- | --- | --- |
| **Send to a character** | The games you play in, one row per character | Becomes that character's VTT token, stored *separately* from their portrait — making a token never overwrites their artwork |
| **Send to a campaign you GM** | GMs only, one row per campaign | Pick the campaign, then either **Set as a character token** (which lists just that campaign's characters) or a resource category, defaulting to the built-in **Tokens** group |

The GM route lists campaigns rather than characters on purpose: running four games would otherwise bury the four campaigns under every player's character. Drilling into one campaign first keeps the character list to a single game's worth.

You are only shown the paths that apply to you. A player with no campaign of their own never sees the GM route, and someone in no campaigns at all is told so rather than offered a destination the server would refuse. Setting your own character's things has always been something a player can do, so **players can make their own tokens** — this is not a GM-only feature.

Opening the editor from a character's portrait is the exception: it already knows who the token is for, so the button simply sets that character's art.

## Custom frames

Five frames ship with Grimoire, in two groups.

**Two plain shapes** — a circle and a square — take any colour you pick, from the same swatches campaign icons use, plus a custom colour picker. Setting the colour to **None** keeps the shape as the crop but draws no ring, which is how you make a plain circular or square token.

**Three role markers** — player character, non-player character, and opponent — keep their own colour. They are deliberately three different *shapes* rather than three colours, because at the size a frame appears in the picker a colour difference disappears in greyscale and for colour-blind users while a silhouette does not; their colour confirms the distinction rather than carrying it.

You can add your own as well. Put an empty file named `.frames-container` in any folder under `tokens/`, and every `.png`, `.webp`, or `.svg` beside it becomes a frame:

```
tokens/
├── Fantasy Frames/
│   ├── .frames-container   ← the marker that makes this a frame folder
│   └── orc-ring.svg
├── Scifi Frames/
│   ├── .frames-container
│   └── hex-plate.png
└── Goblins/
    └── goblin.png          ← an ordinary token folder
```

This is the same convention the book library uses for `.parent-system-container` and its siblings: an empty marker file declares what the folder holding it is for, so the folder keeps whatever name reads best on disk. Frames are grouped in the picker by that folder name.

The marker works at any depth under `tokens/` — `tokens/Fantasy Frames/` and `tokens/Cyberpunk/Frames/` are both frame folders. Both the tokens page and the file manager badge such a folder with **Frames**, so you can see at a glance which folders feed the editor.

**Frames are still indexed as tokens.** A frame folder is an ordinary library folder, and its images appear in your token gallery alongside everything else. That is deliberate — a frame *is* a token image, just one you would usually composite rather than place on a map. If you would rather they stayed out of the gallery, add a `.grimoireignore` rule for the folder; the frames keep working either way.

### Finding a frame in a large collection

The picker is built to stay usable with hundreds of frames:

- **The list scrolls in place** rather than pushing the rest of the page down.
- **Each folder group collapses**, so you can close the ones you are not working from.
- **A search box** filters by frame name *or* folder name — searching `fantasy` finds everything in `Fantasy Frames`.
- **Favourites get their own group** at the top. Favouriting a frame is the same as favouriting anything else in Grimoire: star it in your token gallery. It stays listed under its own folder as well, so a frame never disappears from where you filed it.

A frame you have just dropped in cannot be favourited until the next rescan indexes it — it is still fully usable in the editor in the meantime.

### Authoring a frame

| Requirement | Why |
| --- | --- |
| A closed outline | The editor reads the crop by filling inward from the centre — a gap lets the fill escape |
| Transparent middle | A frame is a border, not a disc — a filled centre hides the art |
| Square overall | Frames are drawn to a square canvas; anything else stretches |
| SVGs need `width` and `height`, not just `viewBox` | Firefox and Safari report a dimensionless SVG as zero-sized and cannot draw it |

**The crop comes from the frame, not from a fixed shape.** Grimoire flood-fills inward from the middle of your frame and keeps whatever the frame encloses, so the art follows the outline you actually drew — a hexagon crops to six sides, a keyhole border crops to a keyhole. Nothing needs to be declared; the shape is read from the image.

The one thing that matters is that the outline be **closed**. A border with a gap in it lets the fill leak out to the edge, and the editor falls back to a plain circular crop. If your design calls for a break — the bundled non-player-character frame has a decorative notch at the top — bridge it with a small bar so the outline still encloses the middle.

The bundled frames use a `0 0 512 512` viewBox and presentation attributes (`fill`, `stroke`) rather than a `<style>` block, which makes recolouring one by hand straightforward. They are a good starting point to copy.

Frames you supply cannot be recoloured from inside Grimoire — an SVG loaded as an image has no styleable interior. The two plain shapes are generated by Grimoire itself, which is why they can take a colour; if you want a custom colour on your own frame, edit a copy of the file.

::: tip Frames are trusted content
Anything in a frame folder was put there by whoever has filesystem access to your library. Grimoire serves frames with a restrictive content-security policy and only ever draws them as images, so an SVG cannot run script — but a deliberately pathological file can still bog down the browser rendering it. Frame folders are for your own art, not a place to drop files from strangers.
:::
