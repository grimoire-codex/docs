# OTFBM and Avrae

[OTFBM](https://otfbm.io) (On The Fly Battle Maps) draws battle maps as images for play-by-post games, most often through [Avrae](https://avrae.io)'s `!map` command on Discord. Any image map in your library can be set up for it in a few clicks — no more guessing cell sizes until the grid looks close. The option is there for anyone allowed to create [share links](./sharing), since that's how OTFBM gets the picture.

## Why the image needs resizing

OTFBM draws your picture at its real size and lays its own grid over the top, at the cell size you give it with `-options c<size>`. It does not stretch the picture to fit. A typical battlemap is drawn at 140 pixels per square, so under OTFBM's default 40-pixel grid it lines up with nothing.

Grimoire already knows the map's grid, so it resizes the image until each of the map's squares is exactly the cell size, serves it through a share link, and hands you the command that matches:

```
!map -mapsize 33x24 -options c70 -bg https://grimoire.example.com/api/share/…/tavern-33x24c70.jpg
```

## Using it

On a map's detail page, open the **⋮** menu and choose **Set up for OTFBM (Avrae)**.

| Step | What to do |
| --- | --- |
| **1. Square size** | Pick 20–100 pixels. Grimoire suggests 70, or the map's own square size if that is smaller — enlarging a picture adds file size, not detail. The dialog shows the grid and image size as you change it. |
| **2. Create the share link** | Choose how long it lasts — *never* by default, since a play-by-post battle can run for weeks and OTFBM fetches the background every time it draws the map. A link you already made at the same square size is reused. |
| **3. Copy the command** | Run it in Discord. |

The link is an ordinary [share link](./sharing) that serves the resized picture rather than the original. It's listed under **Settings → Connections → Share links** as "OTFBM background", and revoking it there takes the background down. Nothing is written to your library.

::: tip Not reachable from the internet?
OTFBM fetches the background from your Grimoire server, so it has to be reachable from the internet. If it isn't, **download the resized image** from the same dialog and host it somewhere OTFBM can reach — see OTFBM's list of [image hosts that work](https://docs.otfbm.io/#/pages/image-hosts) (GitHub, Flickr and Pinterest originals do; Discord and Imgur don't) — then put that URL after `-bg` yourself. The download is a JPEG, or a PNG if the map has transparent areas; OTFBM doesn't read WebP.
:::

## Getting the grid right

The export uses the same grid as the [VTT Editor](./vtt-editor)'s Universal VTT export: the grid you set by hand if there is one, otherwise the one detected from the filename or image, otherwise 140 pixels per square. If the dialog says no grid was found, or the squares do not line up in OTFBM, correct the grid in the map's details first. The link always serves the map at its current grid, so open the dialog again afterwards for the updated command.

- **Partial squares** — maps often carry a fraction of a square of margin. A map 33.25 squares across becomes a 33-square OTFBM map, with the margin left in the image.
- **Large maps** — OTFBM shows at most 99 squares per side. A bigger map is capped at 99, and the dialog warns that its far edge will be cut off.
- **Universal VTT files** — a `.uvtt` exports too, using the picture inside it and its own grid.
- **Linked versions** — unlike the Universal VTT export, a map linked to a `.uvtt` can still be exported for OTFBM, since OTFBM has no walls or lights to lose.
- **Offsets** — if a map's grid does not start at its top-left corner, OTFBM's `-options o<x>:<y>` can shift it. Grimoire does not set this for you.
