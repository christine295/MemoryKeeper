# MemoryKeeper — Project Guide for Claude

## What this is
A multi-page family photo review and tagging app. Served via GitHub Pages at `https://christine295.github.io/MemoryKeeper/`. Source at `https://github.com/christine295/MemoryKeeper.git`.

## Files
- `index.html` — sign-in page (Supabase auth)
- `app.html` — the entire main app (CSS + JS all inline, one large file)
- `share.html` — family contributor view (opened via share link, no login required)

## Key technical facts
- `app.html` uses `<script type="module">` — any function called from an inline `onclick` MUST be assigned to `window.functionName`
- File System Access API (`showDirectoryPicker`) — Chrome/Edge only; sets `dirHandle`; used for reading photos and moving files to subfolders
- On browsers without File System Access API, falls back to `<input type="file">` (no move support)
- Supabase project ID: `jflruwwmrhwgvfzfjtqf` — credentials are hardcoded in both `app.html` and `share.html`
- Metadata is stored both in Supabase (for sharing/contributions) and locally in `mk-metadata.json` in the photos folder
- GitHub Pages CDN caches aggressively — always bump the version number (`vX.Y` in the header) so the user can confirm the right build loaded. Hard refresh (Ctrl+Shift+R) clears it.

## Architecture — app.html
- **State**: `state` object holds `mode`, `view`, `queue`, `queueIdx`, `currentId`, `filter`
- **Modes**: `random-unreviewed`, `all-reviewed`, `favorites`, `best-shots`, `hidden`, `by-person`, `by-tag`, `by-holiday`, `by-year`, `duplicates`
- **Views**: `single` (one photo + form) or `gallery` (grid)
- **`qualifies(id)`** — filters which photos belong in current queue; `all-reviewed` excludes `hidden` and `unreviewed`
- **`advance()`** — called after Save & Next; filters queue then explicitly steps past current photo if it still qualifies (fix for photos that stay in queue in non-random modes)
- **`render()`** — rebuilds `main-content` innerHTML; calls `renderCard(id)`, `renderGallery()`, or `renderEmpty()`
- **`renderCard(id)`** — returns HTML string; uses template literals; move-to-subfolder row only shown when `dirHandle` is set
- **`metaStore`** — in-memory object keyed by photo ID; persisted via `persistMeta()`
- **`photoLibrary`** — array of photo objects with `id`, `fileName`, `filePath`, `_entry`, `_url`

## Architecture — share.html
- No login required — opened via share token in URL
- Contributor can browse photos, prefill form fields come from photo metadata stored in Supabase
- Name/email remembered across photos in session (`cvContributorName`, `cvContributorEmail`)
- Prev/Next navigation via overlay arrows on photo; left/right arrow keys also work
- Submitted photos get a green "✓ Done" badge on the gallery grid

## Supabase tables
- `mk_archives` — top-level family photo archives (one per family grouping)
- `mk_collections` — shared photo sets; `archive_id` + `title` (title = folder name at share time)
- `mk_photos` — individual photos within a collection; metadata columns mirror app metadata fields
- `mk_contributions` — family member text submissions (people, date, location, memory note, tags)
- `mk_uploads` — family member photo uploads

## Known caveats
- Contributions badge + review modal are scoped to `archive_id` AND `title = folderName` — this means if a user renamed a collection title when sharing, that collection won't appear in the folder's contributions view
- File System Access API move-to-subfolder only works in Chrome/Edge; Safari/Firefox users see the input field but the move button won't work
- `buildFreshQueueExposed` is referenced in `onModeChange` filter input — verify it is exposed to `window` if editing that area

## Current version
v2.6 (as of 2026-05-13)

## Workflow
1. Edit `app.html` or `share.html`
2. Bump version string (e.g. `v2.6` → `v2.7`) in the header span
3. `git add . && git commit -m "vX.Y — description" && git push origin main`
4. User hard-refreshes (Ctrl+Shift+R) and confirms version number in header
