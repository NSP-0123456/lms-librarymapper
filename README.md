# LibraryMapper — an Lyrion (LMS) plugin

Map your large music library *inside* Lyrion Music Server, **without touching files**:

- Browse a virtual `Genre \ Artist \ Year - Album` layout as an app with categories
- **Temporal filter**: albums added on the server in the last day / week / month / year
- **Quality filter**: hi-res / CD / lossy
- Simplify hundreds of raw genre tags into 10–20 broad groups (semi-manual curation, then automatic application)
- Consolidate album-artist spelling variants
- Parameter page with an **initialization phase**, mapping **validation**, and **download/upload** of mapping files (JSON, `librarymapper-mapping` v1)
- Compilations stay *Various Artists*; genre-less albums get derived genre + low-confidence flags

## Install (any LMS/Lyrion ≥ 7.9) — through the plugin updater

The plugin is published as a plugin repository, so Lyrion installs it by itself
(download → SHA1 check → replace → restart):

1. Open your server's web UI → **Settings → Advanced → Plugins**
2. At the bottom, in *Plugin repositories*, add:
   `https://raw.githubusercontent.com/NSP-0123456/lms-librarymapper/main/extensions.xml`
   then **Apply**
3. Tick **LibraryMapper** in the list and **Apply** again (the page shows it under
   the repository you just added). Lyrion downloads the release, checks its SHA1
   and asks for a restart.
4. Restart the server (the button is offered at the top of the same page).
5. After the restart, open **Settings → Advanced → Plugins → LibraryMapper** and
   click *Initialize* — this scans the Lyrion library into the plugin's dedicated
   SQLite database and proposes the genre groups.

If the plugin is already installed in an older version, it appears in the
*Updates* section of the same page: tick it there and **Apply**.

### After the restart

- **My Music → Simplified View** — the native browse node, in every skin
  (Material included), with the same drill-down as the application page.
- **Settings → Advanced → Plugins → LibraryMapper** — the parameter page:
  initialization, validation report, repropose, trusted artists,
  download/upload of the mapping file.
- `/plugins/LibraryMapper/index.html` — the browse application itself
  (`Genre \ Artist \ Year - Album` + the two filters).

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

## Notes

- The plugin is **query-only**: it never moves, renames or deletes any file on disk.
- It keeps a dedicated SQLite database (`Cache` directory, `librarymapper.db`).
- Mapping keys are raw text values (genre strings, artist names), so mappings
  survive library rescan and can be carried between servers (e.g. map albums on a
  download NAS, reuse on the main one).
- Physical reorganization into `Genre\Artist\Year\Album` folders is *out of scope* —
  the plugin only previews what that layout would look like.
