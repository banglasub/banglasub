# BanglaSubTitles

BanglaSubTitles is a client‑side web app for browsing and downloading Bangla subtitles for movies and TV series. It groups multiple subtitle files into a single title entry, supports fuzzy search with autocomplete, rich TMDB‑powered detail modals, weekly highlights, and direct subtitle downloads via a backend proxy.

---

## Features

- Intelligent **movie vs TV detection** from filenames (e.g. `S01E03`, `1x02`, `season`, `episode`, `web-series`).
- Smart **title cleaning** from messy release names (resolutions, codecs, encoders, tags, etc. are stripped).
- Grouping multiple files into a single **title card** by cleaned title, year, and type (movie/TV).
- **Fuzzy search** with instant results and autocomplete suggestions (powered by uFuzzy, client‑side only).
- Dual layout: **card grid** and **compact list** view, toggleable on the fly.
- “Latest Subtitles” / **Popular** section using the server’s ordering of files.
- **Weekly uploads slider** showing the latest 20 grouped titles with posters and quick info.
- Filters for **year**, **type** (movie/TV), and **genre** (using TMDB genres).
- Detail modal with:
  - Poster from TMDB (fallback to local `DEFAULT_NOT_FOUND` image).
  - Release date or first air date, rating, director or series creator, description, and top cast.
  - Genre pills, quality and multi‑part tags.
  - TV‑specific labels like `S01E03` or `S01E01-E03` for multi‑episode packs.
  - Download button (via backend proxy) and copyable direct download link (Clipboard API).
- Non‑blocking **TMDB genre prefetch**: metadata is fetched in the background, so first page loads fast.
- Fully responsive (desktop, tablet, mobile) with scrollable modal content on small screens.

> Note: The older light/dark theme toggle is not handled in `script.js`; theming can be managed via CSS and a separate script if desired.

---

## Tech Stack

- HTML5 + modern CSS (flexbox, grid, responsive layouts).
- Vanilla JavaScript (no frontend framework).
- [uFuzzy](https://github.com/leeoniya/uFuzzy) for client‑side fuzzy search over all titles and filenames.
- TMDB (The Movie Database) HTTP API for metadata, posters, cast, and genres.
- Clipboard API (`navigator.clipboard`) for copying direct download links.

---

## Backend Endpoints (Expected)

The frontend expects two HTTP endpoints:

- `GET /api/subtitles-list`  
  Returns JSON with:

  ```json
  {
    "files": [
      {
        "filename": "Movie.Title.2023.1080p.WEB-DL.Bangla.srt",
        "path": "/relative/or/absolute/path/to/file.srt",
        "year": 2023
      }
    ]
  }
  ```
  - `filename`: original filename used for cleaning/type/quality/year parsing.
  - `path`: value passed to the download proxy.
  - `year`: optional; numeric or numeric string.

- `GET /api/download?path=<encoded-path>`  
  Streams the subtitle file for download. The frontend uses this endpoint when the user clicks **Download** inside the detail modal.

You can implement these endpoints in any backend (Node, PHP, Laravel, Django, etc.) as long as the contract matches.

---

## How It Works

### 1. Data Loading and Grouping

On home page initialization (`initHomePage()`):

1. The app calls `GET /api/subtitles-list` with `Cache-Control: no-store` to avoid stale data.
2. Each item in `files` is transformed into an internal subtitle object:
   - `titleKey`: cleaned human‑friendly title from `filename` via `cleanMovieTitle` (removes extensions, quality tokens, codecs, encoders, etc.).
   - `type`: `'movie'` or `'tv'` inferred via `inferType`, based on tokens like `s01e`, `season`, `episode`, `web-series`.
   - `origName`: original `filename` for downloads/display.
   - `path`: raw backend `path` for `/api/download?path=...`.
   - `year`: parsed numeric year if available, otherwise empty string.
   - `timing`: optional sync info from patterns like `[00:02:13]` in the filename.
   - `seasonEpisode`: parsed `SxxExx` or `SxxExx-Exx` for TV releases (supports `S01E03`, `1x03`, multi‑episode packs).
   - `qualityTags`: quality/mode tags inferred from the filename, such as `4K`, `1080p`, `WEB-DL`, `BluRay`, `BRRip`, `Remux`, `Multi-part`.

3. `groupSubtitlesByTitleYear()` then:
   - Normalizes the cleaned title using `normalizeTitleKey` (lowercases and normalizes whitespace, less destructive than before to avoid merging unrelated titles like different “Twelve” variants).
   - Groups items by `normalizeTitleKey(titleKey) | year | type` into `groupedTitles`.
   - Sorts groups by their first appearance in the original `files` list.
   - Populates `latestGroups` with the first 50 groups.

This grouping ensures that multiple subtitle files for the same movie/series entry show up under one card with multiple variants in the detail modal.

---

### 2. Search and Suggestions

The app builds a searchable string array `haystack`:

- Each entry concatenates the group’s title, type, year, and all original filenames of its subtitles into a single string.

Then:

- Initializes a `uFuzzy` instance with tuned options (insertions, deletions, substitutions, transpositions allowed; custom word delimiters like spaces, underscores, dashes, `@`, parentheses).
- Maintains `idxToGroup` to map search results (indices) back to `groupedTitles`.

When the user types in the search box:

- Input is debounced (180 ms).!
- `searchGroups(query)` runs `uf.search(haystack, query)` to get best matches.
- Matching groups are:
  - Rendered in the **Search Results** grid/list (`resultsSection`).
  - Also used to build up to 8 clickable suggestions in `searchSuggestions` under the input.

Clicking a suggestion opens the detail modal directly for that title.

---

### 3. Card vs List View

Two views share the exact same data:

- **Card view** (`.card-grid`):
  - Poster on top (lazy‑loaded via TMDB), title and type badge, year, subtitle count, and up to 4 quality tags.
- **List view** (`.list-view`):
  - Thumbnail on the left, text info and tags to the right.

The toggle:

- Buttons `#cardViewBtn` and `#listViewBtn` manage which mode is **active**.
- `toggleView(isCard)`:
  - Switches container classes between `card-grid` and `list-view` for both **Latest** and **Search Results** containers.
  - Optionally re‑renders current content in the new layout (unless `skipRerender` is set on initial load).

---

### 4. Weekly Slider

The weekly (or “fresh uploads”) slider showcases recent titles:

- `computeWeeklyGroups()` picks the first 20 elements from `groupedTitles`.
- `renderWeeklySlider()` renders them as `.weekly-card` elements with:
  - Poster (from TMDB or fallback).\
  - Title, year, and a tag (first quality tag or type label, e.g. `TV` or `Movie`).
- Navigation:
  - `#weeklyPrev` and `#weeklyNext` buttons scroll by one card width at a time.
- Auto‑scroll:
  - `startWeeklyAutoScroll()` automatically scrolls forward every `WEEKLY_AUTO_INTERVAL` milliseconds (4.5 seconds), loops back at the end, and pauses on hover/touch.

---

### 5. Filters: Year, Type, Genre

The app supports three filter dimensions on both home and results views:

- **Year**:
  - Extracted from `groupedTitles` and populated into `#yearFilter` and `#yearFilterResults` as `<option>` values.
- **Type**:
  - UI side select for `'movie'` or `'tv'` (you define the options in HTML and `script.js` reads them).
- **Genre**:
  - Uses TMDB genres (Action, Comedy, Drama, etc.), provided by the `KNOWN_GENRES` constant and stored per group from TMDB metadata.

`applyYearTypeGenreFilter(list, yearValue, typeValue, genreValue)` filters any list of groups based on the selected values and the `group.genres` array.

---

### 6. Detail Modal

When the user clicks any card/list item:

1. `openDetailModal(group)`:
   - Locks background scroll by adding `modal-open` to `<html>` and `<body>`.
   - Sets title, year label, type badge (`Movie` / `TV Series`).
   - Sets poster to `DEFAULT_NOT_FOUND`, then asynchronously updates via `getPosterUrl`.
   - Renders aggregated quality tags for the whole group in `#detailQualityTags`.
   - Shows “Loading details…” placeholders for overview, release date, director/creator, rating, and cast.

2. `getMovieDetails(group.title, group.year, group.type)`:
   - Calls TMDB `search/movie` or `search/tv` and then `movie/{id}` or `tv/{id}` with `append_to_response=credits`.
   - For TV:
     - Prefers exact `name.toLowerCase()` match to avoid wrong series when titles like “Twelve” collide.
     - Then exact `original_name.toLowerCase()`.
     - Finally falls back to the first result.
   - Extracts overview, release/first air date, rating, director or first `created_by`, top 6 cast (name, character, profile image), and genres.
   - Populates `#detailDescription`, `#detailReleaseDate`, `#detailDirector`, `#detailRating`, `#detailCastGrid`, and `#detailGenres` accordingly.

3. Subtitle variants:
   - All `group.subs` are rendered as rows inside `#subtitleVariants`.
   - Each row shows:
     - Display name (clean title).
     - For TV: `Episode: SxxExx` or `SxxExx-Exx` and optional sync timing like `Sync: 00:02:13`.
     - Quality pills; falls back to a `General` pill if none.
     - **Download** button:
       - Calls `GET /api/download?path=<encoded sub.path>`.
       - Shows spinner while loading, checkmark on success, warning on error.
     - **Copy link** button:
       - Builds full URL: `window.location.origin + DOWNLOAD_PROXY + encodeURIComponent(sub.path)`.
       - Uses `navigator.clipboard.writeText` to copy.
       - Shows checkmark or warning based on success/failure.

4. Sidebar:
   - Shows last 5 `latestGroups` as small posters with title and year.
   - Clicking a sidebar item opens its detail modal.

5. Closing:
   - Click `#closeModal` button.
   - Click on backdrop area (`.modal-backdrop`).
   - Press `Escape` key.

The modal body uses CSS max‑height and `overflow-y: auto` (in your styles) so content remains scrollable on smaller screens.

---

## Performance Notes

- First page render is **not blocked** by TMDB extended metadata anymore:
  - `fetchSubtitles()` now:
    - Fetches subtitle list, builds groups, initializes search and filters.
    - Immediately calls `displayLatest(1)` to render the home content.
    - Starts `prefetchGenresForGroups()` in the background (without `await`) to fill `group.genres` for genre filtering.
- Poster loading is per card, using `getPosterUrl` with basic retries (full title and first 3 words, with/without year).

---

## Setup

1. **Place frontend files**

   - Put `index.html`, `style.css`, and `script.js` into your project (adjust paths as needed).
   - Ensure all IDs referenced in `script.js` exist in your HTML structure (containers, filters, modal elements, etc.).

2. **Include uFuzzy**

   Include the uFuzzy script in your HTML (CDN or bundled):

   ```html
   <script src="https://unpkg.com/@leeoniya/ufuzzy/dist/uFuzzy.iife.min.js"></script>
   ```

   `script.js` expects `uFuzzy` to be available globally.

3. **Implement backend endpoints**

   - `GET /api/subtitles-list` returning `files: [{ filename, path, year }]`.
   - `GET /api/download?path=<encoded-path>` streaming the file.

4. **Configure TMDB API key**

   In `script.js`, set:

   ```js
   const TMDB_API_KEY = 'YOUR_TMDB_API_KEY_HERE';
   ```

   For production, consider injecting this at build time instead of hard‑coding.

5. **Bootstrap the home page**

   In your layout script:

   ```js
   window.addEventListener('DOMContentLoaded', () => {
     if (window.initHomePage) {
       window.initHomePage();
     }
   });
   ```

   And in your HTML:

   ```html
   <body data-page="home">
   ```

   so `initHomePage` knows when to run.

---

## Customization

- **Type detection**: tweak `inferType(filename)` if you want different rules for TV vs movies.
- **Title cleaning**: adjust the big regex in `cleanMovieTitle` to handle your naming patterns (extra tags, languages, groups, etc.).
- **Quality tags**: extend the `map` in `extractQualityTags` for new formats like `HDR10`, `DolbyVision`, etc.
- **Grouping rules**: if you want stricter or looser grouping, modify `normalizeTitleKey` or the composite key in `groupSubtitlesByTitleYear` (e.g. include more dimensions like resolution or source).
- **Genres**: `KNOWN_GENRES` can be updated to customize visible genre options in the filters.

---

## License

license (MIT). You must also follow TMDB’s terms of use and attribution requirements for API usage and image assets.
