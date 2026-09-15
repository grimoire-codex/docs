# VTT Editor

Draw the walls, doors, and lights a virtual tabletop needs for dynamic lighting, on any map you already have — then export a Universal VTT file that drops into Foundry, Roll20, or anything else that reads the format.

This is the part of a `.uvtt` that normally takes a separate tool like Dungeondraft to produce. Grimoire does it in the browser, against the map you are already looking at.

## Nothing is written to your library

Everything you draw is saved **against the map inside Grimoire**, not beside it on disk. Your original image is never modified, no extra file appears next to it, and an existing `.uvtt` is never rewritten. The export file is built fresh each time you ask for it.

That has two consequences worth knowing up front: the editor works perfectly on a **read-only library mount**, and an edited map is still an ordinary map — add it to a campaign, download it, or re-export it whenever you need the file.

## Opening it

| From | How | You get |
| --- | --- | --- |
| A map's detail page | **Edit VTT** | The map, on whatever geometry it already carries |
| The maps page | The **VTT Editor** button in the toolbar | An empty editor waiting for a file |

**Editing a `.uvtt` you already have** works the same way. A Universal VTT file is not just viewable — it opens in the editor on the walls, doors, and lights it already carries, so you can move one wall that is wrong instead of redrawing the map. Exporting gives you a new file carrying the original's own image alongside your edits.

When a map image and a `.uvtt` are **linked as versions of each other**, **Edit VTT** becomes a menu and asks which you mean:

| Choice | Does |
| --- | --- |
| **Edit the map image** | Draws new walls, doors and lights over the picture |
| **Edit the linked VTT** | Changes the walls, doors and lights the `.uvtt` already carries |

Picking one for you would quietly send half of everyone to the wrong document, so both are named. Only PDFs and videos are left out — they have no single image to draw on, and the button simply does not appear.

### A map that is not in your library yet

The **VTT Editor** button on the maps page opens the editor with no map at all. Drop in a map image or an existing Universal VTT file — from your desktop, a purchase you have not filed yet, anywhere — and edit it without adding anything to your library.

It accepts PNG, JPEG, WebP, GIF, `.uvtt`, and `.dd2vtt`. The file stays in your browser until you download it or send it to a campaign; nothing is uploaded to your library.

When you are done, **download** the `.uvtt`, or **send it to a campaign** you run as a linked resource, choosing the category it should be filed under.

## Confirm the grid first

Everything you draw is measured in grid squares, so the editor opens on the grid rather than the drawing tools. Your map appears with the grid Grimoire detected drawn over it, and you can see at a glance whether it lines up.

**Zoom in to check it.** At fit-to-window on a large map, one screen pixel covers several of the image's, and a grid that is half a cell out looks perfect.

If it does not line up, two ways to fix it:

| Tab | Set | Good for |
| --- | --- | --- |
| **By size** | Cells across and cells down | How most people already know their grid — the overlay redraws as you type |
| **Fine-tune** | Cell size in pixels, plus offset X and Y | Walking an almost-right grid into place, a pixel at a time, with nudge buttons |

Then **Confirm grid**. You can come back later with **Recalibrate grid** in the sidebar, but Grimoire warns you first: changing the grid moves everything already drawn, because the geometry is stored in grid units rather than pixels.

## Drawing

| Tool | Placed by | Is |
| --- | --- | --- |
| **Wall** | Click each corner, double-click or `Enter` to finish | A vision-blocking wall |
| **Object wall** | Same | A separate layer for furniture, pillars, and scatter |
| **Door** | Two clicks, across the opening | Blocks sight until opened at the table |
| **Window** | Two clicks | Can be seen through, but not passed |
| **Light** | One click | A light source, which opens for editing straight away |
| **Select** | Click a feature | Picking something to change or delete |

Close a room by ending a wall on the point you started from.

**Object walls are their own layer on purpose.** Virtual tabletops treat them differently — Roll20 turns them into transparent barriers rather than solid walls — so furniture is a genuinely different thing to draw, not a style of wall.

**Doors and windows are separate tools** for the same reason of directness: drawing a window used to mean placing a door, switching to select, clicking it, and flipping a toggle. Four steps to say something you knew before the first click. A door can also be marked **Freestanding** — an archway or a standing screen, not set into a wall.

### Snapping

Three choices, in the **Snap** section:

| Mode | For |
| --- | --- |
| **Grid** | Ordinary rooms and corridors |
| **Half** | A diagonal, or a split doorway |
| **Free** | An irregular cave wall |

Hold **Alt** to place a single point off the grid without leaving your snap setting — for a room that is square apart from one canted corner.

### Getting around

| Input | Does |
| --- | --- |
| Scroll | Zoom |
| Drag with the middle or right button | Pan |
| `Escape` | Abandons the shape you are drawing |
| `Enter` | Finishes a wall |
| `Delete` / `Backspace` | Removes what is selected |
| `Ctrl`/`Cmd`+`Z` | Undo |
| `Ctrl`/`Cmd`+`Shift`+`Z` | Redo |

Whichever tool you pick, the panel on the right tells you how to drive it. None of these gestures are discoverable — nothing on screen would otherwise tell you a wall ends on a double-click, which leaves you stuck mid-wall with no visible way out — so the help is shown for the active tool rather than hidden in a manual.

## Lights

A light is placed with a single click and opens for editing immediately: the panel at the top of the sidebar shows the new light's settings, so you do not have to select it again.

Start from a **preset**, picked from a menu or a row of colour swatches:

| Group | Presets |
| --- | --- |
| Small flames | Candle, Lantern |
| Room fixtures | Wall sconce, Hearth, Torch |
| Large fires | Brazier, Campfire, Bonfire, Daylight |
| Cool and magical | Moonlight, Swamp glow, Arcane, Gloom |
| Neutral | Ambient, Shade |

Then adjust **range** (in grid squares — not pixels or feet), **colour**, **opacity**, **intensity**, and whether it **casts shadows**. Tune any of them and the picker simply reads as **Custom**; set values back onto a preset's numbers and its name comes back, including on a light that arrived in an imported map.

::: tip Why the presets look dark in a colour picker
The preset values are measured from what Dungeondraft and its kin actually write — roughly 1,900 lights across real maps, where only 79 distinct range/colour/intensity combinations account for all of them. Authored light colours are **dark and strongly saturated**, because `intensity` does the brightening; a "bright orange" torch colour double-counts it and blows out. A torch therefore starts out looking like a torch.

The format stores no name or type for a light, so the names are Grimoire's inference from each value's character. The numbers are real.
:::

### Map-wide lighting

The **Environment** section sets how bright a spot with **no light of its own** looks, with a swatch showing the result. Black is a pitch-dark dungeon; a dim blue reads as moonlight.

You can also mark a map whose **lighting is already painted into the artwork**. Tick it if the picture shows its own torchlight, shadows, or glow — a virtual tabletop may then ignore or dim the lights you place, so they do not double up. Grimoire warns you if you have done both.

## Checking it from a player's chair

**Show player view** drops a token on the map and darkens everything that token could not see. Walls cast real shadows, a closed door hides the room behind it, and a window does not. It answers "can my players see around that corner?" in place, rather than by exporting, importing, and moving a token in another program to find out.

Drag the token, or walk it with the arrow keys — hold `Shift` for half a square.

Its sight is described the way a virtual tabletop describes one, with three independent switches:

| Switch | Does |
| --- | --- |
| **Vision** | The master switch. Off, the token sees nothing at all — which is what an object or scenery token is set to |
| **Night vision** | Lets it see without any light, out to a distance you set. This is darkvision, and it is the only one of the three with a real range |
| **Token light** | The torch the token carries, which lights the ground around it for everyone |

Night vision is tinted a cool blue in the preview, so you can tell ground that is merely *visible* from ground that is actually *lit*.

Sight itself is not capped by a number of squares. What stops you seeing is walls and darkness. A token with vision but no night vision and no torch sees only where a placed light reaches — and Grimoire tells you so rather than leaving you staring at a black screen.

Lights you have placed light the **room** they are in, not just the square they sit on: stand anywhere within a lamp's range and you see by it, and you see the ground it falls on from across the room. Walls still apply — a lamp behind a shut door stays behind it, and light spilling around a corner reveals only floor you can actually see. The ambient level you set for the map drives how dark the unlit area looks here too, so a map authored with moonlight previews as moonlight.

The preview is a toggle, and **off by default**, because a darkened map is in the way while you are tracing walls.

::: warning The token is preview only
A `.uvtt` has no concept of a player token. The token and its three vision settings are never saved and never reach the exported file.
:::

## Exporting

**Export** downloads a `.uvtt` carrying the map image, the grid, and everything you authored. Any image map can also be exported straight from the **Download** menu on its detail page, without opening the editor — that file carries the image and grid alone.

A map that already has a real `.uvtt` linked to it does not offer the plain export, since the file you have already carries walls and lighting of its own.

## What the editor deliberately leaves out

The editor offers only what the format can actually carry, so nothing you set is quietly dropped on export. That is why there is no wall thickness, no secret or locked door, no one-way wall, and no light animation — **none of them exist in a `.uvtt`**. Roll20 and Foundry let you add things like secret doors after importing, and that is the right place for them.

For the same reason, a selected wall shows no properties beyond its point count: Universal VTT walls carry no thickness, type, or height.

## Who can use it

Reading is open to anyone who can see the map — the editor doubles as the way to inspect what a map carries. **Saving requires the GM or admin role**, like every other map edit.

The standalone editor on the maps page is the exception that proves the rule: it saves nothing to your library, so it is bounded only by what you can do with the resulting file.

## Fixing a grid outside the editor

Grimoire works the grid out on its own, from a `(30x40)` in the filename, the image's DPI, or the pixel dimensions. When it gets that wrong, the **Grid** panel on the map's detail page is editable: set the width and height in cells, and optionally the pixels per cell.

Fractional values are accepted, because plenty of maps bleed a partial cell past the grid — a 33×24 map with a quarter-cell margin at each edge is really 33.25×24.25. If your numbers imply cells that are not square, Grimoire says so and shows what the image suggests instead, but still saves what you typed: unusual maps exist, and you are the one who can tell. **Reset** puts the map back to automatic detection.
