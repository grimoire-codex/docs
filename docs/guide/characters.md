# Characters

Grimoire has a built-in **character builder**. You install a sheet for your
game, build characters on it, and the sheet does the arithmetic as you fill it
in — modifiers, saves, totals, spell slots. **Characters** sits under Campaigns
in the sidebar, with each character shown as a card with its art.

The builder is **schema-driven**: a sheet is a document describing its fields,
the values worked out from them, and how it is drawn. There is no game-specific
code in Grimoire, so the same engine draws D&D, Draw Steel, Pathfinder, Cairn,
or a system you invent this evening.

::: tip Not the same as campaign PDF sheets
A campaign member can still upload a PDF or image character sheet to a campaign
(see [Campaigns](./campaigns#characters-art-and-sheets)). The character builder
is separate: its sheets are live, calculate their own values, and live under
**Characters**.
:::

## Sheets in the catalogue

The [community add-ons](https://github.com/grimoire-codex/community-add-ons)
repository carries ten sheets, each under the licence its game is published
with:

| System | Layout | Content pack |
| --- | --- | --- |
| Dungeons & Dragons 5e (2024) | Custom | D&D 5e SRD 5.2 |
| Draw Steel | Custom, after the printed sheet | Draw Steel: Heroes |
| Pathfinder 2e | Custom | Pathfinder Player Core |
| Pathfinder 1e | Custom, after the 2009 sheet | |
| Call of Cthulhu 7th Edition | Custom | |
| Traveller (Mongoose 2nd Edition) | Custom | |
| Cosmere RPG | Custom | |
| Dungeon Crawler Carl | Custom, after the printed sheet | |
| Cairn | Default | |
| Basic Fantasy RPG | Default | |

A *custom* sheet follows its published original's content and order. A
*default* sheet is drawn in plain sections — perfectly usable, and all a
rules-light game needs. Each works out the arithmetic its system actually uses:
D&D scales proficiency into saves and passive Perception, Pathfinder adds your
level only once you are trained, Traveller gives every skill a DM against each
characteristic.

## Installing a sheet

**Characters → Manage sheets** opens on **Sheets**:

- **Browse sheets** lists the community catalogue, with each sheet's licence and
  credit shown before you install it.
- **Paste a sheet** takes one you wrote or were sent: the sheet itself (JSON or
  YAML - without anchors and aliases), and optionally its layout HTML and stylesheet.

Sheets are **per user**, like themes — installing one changes nothing for anyone
else, so no admin approval is involved, and guests can install them too. A
downloaded sheet is checked against the catalogue's digest and validated before
it is stored, and nothing in a sheet runs. Uninstalling a sheet keeps the
characters built on it: they open showing their stored values until you
install it again.

### Updating a sheet

Every character follows your installed copy of its sheet, so updating the sheet
updates all of them. When the catalogue has a newer version of a sheet you
installed from it, Grimoire offers the update in three places:

- A notice at the top of the **Characters** page names each sheet with your
  version and the new one. **Update all** installs every update in one go.
- **Manage sheets** shows an **Update to v…** button beside the sheet.
- **Browse sheets** shows the same button in place of *Installed*.

A sheet you pasted is never offered an update, even if the catalogue has one
with the same id, because it may be your own edited copy. To switch it to the
catalogue's version, install it from **Browse sheets**.

## Building a character

**New character** asks for a name, the sheet, and optionally a campaign. Then
fill in the sheet. Edits save as you type.

- **Derived values recalculate live.** A modifier updates the instant its score
  changes.
- **Warnings** appear when the sheet notices something — more spells prepared
  than your maximum, say. They never block saving.
- **Art** sits in the sheet's header: click the empty thumbnail to add some.
  Some sheets also draw it large on a page - the D&D 5e sheet puts it on Spells
  & Details.
- **Click the art to see it properly.** It opens large, and on your own
  character this is where you replace or remove it and choose its **framing**:
  click or drag on the picture to pick the part that stays in view, and zoom in.
  The header, your character list and the sheet all keep that point showing, so
  tall art keeps its head.
- **Some sheets hold other pictures** - a knight's coat of arms, a ship. Add,
  replace or remove one where the sheet shows it (PNG, JPEG, WebP or GIF, 5 MB).
- **Picked entries open.** Click a feat or a spell to read the whole entry.
- **Choices fill in the rest.** On the D&D 5e sheet, picking a class sets your
  saves, hit dice and spellcasting ability, then asks for its skill choices;
  picking a background adds its skills and origin feat. Swap the pick and only
  what it added comes off again.

### You can set any value

Everything a sheet works out is a suggestion — the table is the authority, not
the formula:

- A value filled in from your choices is marked **auto**. Type over it and it is
  yours; the reset button beside it hands it back.
- A calculated value — Armour Class, a save DC — can be clicked and set, and
  everything built on it follows.
- **All values**, in the sheet's header, lists everything the character stores
  and calculates, whatever the sheet's layout shows.

None of this needs content installed. With no class or species loaded — a
homebrew class you are playing from your own notes, say — every one of these is
an ordinary field you fill in.

### What the builder does not do

Grimoire keeps your character's **record** and does its **arithmetic**. It does
not run a game's character creation *procedure* — careers, lifepaths, rolling on
random tables. Those belong at the table with the book; the sheet holds what
they produced.

## Characters and campaigns

Put a character in a campaign and everyone at that table can **read** its sheet
— so a GM can see what the party is playing. Only the player who made it can
edit it, unless they [let the GM edit](#letting-the-gm-edit).

You can put a character in any campaign you are at the table for:

- **One you run** — a GM campaign, or a [personal campaign](./campaigns#personal-campaigns)
  you keep for your own notes as a player.
- **One you have joined.** A pending invitation does not count until you accept
  it.

Choose the campaign when you create the character, change it any time from
the dropdown under the character's name in the sheet's header, or press **Add
character** in the **Characters** card on the campaign's page. **No campaign**
makes the sheet private again.

A GM can put their own characters in a campaign they run too — NPCs, or the
whole party of a solo game — and a GM-only campaign with no players works the
same way.

### On the Characters page

Your own characters come first. Other players' characters you can read through
a campaign are listed below them under **Party members**, one group per
campaign in order of campaign name, and each card names its player. Click a
campaign's name to fold its group away; Grimoire remembers which groups you
folded in this browser.

The **Sort** menu orders your characters by when you last edited them, by
name, by system, by **campaign**, or by sheet and version. Sorting by system or
campaign puts each one under its own heading. Characters still being played
always come before retired and fallen ones.

### On the campaign page

The campaign's page has a **Characters** card listing every character in the
campaign, grouped under the player who owns it: the GM first, then you, then
everyone else. Click a character to open its sheet.

Each player shows only the characters they are playing now. Their retired and
fallen characters fold away behind a **past characters** toggle under their
name, so a player on their tenth character still takes up one line until you
want the history.

### Letting the GM edit

Tick **Let the GM edit** in the sheet's header and the campaign's GM can edit
your character too — fix a typo, take off hit points, add the loot they handed
out. It is off for every new character, and works the same whatever sheet the
character is built on.

- The GM edits the sheet itself: its values, the character's name, its art and
  the pictures on it. Where the character is played, its status, and this
  setting stay yours, and only you can delete the character.
- Only the GM who runs the campaign gets in. The other players at the table
  still just read the sheet.
- Moving the character to another campaign turns it off, so a new GM has to be
  let in again.
- It is only offered when the campaign has a GM besides you — not for a
  character in no campaign or in your own personal campaign.

### Giving a character to a player

A GM can prepare characters for their players: pregens for a one-shot, a
replacement mid-campaign, or a sheet set up for someone new to the builder.
Create the character in a campaign you run, then pick a player from **Give to
player…** in the sheet's header. The menu lists the players who have joined
that campaign, guests included. It does not appear for a character in no
campaign, or in a campaign someone else runs.

The character becomes the player's, as if they had made it:

- It stays in the campaign, with its art and pictures.
- **Let the GM edit** is switched on, so you can keep adjusting it. The player
  can switch that off like any other time.
- If the player hasn't installed the sheet, your copy is installed for them.
  If they already have it, their copy is used, even an older one.
- Content you picked from your own *Only you* rulesets is copied into a new
  personal ruleset for the player, so the character still reads correctly for
  them. Content they can already read is left as it is.

There's no undo. To get a character back, ask the player to export it for you.

### More than one character

Characters die, retire, and get replaced, so a player can have any number of
characters in the same campaign. Instead of deleting the old one, set its
**status** — the second dropdown in the sheet's header — to **Retired** or
**Dead**. It stays readable, greyed out on the Characters page and listed after
the characters still being played, and on the campaign page it moves under that
player's **past characters**.

## Content packs and rulesets

Fields like a class or a spell list open a **catalog** you can search and
filter. A character **references** an entry rather than copying it, so an
erratum reaches every character that chose it. Anything the catalog lacks can be
added by hand with **Add custom**.

The catalog is filled by two things, both managed under **Characters → Manage
sheets → Content**:

- **Content packs** are a system's published content — the SRD's spells,
  classes and feats. They are server-wide, so **an admin installs them**, from
  **Browse content packs**. Anyone can browse, so a GM can see what a pack offers
  before asking for it. Each file is checked against the catalogue's digest.
- **Rulesets** are editable sets of content. A ruleset **belonging to a
  campaign** is read by everyone at that table and edited by the GM who runs it;
  a ruleset **for the server** is available in every game and only an admin can
  create one. Two games in the same system can therefore allow different
  content. A **personal** ruleset, labelled *Only you*, is yours alone — nobody
  else can read it, admins included.

Getting content into a ruleset:

- **Import a content pack** in a click once it is installed. Its licence and
  credit are copied along with it.
- **Fork an entry** to change it. Pack content is read-only, so editing a spell
  means taking a copy, which remembers where it came from.
- **Write one by hand**, or **Import a document** someone exported (JSON or
  YAML). An import keeps what you already have unless you choose to replace it.

Deleting an entry never damages a character: the sheet shows it as missing, and
it reads correctly again if the entry comes back.

::: tip Installing packs by hand
An admin can also copy a pack's directory into `character-content/` inside the
data directory, then use **Settings → Add-ons → Content packs →
Reload from disk**.
:::

## Sharing a character

**Export** in the sheet's header writes a self-contained file: every entry the
character uses is embedded, and the sheet definition travels with it, so it
opens on a server that has neither the pack nor the ruleset. The art, its
framing and any other pictures on the sheet travel with it too. Import it from
**Characters → Import a character**. The receiving server prefers its own copy of
each entry, so a shared character picks up local corrections. Anything the file
carries that the server lacks goes into a new **personal** ruleset for the
person importing it, so it is never shared with anyone else — though a party the
character joins still sees the sheet whole.

## Importing from other tools

**Characters → Import a character** also reads characters built in other tools,
through **character importers** - community [add-ons](./addons). There are hundreds of character
builders, each changing its format on its own schedule, so Grimoire knows none
of them itself: each is a small community definition that can be fixed without
a Grimoire release.

**Getting one.** The dialog lists the importers you have and, under **More
importers**, the ones you could add. One that only reads files you upload -
Forge Steel and Pathbuilder are both this kind - you can **Install** yourself,
for your account only, just like a sheet. One that fetches from a link or runs a
script needs an admin: they see **Install for everyone**, everyone else sees
**Ask an admin to install this**.

Drop a file on the dialog or choose one, and Grimoire works out which importer
it belongs to from its extension and contents. An importer that reads links
adds a paste box. **Read it as** is only needed when two importers both claim a
file. The dialog lists the importers your server has, and the sheet each one
builds on.

| Importer | Reads | Builds on |
|---|---|---|
| Forge Steel | `.ds-hero` files from Forge Steel's **Export** | Draw Steel |
| Pathbuilder 2e | **Menu → Export → Export JSON** - open its link in your browser, save the page as a `.json` file, and upload that | Pathfinder 2e |

- **The sheet comes with it.** If you have not installed the importer's sheet,
  the import installs it from the catalogue.
- **Picks become references.** A class, a spell or a feat is matched by name
  against your content packs and rulesets, as if you had picked it yourself.
  Anything your content lacks is kept as a custom entry, with the export's
  description where it has one.
- **Derived values stay derived.** Stamina, hit points, AC and speed follow from
  your class, kit and ancestry, so the import doesn't freeze them.
- **It's a one-time copy.** Editing the character never reaches back to the
  tool it came from.

Some sources need something from you - an API key for a private character, say.
An importer that does shows the field once it is chosen; if an import turns out
to need it, the dialog picks that importer, marks the field, and offers **Try
again**. What you type goes with that one import and is never stored.

::: tip Pathbuilder links
Pathbuilder's site only answers browsers, so Grimoire can't fetch an Export JSON
link for you. Open the link, save the page, and upload the file.
:::

## Writing your own sheets and content packs

A sheet you write works exactly like a catalogue one — paste it under **Manage
sheets → Paste a sheet**. To share it with everyone, contribute it to the
community repository:

- [Adding a character sheet](https://github.com/grimoire-codex/community-add-ons/blob/main/character-sheets/README.md)
  — a walkthrough from the first field to the pull request.
- [Character sheet reference](https://github.com/grimoire-codex/community-add-ons/blob/main/docs/character-sheets.md)
  — every field type, formula function, layout directive, HTML tag and CSS
  property a sheet can use.
- [Adding a content pack](https://github.com/grimoire-codex/community-add-ons/blob/main/content-packs/README.md)
  — the spells, classes and feats a sheet draws from, and the licensing they
  need.

Content for one table rather than everyone belongs in a ruleset instead, which
needs no pull request.

## Using a different catalogue (admin)

The sheet and content-pack catalogues follow the add-on index set under
**Settings → Add-ons**. That setting takes a **comma-separated list**, so you
can offer several catalogues at once — the official one alongside a friend's
fork. Sheets from every source appear together, labelled with where they came
from, and a source that cannot be reached is called out rather than quietly
left out.

A source can also be a **private repository**, for licensed or non-SRD sheets
and content packs that should stay within your table. Give it an access token
and its sheets and packs install and update like any others - see
[Using a private repository](/guide/addons#using-a-private-repository).
