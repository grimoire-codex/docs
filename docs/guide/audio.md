# Audio Library

Grimoire browses ambient tracks, soundscapes, music, and sound effects alongside your books, maps, and tokens, with in-browser playback and a persistent player that keeps playing while you navigate. Short sounds get a [soundboard](#soundboard) of their own, playing over the queue rather than interrupting it.

## Folder layout

Add an `audio/` folder at the root of your library, organized by category or creator:

```
audio/
└── Ambient/                ← shown as a group header in the audio library
    ├── cover.jpg           ← optional folder artwork (cover.* or folder.*)
    └── tavern-night.mp3
```

The folder name is used as a group header, exactly like maps and tokens.

## Supported formats

`.mp3`, `.ogg`, `.opus`, `.flac`, `.wav`, `.m4a`, `.aac`

On scan, Grimoire reads each track's embedded **duration** and **title / artist / album** tags.

## Artwork

For a track's artwork, Grimoire uses, in order:

1. A `cover.*` or `folder.*` image in the track's folder, if present.
2. The track's embedded album art.

## Global audio player

Playback happens in a persistent **pop-out player** that keeps playing while you move around the app. Build a local queue by:

- Playing a whole folder at once
- Queueing tracks one at a time with **Play Next** — which turns into a check once a track is in the queue, so a glance tells you what you have already lined up
- Having a GM play a campaign resource group
- Playing all the audio embedded in a campaign wiki note (a note with several tracks plays as a playlist via **Play all**)

Expand the player to see and reorder upcoming tracks, with a repeat-current-track toggle.
Once a queue is the way you want it, the **save** button in the queue header keeps it as a
named playlist — see [Saved playlists and soundboards](#saved-playlists-and-soundboards).

## Soundboard

The player queue suits long tracks — ambience, music, a scene's backing loop. Short
sounds want something different: a door slam or a thunderclap needs to fire *now*,
over whatever is already playing, without disturbing it. That is the **soundboard**.

Any track can go to either destination. The **Add to soundboard** button sits next to
**Play Next** on audio rows and on a track's detail page, and multi-select adds a whole
selection at once — select the tracks you want, then **Add to Soundboard** in the action
bar at the bottom.

Each sound gets its own button in a grid. Tapping one plays it over the queue *and* over
other pads, so several can overlap; tapping a pad that is already sounding restarts it.
Every pad has its own **loop** toggle in its corner, and the panel's stop button silences
everything at once.

### Moving and shaping the board

The panel starts in the bottom-right corner and can be dragged anywhere on the page by
its title bar. Its position, its pads, and its grid size are remembered — the board is
still there after you navigate away, and after a restart.

The **configure** button (the gear) opens everything that changes the board's shape:

- **Grid size** — anywhere from 1 to 8 columns by 1 to 15 rows. A 5×5 square, a 3×8
  block, a single column of 15 down the side of the screen: whatever suits your table.
  More pads than the grid shows simply scroll.
- **Rearranging** — drag pads into the order you want them.
- **Removing** — each pad gets an **×** to take it off the board, and **Clear board**
  empties it in one go.

::: tip Nothing destructive sits on a live pad
Remove and clear appear only while you are configuring the board. Mid-session, a pad is
just a pad — there is no delete button next to the sound you are reaching for.
:::

Closing the panel leaves a small button in the corner to bring it back.

The board you are working on is stored in your browser, on the machine you built it on:
it survives a reload and a restart, but it does not by itself follow you to another
device. To keep a board for good — and to reach it from anywhere — save it as a named
soundboard, below.

## Saved playlists and soundboards

The queue and the soundboard are each a single, live thing: one queue, one board. That
is fine while you are running a scene, and awkward everywhere else. A tavern board built
for tonight has to be torn down to make room for the boss fight, and building the tavern
again next session means starting over.

Saving fixes that. Both the player queue and the soundboard have a **save** button — in
the queue header, and in the soundboard's title bar — which asks for a name and keeps
what is currently loaded:

- For a **playlist**, the tracks and their order.
- For a **soundboard**, the pads, their order, each pad's loop setting, and the grid size.

Saved sets are stored on the server against your account, not in the browser, so a board
built at home is waiting for you at the table, on whatever device you bring.

### Loading one back

The **Saved sets** button on the Audio page lists everything you have kept — playlists
and soundboards together, each with its track or pad count. From there you can:

- **Load** a set into the live player or board.
- **Add a playlist to the queue** instead of replacing it, so the next scene lines up
  behind the track that is still playing.
- **Rename** or **delete** a set.

Loading replaces what is currently loaded, so if there is something there to lose,
Grimoire asks first.

::: tip Saving over a name updates that set
Re-saving under a name you have already used updates that set rather than leaving you
with two called "Tavern". A playlist and a soundboard *may* share a name, though — a
"Tavern" of each is a perfectly reasonable pair.
:::

### When tracks go missing

A saved set points at tracks in your library rather than keeping its own copy of them.
Two things follow from that:

- Retitle a track in the library and the new title shows up everywhere it is saved.
- Remove a track from the library and the sets referencing it still load — the missing
  entries are skipped, and Grimoire tells you how many it left out.

So a set built a year ago still works after a library reshuffle, rather than failing or
leaving you with dead pads.

::: warning Saved sets are yours alone
A saved playlist or soundboard belongs to the account that made it. There is no way to
hand one to another user or attach it to a campaign yet.
:::

## Tagging

Drop a `tags.json` file into any `audio/` folder or subfolder to auto-apply tags on scan, just like maps and tokens. See [Tags](/guide/tags).

## Search

Audio tracks are matched in global search by filename, folder path, and tag. See [Search](/guide/search).
