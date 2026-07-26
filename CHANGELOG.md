# Changelog

All notable changes to RyDF are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/); versions are `MAJOR.MINOR.PATCH`
matching `src-tauri/Cargo.toml` and `src-tauri/tauri.conf.json`.

The in-app updater reads this file — at each release tag — to show the user
exactly what changed between the version they have and the one on offer, so keep
the `## [x.y.z] — YYYY-MM-DD` heading format stable.

## [Unreleased]

## [0.2.0] — 2026-07-26

A performance release: moving around a large drawing set should now show you
what you're looking at first, and fill in the rest outward from there. Plus a
batch of markup fixes and the text/colour controls that were missing.

### Changed
- **Rendering now works outward from where you are.** The old background pass
  rendered page 1, then page 2, then page 3 — however far away from you that
  was — and every one of those renders could sit in front of the page you were
  actually looking at. Now:
  - the tile under your viewport is always rendered next, ahead of anything
    queued for a page you've scrolled past;
  - the background pass starts at *your* page and walks outward a page at a
    time (yours, one back, one forward, two back, two forward…), so the pages
    you're about to reach are ready before you get there;
  - it re-reads where you are between each step, so jumping to a distant page
    immediately re-aims the whole background pass instead of finishing a walk
    across the document you've left.
- **Zoomed-in panning no longer shows blank space.** Every page now draws its
  low-resolution preview underneath the sharp tiles, so panning into an area
  that hasn't rendered yet shows a blurry version of the right drawing instead
  of empty white, and it sharpens in place.
- Background rendering is broken into single tiles rather than whole pages, so
  it can never hold the renderer for seconds at a time while you're waiting on
  the page in front of you.
- Pages are kept open between renders instead of being re-opened from the file
  for every tile — a large part of the cost of the first few tiles on a heavy
  CAD sheet.
- **Opening a document is markedly quicker.** The viewer used to wait out a
  debounce before it would render anything at the starting zoom; the first
  scale now goes straight through.
- Scrolling *toward* a page now renders the edge you're approaching first,
  rather than nothing at all until the page is partly on screen.

### Added
- **Text alignment** — left, centre and right, for callouts and text boxes. In
  the properties bar whenever one of those is selected. (Stamps aren't included:
  their wording is positioned by dragging it anywhere inside the stamp, which is
  a freer control than alignment.)
- **Text colour is now separate from line colour.** Markups get their own text
  colour control with a link toggle: linked (the default, and what every
  existing markup stays as) means text follows the line colour exactly as
  before; unlink it to set them independently.
- **Set scale ▸ From ratio** takes the drawing scale as two fields — the paper
  length and the real-world length it represents — instead of one string you
  had to punctuate correctly.
- **Quick measure results stay on screen** until you switch tools or start
  another measurement, labelled *temporary dimension* so it's obvious they
  aren't part of the document.
- **Snapping now applies to the Dimension and Quick measure tools**, not just
  the calibration drag — so measurements land on the geometry, not near it.
- **RyDF asks before discarding unsaved markups when you quit.** Quitting used
  to lose everything since the last autosave, which at the default five-minute
  interval could be a lot; you now get Cancel / Don't Save / Save & Quit. Both
  ways out are covered — the window's close button and ⌘Q — and if a save
  fails, RyDF stays open and tells you instead of quitting anyway.
- **Tool styles are remembered between launches.** The colour, line width,
  opacity and font size you last set for each tool are restored on the next
  start instead of resetting to the factory defaults.
- **Signatures can optionally be remembered.** The signature dialog has a
  *Remember on this Mac* tickbox — off unless you turn it on, and unticking it
  erases the stored copy immediately. Left off, a signature lasts only for the
  session, as before.

### Fixed
- **Right-clicking a markup no longer offers "Reload", which restarted the app
  and lost unsaved work.** The webview's built-in context menu is suppressed
  everywhere except in text fields, where it's still needed for cut/copy/paste.
- **Pinch-to-zoom responds properly again.** It was advancing roughly 1% per
  gesture: each event in a pinch was computed against the last *committed*
  zoom rather than the one the gesture had already built up, so all but the
  final event of every burst were thrown away.
- **Notes can be edited after they're placed.** The note popup opted out of
  pointer events on its SVG container, and WebKit stops hit-testing at that
  point — so clicks never reached the text area inside it.
- **Set scale ▸ From ratio accepts the marks macOS actually types.** Entering
  `1/8″=1'` failed because macOS substitutes a prime (`″`) for the straight
  inch mark and the ratio parser only accepted straight quotes.
- The background render pass no longer reports itself complete the moment it
  starts — the progress readout counted every already-warm page again on every
  step, so it hit 100% within a second of opening and stayed there.
- The background pass now gives up instead of looping forever on a document
  whose pages don't fit the cache budget. It had no way to tell "rendered" from
  "rendered, then immediately evicted", so on a big enough sheet set it would
  re-render the same tile for the rest of the session.

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
