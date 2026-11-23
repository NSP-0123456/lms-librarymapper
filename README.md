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
   SQLite database and proposes the genre groups.

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
(Material included) with its own icon, and the virtual tree opens from there.
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
albums, the order of the groups, the number of items, a filter on the name
of the artist (transliterated when **Translit** is ticked) and a free
**Expression** (`year:1980..1990 duration:>45:00 …`). The 🎲 **Random
albums** link opens a limited draw of albums matching the choices — the
listbox beside it picks the count (5 / 10 / 20 / 50 / 100). The draw opens
as a level of its own and the walk carries on from there normally.

## The choices of the tree

The first line of every level of "My Music > Simplified View" reads
`Choices · <order> · <dates> · <quality> · <limit>`. It does **not** open the
web page: it opens a small tree of the **choices themselves**, plain list
items that every skin renders inside its own list (Material included, and
Squeezer on Android, which cannot show a web page at all):

- **Ordering / Date range / Quality / Count per level** — the values of the
  group, with a tick on the one in use; picking one writes the preference the
  parameter page writes, and the list is shown again with the tick moved.
- **Expression** — a text entry in the skin's own input window; the typed
  expression is stored and applied to the walk, with a *Clear expression* row
  while one stands.
- **Web page: filters and help** — the old browse page, at the end.

**The choices are global**: they hold for the whole tree at once. The levels
carry no choice of their own — each one reads the stored preferences when it
is shown, so a pick made anywhere applies to the level the user came from and
to every level shown afterwards (the browse page reads the same preferences).

**Experimental** (off by default, parameter page): with *Open the page beside
the list* ticked, the browse page opens in the Material skin as a **panel on
the right half of the window** instead of a fullscreen cover — a script on
the page docks the dialog beside the browse list; nothing is modified in the
skin, and the page falls back to the plain fullscreen window when the trick
cannot work.

## The order of the artists

The artists are not sorted by the name as it is written: **a person is filed
under its last name, a group under the name it carries**.

    "Claude Nougaro"        -> "nougaro claude"   (a person)
    "Nougaro, Claude"       -> "nougaro"          (the inverted form)
    "Metallica"             -> "metallica"        (a group)
    "The Beatles"           -> "beatles"          (the leading article is skipped)
    "Miles Davis Quintet"   -> "miles davis"      (the ensemble word is skipped)

A name that is not written in Latin letters is transliterated (Cyrillic,
Hebrew, Japanese, Chinese), so that it is not left after the "z".

## The number of items that are shown

A level of the tree is cut at the limit of the plugin. The header of the tree
shows the limit that is in use, and it opens the browse page where **10**,
**100**, **1000** and **unlimited** (or whatever the parameter page allows)
can be changed. The choice holds for the whole drill-down, and the page opens
on the level the header line is on.

## Notes

- The plugin is **query-only**: it never moves, renames or deletes any file on disk.
- It keeps a dedicated SQLite database (`Cache` directory, `librarymapper.db`).
- Mapping keys are raw text values (genre strings, artist names), so mappings
  survive library rescan and can be carried between servers (e.g. map albums on a
  download NAS, reuse on the main one).
- Physical reorganization into `Genre\Artist\Year\Album` folders is *out of scope* —
  the plugin only previews what that layout would look like.
