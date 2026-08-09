# Changelog

All notable changes to RyDF are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/); versions are `MAJOR.MINOR.PATCH`
matching `src-tauri/Cargo.toml` and `src-tauri/tauri.conf.json`.

The in-app updater reads this file — at each release tag — to show the user
exactly what changed between the version they have and the one on offer, so keep
the `## [x.y.z] — YYYY-MM-DD` heading format stable.

## [Unreleased]

## [0.3.1] — 2026-08-09

### Fixed
- **The print preview shows the page again.** On a large sheet the preview
  turned into a broken-image box with a “?”. The preview renders the page at
  its own fit-to-paper scale, which had nothing to do with the main view's
  zoom, and the renderer was rejecting it as a stale scale — the same reason
  applied to the Organize Pages thumbnails. Surfaces that render the document
  outside the main viewport are now exempt from that check.
- **A drawing bigger than the paper scales down to fit again.** Printing an
  oversized sheet — an ASME E drawing onto Tabloid, say — with the Vector
  method and “Fit to printable area” printed a cropped corner at full size
  instead of shrinking the sheet onto the page. The Vector method hands the PDF
  straight to the printer, which won't scale an outsized page down, so a sheet
  meaningfully larger than the paper now renders through the same path the
  Raster method uses, which places it precisely. Ordinary same-size prints are
  untouched and keep their original vector quality.

## [0.3.0] — 2026-07-31

### Added
- **The search box remembers what you've searched for.** Click into it and the
  queries you've already run on this drawing drop down underneath — five at a
  time, scroll for the rest. Click one, or walk the list with ↑/↓ and press
  Enter, to run it again. Only searches you actually *used* are kept, so the
  list holds whole queries rather than every half-typed fragment on the way to
  one, and re-running an old term floats it back to the top instead of
  appearing twice. Each tab keeps its own list for as long as that file is
  open: rotating, deleting or inserting pages and running OCR all leave it
  alone, and opening a different file into the tab starts a fresh one. While
  the list is down, ↑/↓ and Enter drive it; with it closed they step through
  search results exactly as before, and Esc closes the list before it clears
  the box.

## [0.2.3] — 2026-07-31

### Fixed
- **Clicking a search result now centres the view on it.** It used to take you
  to the right page and leave you hunting. The jump put the match a hair below
  the top edge — that gap was measured in page points, so the more you were
  zoomed in the smaller it got — and it never adjusted the sideways scroll at
  all, so on a wide sheet the match could sit off past the edge of the window
  with the view exactly where you left it. The match now lands in the middle of
  the window, both directions.
- **Pinch-to-zoom sharpens the moment you lift your fingers.** The re-sharpen
  added in 0.2.2 was reading the zoom level from just before the gesture's last
  frame, so it often decided nothing had changed and did nothing; the drawing
  then stayed soft for another fraction of a second until a follow-up pass
  caught it. The same stale reading also threw off the start of the *next*
  pinch by a small amount.

## [0.2.2] — 2026-07-29

### Added
- **Recognising text now runs in the background.** OCR used to hold a dialog
  open for the whole run, which on a scanned drawing set meant watching a
  progress bar for minutes. It now works away while you keep reading, scrolling
  and marking up; progress and a **Stop** appear in the status bar at the bottom
  of the window. Stopping finishes the page it is on and keeps everything
  recognised so far, and you get a summary at the end. One job runs at a time —
  two at once would only make both slower.

### Fixed
- **Pinch-to-zoom is much smoother, and reaches further.** Two separate problems.
  macOS hands the app the trackpad's magnification only about 40–60 times a
  second, in jumps of 6–8% — far coarser than the display refreshes — so the
  drawing sat still and then lurched. It now eases between those updates instead
  of waiting for them, so the motion is spread across every frame. Separately,
  the raw trackpad magnification is conservative: at 300% a full finger-spread
  barely moved you. The gesture now travels further the further in you already
  are, ramping up to about twice the reach at maximum zoom, while staying 1:1 at
  100% and below where precision matters more.
- **“Open With ▸ RyDF” always opens the file.** It sometimes just launched the
  app and sat there. A file handed over during a cold launch could arrive in the
  moment between RyDF checking for one and being ready to be told about one, and
  was then dropped.
- **Zooming in past a re-render threshold no longer leaves broken image boxes.**
  A whole screen of tiles could be requested a fraction before the renderer was
  told the new zoom level, and it rejected every one of them. A tile that fails
  outright now leaves its low-resolution preview showing rather than a broken
  image.
- `npm run tauri dev` works again for anyone building from source; it aborted
  before opening a window.

### Changed
- **Tabs are sized to the drawing's name.** They used to be locked between two
  fixed widths, so a short name sat in a half-empty tab while a real sheet name
  — *2222-0102 - Damrosch Park Renovation - Issue A* — was cut off at a boundary
  that had nothing to do with the name. Each tab now takes exactly the room its
  title needs, up to a generous limit so one very long name can't push the rest
  off screen. The full path is still there on hover.
- **The tab strip scrolls with arrow buttons instead of a scrollbar.** Chevrons
  appear at either end only once the tabs stop fitting, and each greys out when
  there's nothing further that way. **+** stays put no matter how many documents
  are open, and switching to a tab that's scrolled out of sight brings it back.
  There is no limit on how many documents you can have open — there never was;
  the old fixed widths simply made it look like there was.
- **Another step faster when moving around a drawing.** Compressing a tile no
  longer happens on the same thread that draws it, so those two now overlap
  instead of queueing: a fresh screenful of a vector drawing arrives about 1.4×
  quicker, a dense architectural sheet about 1.25×.
- **The sidebar preview cache follows the Render cache setting.** It was a fixed
  size, which quietly became a smaller cache in real terms when tiles grew in
  v0.2.1 — with nothing you could turn up. Raising **Preferences ▸ Render
  cache** now buys more page previews as well as more page tiles.
- **Zooming out restarts background pre-rendering.** v0.2.1 stopped a zoom *in*
  from sending the background pass off after work nobody needed; the side effect
  was that zooming back out left it preparing pages at a resolution higher than
  anyone wanted, with no way to recover. It now re-aims itself when you settle
  at a lower zoom.

## [0.2.1] — 2026-07-26

A follow-up to v0.2.0's performance work, from profiling what the renderer
actually spends its time on rather than what seemed likely.

### Changed
- **Drawings appear noticeably faster when you move around.** Profiling showed
  half of a page's render time went not to drawing the page but to *compressing*
  the result — so tiles now drop the alpha channel they never used (every tile
  sits on opaque white) and compress with a faster setting. A fresh screenful of
  a vector drawing renders **1.6× quicker**; a dense architectural sheet 1.3×.
  The trade is memory: tiles are roughly 2.4× larger, so the render cache holds
  proportionally fewer of them. Background pre-rendering now stops once the
  cache is full rather than endlessly re-rendering and discarding, so a document
  bigger than your cache setting degrades gracefully instead of pinning a CPU
  core — raise **Preferences ▸ Render cache** if you work with very large sets.
- **Opening a document shows the first page much sooner.** RyDF used to hold the
  “Opening…” screen until five pages *and* five sidebar previews had finished —
  four of them pages you can't see yet. It now waits for the one page you're
  about to look at: **280 ms → 148 ms** on a 120-page drawing set, and
  **225 ms → 61 ms** on a 1,500-page document. Pages further down still show
  their low-resolution preview the moment you scroll to them.
- **Zooming out no longer sends the background renderer off on a wild goose
  chase.** It had been pre-rendering at whatever zoom the document opened at, so
  after you zoomed out everything it prepared was at the wrong size and thrown
  away. It now follows the zoom you're actually at — but never *above* the zoom
  the document opened at, because the number of tiles in a page grows with the
  square of the zoom, and speculatively pre-rendering a whole drawing set at 4×
  would cost more than it could ever save. What you're looking at while zoomed
  in is handled by the foreground renderer, which always takes priority.

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
