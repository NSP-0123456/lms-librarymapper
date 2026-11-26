# LibraryMapper — an Lyrion (LMS) plugin

Map your large music library *inside* Lyrion Music Server, **without touching files**:

- Browse a virtual `Genre \ Artist \ Year - Album` layout as an app with categories
- **Temporal filter**: albums added on the server in the last day / week / month / year
- **Quality filter**: hi-res / CD / lossy / **1bit Hi-Res (DSF/DFF)**
- Simplify hundreds of raw genre tags into 10–20 broad groups (semi-manual curation, then automatic application)
- Consolidate album-artist spelling variants
- Parameter page with an **initialization phase**, mapping **validation**, and **download/upload** of mapping files (JSON, `librarymapper-mapping` v1)
- Compilations stay *Various Artists*; genre-less albums get derived genre + low-confidence flags

## Install (any LMS/Lyrion ≥ 7.9) — through the plugin updater

The plugin is published as a plugin repository, so Lyrion installs it by itself
(download → SHA1 check → replace → restart):

1. Open your server's web UI → **Settings → Advanced → Plugins**
2. At the bottom, in *Plugin repositories*, add the address of the manifest:
   `https://raw.githubusercontent.com/NSP-0123456/lms-librarymapper/main/extensions.xml`
   then **Apply**
3. Tick **LibraryMapper** in the list and **Apply** again (the page shows it under
   the repository you just added). Lyrion downloads the release, checks its SHA1
   and asks for a restart.
4. Restart the server (the button is offered at the top of the same page) — the
   new version is only put in place at the next start.
5. After the restart, open **Settings → Advanced → Plugins → LibraryMapper** and
   click *Initialize* — this scans the Lyrion library into the plugin's dedicated
   SQLite database and proposes the genre groups. Later LMS library scans refresh
   this cache automatically when they finish.

If the plugin is already installed in an older version, it appears in the
*Updates* section of the same page: tick it there and **Apply**.

> The archive itself is served by jsdelivr, and the manifest points at it with the
> immutable reference of the commit that holds it (so the SHA1 never changes).
> The CDN of GitHub Pages (`raw.githubusercontent.com`) is not reachable from every
> network; when it is not, use the address of the manifest on the CDN instead:
> `https://cdn.jsdelivr.net/gh/NSP-0123456/lms-librarymapper@<commit>/extensions.xml`
> (the commit is the one of the manifest, printed by the publish script).

### The application

The plugin is a real application: it is listed in the **Apps** menu of the skins
(Material included) with its own icon. The application is a **shortcut to the
choices** of the player (the menu *Filters & sorting*: order, dates, quality,
...) and nothing more; the virtual tree itself opens from **My Music >
Simplified View**.
The contents of the application open on a server where **no player is connected**.

The tile *Apps* of the home page of the Material skin is a different thing: it
asks the server for the feed `myapps items`, and the server only answers that
feed when a player is connected. On a headless server, turn on the option
**"Combine 'Apps' and 'Radio'"** in the settings of the Material skin: the tile
then uses the clientless command of the skin and shows the applications (the
separate *Radio* tile disappears, the radio stations join the *Apps* menu).


### After the restart

- **My Music → Simplified View** — the native browse node, in every skin
  (Material included), with the same drill-down as the application page.
- **Settings → Advanced → Plugins → LibraryMapper** — the parameter page:
  initialization, validation report, repropose, trusted artists,
  download/upload of the mapping file.
- `/plugins/LibraryMapper/index.html` — the browse application itself
  (`Genre \ Artist \ Year - Album` + the two filters).

## The browse page

The page of the application carries every choice of the walk in one form:
the temporal and quality filters, the order of the artists and of the
albums (with an **Unsorted** choice — no sort at all; the selector also
reads Unsorted while the expression carries a `sort:` term of its own), the
order of the groups, the number of items, a filter on the name
of the artist (transliterated when **Translit** is ticked) and a free
**Expression** (`year:1980..1990 duration:>45:00 genre:chamber
sgenre:(Jazz,Pop) track:suite trackN:5 …` — `genre` matches the genre string
the TAG carries as a case-insensitive regular expression, `sgenre` means the
group of the Simplified View, the album is filed at; `track` keeps the albums
that hold a track whose title matches (a regular expression) and `trackN`
those that hold a track of the given number (a "1/20" tag is read as its
first number) — when such an album is opened, only the matching tracks are
shown). The 🎲 **Random
albums** link opens a limited draw of albums matching the choices — the
listbox beside it picks the count (5 / 10 / 20 / 50 / 100). The draw opens
as a level of its own and the walk carries on from there normally.

## The album rows

Both the browse page and **My Music → Simplified View** show the original genre
and the original album artist on a second line under the album title, with a
small drawn arrow between them. Hover an album cover on the page (or focus it
with the keyboard) for a compact **two-column card** of every original tag as
`NAME:value`, sorted alphabetically; the tree shows the same list as a flat
tooltip. Repeated tags are all kept — an album whose tracks carry different
`GENRE` spellings shows every one of them. `SV_GENRE`, `SV_ARTIST` and
`SV_MODE` are the **virtual** Simplified View values, not raw tags.

Every track row carries `Playing`, `#N` (its index in the queue) or `------`;
the `STATUS` of an album uses `Playing` > the index of the album's **first**
queued track > `------`. The page follows the queue live, the tree reads it when
the level is fetched. A fresh library scan fills the new `album_tags` store
with every raw genre spelling; until then a server upgraded from an earlier
release shows the one genre it already had.

## The line of the selections

The **last** line of every level of "My Music > Simplified View" reads
`<order> · <dates> · <quality> · <limit>` — the choices in use, straight from
the plugin's preferences. It opens the **browse page**, where the choices are
made: the **Order** selector sets the sorting of the artists and of the
albums, the separate **Group order** selector the order of the genre groups,
and pressing **OK** writes the preferences of the plugin.

**The choices are global**: they hold for the whole tree at once. The levels
carry no choice of their own — each one reads the stored preferences when it
is fetched, so a pick made on the page applies to every level entered
afterwards (the level the user came from included). The walk rows carry the
path only, and the URL of the header line carries no choice either — a
remembered link can never re-register an old value over the picks. One
exception no server can lift: the **Back** of the Material skin re-serves the
snapshot it kept of the previous level, without any request — that level keeps
showing the old list until it is entered again.

On top of the artist list of one genre the tree also offers a first row,
**Selected albums** — the albums of that whole genre under the readings of
the choices, whatever the artist.

**Experimental** (off by default, parameter page): with *Open the page beside
the list* ticked, the browse page opens in the Material skin as a **panel on
the right half of the window** instead of a fullscreen cover — a script on
the page docks the dialog beside the browse list; nothing is modified in the
skin, and the page falls back to the plain fullscreen window when the trick
cannot work.

## Choices per player, and the native menu (Squeezer)

Every choice of the walk (album order, group order, dates, quality, items per
list, random draw, Uncategorised, ignore accents, expression) belongs to the
**player that asks**: a player keeps its own values (client preferences), a
player with none uses the values of the server — the ones the settings page
edits, which are therefore *the defaults of the players*. With no player (the
CLI, a headless server, a page opened without a player) the server values are
read and written.

The native clients (Squeezer on Android, the jive players) show no HTML: the
line of the selections and the settings page do nothing there. Every level of
the tree therefore carries a row **⚙ Filters & sorting** just before the line
of the selections. It opens a native menu titled after the player:

- one row per choice list (order, group order, dates, quality, items per list,
  random draw): it opens the list of the values, the current one is marked ✓;
  a pick writes it for the player and goes straight back to the list, reloaded (so the Back button of Squeezer never shows a stale value)
- two switches (Uncategorised, ignore accents) that flip in place
- the expression, a text input, with a *Clear* row when one is set
- *Reset this player to the server defaults*, when the player has values of its own
- the first row (← title) goes back to the list, which reads the new choices

The menu travels in plain-ASCII parameters carried like `lm_path`:
`lm_menu` (`sel` | `sel.<group>`), `lm_set` (`<group>.<value>` | `reset`) and
`lm_expr_set` (the typed expression). The groups: `o` order, `g` group order,
`t` dates, `q` quality, `c` items per list, `r` random draw, `u` Uncategorised,
`f` ignore accents, `e` expression. From the CLI:

    librarymapper items 0 50 lm_menu:sel
    <playerid> librarymapper items 0 50 lm_menu:sel.q lm_set:q.hires

The settings page (scan, mapping upload, genre rules, icons) stays web-only:
open `http://<server>:9001/settings/` in the browser of the phone.
## The order of the artists

The artists are not sorted by the name as it is written: **a person is filed
under its last name, a group under the name it carries**.

    "Claude Nougaro"        -> "nougaro claude"   (a person)
    "Nougaro, Claude"       -> "nougaro"          (the inverted form)
    "Metallica"             -> "metallica"        (a group)
    "The Beatles"           -> "beatles"          (the leading article is skipped)
    "Miles Davis Quintet"   -> "miles davis"      (the ensemble word is skipped)
    "Peyceré Trío"          -> "peycere"          (trío is trio, accented or not)

The ensemble words are known in several languages — *trio* and its cousins in
English, French (quatuor, quintette), German (quartett), Spanish (duo, cuarteto),
Italian (quartetto), Dutch, the Scandinavian, Polish (kwartet), Czech, Hungarian,
Finnish, the Russian transliterations…, the orchestra words (orchestra, orchester,
orquesta, orkester…), the band, group and choir words (band, banda, conjunto,
zespół, yhtye, chor, koor…), *big band*, *symphony*, *philharmonic* and the guest
words (feat., and, with). They are matched with or without their accent, so
`Peyceré Trío` is filed under **P**, not under **T**. One exception: when the
word is the whole name — a group called just `Trio` (a French group) or `Duet`
(a French jazz duo) — nothing is skipped and the word is the key.

A name that is not written in Latin letters is transliterated (Cyrillic,
Hebrew, Japanese, Chinese), so that it is not left after the "z".

## The number of items that are shown

A level of the tree is cut at the limit of the plugin. The header of the tree
shows the limit that is in use, and it opens the browse page where **10**,
**100**, **1000** and **unlimited** (or whatever the parameter page allows)
can be changed. The choice holds for the whole drill-down, and the page opens
on the level the header line is on.

## The Uncategorised fallback

An album without any real genre tag is not lost: it gets a **derived group**
shown with a low-confidence marker `(?)` after its year — the group most of the
artist's other albums stand in, or, for a compilation (Various Artists), the
group of the words of its title. An album without any clue uses **Uncategorised**.
A **tagged** spelling that no rule recognizes uses the same fallback (marked
`(?)`). Approving the spelling on the parameter page moves its albums out at once.

Enable **Show the Uncategorised fallback** on the parameter page to display this
virtual genre — or, quicker, press the **Uncategorised** chip in the header box
of the browse page (beside **Help** and **Playlist**). Its ring fills when the
group is shown; the chip writes the same preference the parameter page and the
**My Music** tree read, so the browse page, the tree and the settings checkbox
stay in step. The group is a top-level entry: on a deeper level the chip keeps
its state, and the group appears when the list of the genres is opened again.

It covers both missing genre tags and unmapped genres, not only
compilations. Albums without an artist credit and genuine compilations (including
the usual `VA`, `Various`, and `Sampler` spellings) share the **Various Artists**
virtual artist.

The broad groups themselves come from keyword rules (*blues*, *blues rock*,
*chicago blues* all stand in **Blues**); the rules match whole words of the
normalized spelling (without accents, without punctuation). Your own rules
replace or extend the shipped ones on the parameter page, one line per group,
applied from top to bottom (the first line that matches wins):

    Blues : blues
    Musette : bal musette
    My Favorites :

A line **without words** is the *fallback*: it closes the order, and every
spelling no line matched lands in its group (by default **Uncategorised**). A
list with its own fallback line is applied alone; a list without one plays in
front of the shipped rules. The list is **downloadable**
(`plugins/LibraryMapper/genre-rules.txt`) and **uploadable** back, and it
travels in the JSON mapping file, so it carries over to another server.

## Notes

- The plugin is **query-only**: it never moves, renames or deletes any file on disk.
- It keeps a dedicated SQLite database (`Cache` directory, `librarymapper.db`).
- Mapping keys are raw text values (genre strings, artist names), so mappings
  survive library rescan and can be carried between servers (e.g. map albums on a
  download NAS, reuse on the main one).
- Physical reorganization into `Genre\Artist\Year\Album` folders is *out of scope* —
  the plugin only previews what that layout would look like.
