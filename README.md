# LibraryMapper — an Lyrion (LMS) plugin

Map your large music library *inside* Lyrion Music Server, **without touching files**:

- Browse a virtual `Genre \ Artist \ Year - Album` layout as an app with categories
- **Temporal filter**: albums added on the server in the last day / week / month / year
- **Quality filter**: hi-res / CD / lossy
- Simplify hundreds of raw genre tags into 10–20 broad groups (semi-manual curation, then automatic application)
- Consolidate album-artist spelling variants
- Parameter page with an **initialization phase**, mapping **validation**, and **download/upload** of mapping files (JSON, `librarymapper-mapping` v1)
- Compilations stay *Various Artists*; genre-less albums get derived genre + low-confidence flags

## Install (any LMS/Lyrion ≥ 7.9)

1. Open your server's web UI → **Settings → Extensions**
2. At the bottom set *Additional Repositories* to:
   `https://raw.githubusercontent.com/NSP-0123456/lms-librarymapper/main/extensions.xml`
   (restart, apply)
3. In the plugin list, install **LibraryMapper**.
4. After the restart, open **Settings → Plugins → LibraryMapper** and click *Initialize* —
   this scans the Lyrion library into the plugin's dedicated SQLite database and proposes genre groups.
5. The browse app is under **Home → LibraryMapper**.

## Pages

| Page | Purpose |
|---|---|
| Settings → LibraryMapper | initialization, validation report, repropose, download/upload mapping file |
| `plugins/LibraryMapper/index.html` | virtual browse app (Genre \ Artist \ Year - Album + filters) |
| `plugins/LibraryMapper/download.json` | download mapping JSON |

## Notes

- The plugin is **query-only**: it never moves, renames or deletes any file on disk.
- It keeps a dedicated SQLite database (`Cache` directory, `librarymapper.db`).
- Mapping keys are raw text values (genre strings, artist names), so mappings survive library rescan and can be carried between servers (e.g. map albums on a download NAS, reuse on the main one).
- Physical reorganization into `Genre\Artist\Year\Album` folders is *out of scope* — the plugin only previews what that layout would look like.
