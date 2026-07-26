# Changelog

All notable changes to RyDF are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/); versions are `MAJOR.MINOR.PATCH`
matching `src-tauri/Cargo.toml` and `src-tauri/tauri.conf.json`.

The in-app updater reads this file — at each release tag — to show the user
exactly what changed between the version they have and the one on offer, so keep
the `## [x.y.z] — YYYY-MM-DD` heading format stable.

## [Unreleased]

## [0.1.4] — 2026-07-26

### Changed
- When an update is ready and you have unsaved markups, RyDF now **asks** whether
  to save them before it quits to finish updating, instead of saving for you
  automatically. (A session with nothing unsaved still goes straight to the
  installer.)

## [0.1.3] — 2026-07-26

### Changed
- When an update finishes downloading and its installer disk image opens, RyDF
  now saves any unsaved markups and **quits automatically**, so you can drag the
  new version straight into your Applications folder over the old one without
  quitting the app yourself first.

## [0.1.2] — 2026-07-26

### Fixed
- Pinch-to-zoom — and ⌘/Ctrl-scroll zoom — works again. A regression in the
  recent "keep every tab's viewer mounted" change left the zoom-gesture
  listeners unattached whenever a viewer first mounted while its tab wasn't the
  active one; they now attach as soon as the tab becomes active.

### Added
- Right-clicking an entry in the **Outline** panel now offers the same page
  operations as the **Pages** panel — rotate, extract, insert PDF, insert blank
  pages, copy & paste measurement scale, and delete — acting on that entry's
  page.

### Changed
- Sidebar page previews are now warmed in the background right after a document
  opens (into their dedicated thumbnail cache), so scrolling the **Pages** list
  stays smooth instead of rendering each preview on demand as it scrolls into
  view.

## [0.1.1] — 2026-07-25

### Fixed
- Sidebar page previews no longer re-render when you zoom the main view (or
  during the background render pass). Thumbnails now use a dedicated cache, so
  they draw once and stay — the visible page renders first, then its previews.
- The Preferences window now scrolls when its content is tall, so the
  **Software updates** section is no longer clipped.

### Changed
- Search runs much faster on large documents: each page's text is indexed once
  in the background after a document opens, so search skips the expensive text
  scan on pages that can't contain the query (OCR'd text is included). The index
  is revision-tagged, so any page edit or OCR invalidates it automatically.
- The search-result highlight now defaults to **2 pt** on new installs.

## [0.1.0] — 2026-07-21

### Added
- Fast, tiled, virtualized PDF viewer for very large documents, with a custom
  priority-scheduled render cache and background prewarm.
- Full markup toolset (Autodesk Design Review-style): shapes, clouds, text,
  callouts, notes, dimensions, ink, highlighter, stamps, and a signature tool.
- Measurement calibration with per-page scale, a quick-measure tape, and an
  optional snap to PDF geometry.
- Organize Pages grid view (rotate, delete, extract, insert, reorder).
- Custom print pipeline and print dialog.
- Native PDF annotation export on "Save a Copy" for cross-viewer compatibility,
  plus the exact markup JSON embedded as an attachment so an exported copy stays
  losslessly re-editable in RyDF.
- Auto-update: checks GitHub releases (with a launch/interval/manual policy),
  shows what's new, and downloads + opens the installer.
