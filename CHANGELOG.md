# Changelog

All notable changes to RyDF are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/); versions are `MAJOR.MINOR.PATCH`
matching `src-tauri/Cargo.toml` and `src-tauri/tauri.conf.json`.

The in-app updater reads this file — at each release tag — to show the user
exactly what changed between the version they have and the one on offer, so keep
the `## [x.y.z] — YYYY-MM-DD` heading format stable.

## [Unreleased]

## [0.10.2] — 2026-09-08

### Changed
- **Updating on Windows no longer walks you through an uninstall.** The
  installer was being run with no arguments, so it showed its full wizard —
  and because an older RyDF was already there, that wizard's job was to
  uninstall the old version and reinstall the new one, for what should be a
  routine update. It now runs the way Tauri's own updater runs it: passive, so
  there is a progress bar and nothing to click; flagged as an upgrade, so your
  settings and shortcuts are left alone; and set to relaunch, so RyDF reopens
  itself on the new version instead of simply vanishing. macOS is unchanged —
  a .dmg has no equivalent, so it still opens the disk image and quits.

  Note that updating *to* this version still uses the old behaviour, because
  the updater doing the work is the one already installed: this one update will
  still open the installer's wizard and offer to remove the existing version.
  That is expected — letting it run through installs 0.10.2 normally, and every
  update after it is the quiet kind.

## [0.10.1] — 2026-09-07

### Fixed
- **Opening a PDF while RyDF is running now adds a tab instead of starting a
  second copy of the app** (Windows). A file association there launches a whole
  new process with the path in argv, where macOS messages the app that is
  already open — so double-clicking a second drawing gave you two RyDFs. The
  duplicate window was the visible half of the problem; the real one was that
  both copies then shared `preferences.json` and the crash-recovery directory
  with nothing arbitrating between them, so whichever wrote last won and the
  other's state was silently gone. The second process now hands its arguments
  to the first and exits, and the first raises its window and opens the file —
  the same path a Finder double-click has always taken on macOS. Launching RyDF
  from the Start menu while it is already running now raises the window you
  have rather than opening another one. Selecting several PDFs and opening them
  together works the same way: Windows starts one process per file, and each
  hands its path over in turn, so they arrive as tabs in one window rather than
  as separate copies of the app.

## [0.10.0] — 2026-09-07

### Added
- **RyDF runs on Windows.** Every release now ships a Windows installer beside
  the macOS one. Printing goes through a printer device context, so a Windows
  print is real vector output at driver resolution rather than a rasterised
  fallback; and text recognition uses Windows' own OCR (it needs a recognition
  language installed under Settings → Time & language). Opening a file by
  double-clicking it in Explorer works — Windows passes the path differently
  from macOS and nothing had read it.
- **RyDF can offer to become your default PDF viewer**, once on launch and any
  time afterwards from Preferences. The launch offer takes **Yes**, **No** or
  **Ask Me Later**; "No" means never again. On macOS "Yes" simply does it. On
  Windows no application is allowed to claim a file type for itself, so "Yes"
  opens the Settings page where you can — and the wording says so rather than
  claiming a change that did not happen.

### Fixed
- **A pinch-zoom no longer flashes a grid of broken-image "?" boxes over the
  page.** The worker rejects any tile whose scale no longer matches the
  viewport hint, and a pinch crosses scale rungs constantly; each rejection is
  an HTTP 500, which the webview paints as its broken-image glyph and border —
  laid over the perfectly good coarse underlay and previous-scale tiles
  beneath. Tiles are now invisible until they have actually decoded, so a
  rejected one paints nothing at all while the retry that recovers it runs.
- **GIF and BMP now open on every platform**, not just macOS. The same build
  used to accept a .bmp on one platform and refuse it on another.
- **A missing PDFium no longer leaves a running app with a dead engine.** The
  failure killed only the engine thread, so the window opened and every
  document died silently against it.
- **The updater now downloads the right file, to the right place, on Windows.**
  It saved the NSIS installer with a `.dmg` extension into `%TEMP%` — a file
  Windows has no handler for, in a folder the dialog did not name — because the
  destination path was written for macOS only. It also told Windows users to
  drag RyDF into their Applications folder.
- **Windows print dialog preselects your actual default printer** instead of
  whatever the spooler happened to list first, which was typically Fax or
  Microsoft Print to PDF.

### Known limitations on Windows
- The installer is **not code-signed**, so SmartScreen shows "Windows protected
  your PC" on first run — choose **More info → Run anyway**. Signing needs an
  Authenticode certificate.
- Printing does not pass **paper size** to the driver (the sheet comes from its
  own defaults, and the Paper size panel offers a standard set rather than your
  printer's list), never offers **Two-sided**, and applies **Reverse order**
  only when saving to PDF. **Greyscale** works in Raster mode but not Vector.
  Page range, copies, collate, orientation and scale all work. The macOS path
  passes everything through CUPS.
- **HEIC** needs the free **HEIF Image Extensions** from the Microsoft Store.
  Bundling a decoder was tried and reverted: Microsoft ships that codec only
  through the Store with no redistributable, and every pure-Rust HEIC decoder
  published today is AGPL-3.0, which a signed closed-source binary cannot
  satisfy. Every other supported image format works with nothing extra.
- Opening a second PDF while RyDF is running starts a second copy of the app
  rather than a new tab. *(Fixed in 0.10.1.)*

## [0.9.0] — 2026-09-05

### Added
- **Pictures open like documents.** JPEG, PNG, TIFF, WebP and SVG open on every
  platform; HEIC, GIF and BMP open on macOS. A picture becomes a one-page PDF
  internally, so tiles, markup, measuring, search, OCR, crop and printing all
  work on it with no second code path. A JPEG's bytes go in **verbatim** — the
  wrapper is 1.01x the photo, where re-encoding it would be 7.5x.
- **SVG opens as true vector**, not a raster. It is converted to PDF page
  content, so the drawing stays sharp at any zoom and its text stays selectable
  and searchable. `.svgz` works too.
- **Saving a picture asks what you meant.** Convert to a PDF and carry on there
  with your markups still editable, or render them into a new picture. Neither
  branch touches the original file.
- **Export any page as an image** — JPEG, PNG, TIFF or WebP. TIFF is written
  Deflate-compressed (an A1 sheet at 300dpi is 64.5MB, against 209MB
  uncompressed) and records its resolution, which matters for print. WebP is
  lossless.
- **Export any page as SVG.** Vector where that is provably faithful, and where
  it is not — gradients, blend modes, soft masks — that part is rendered and
  embedded at exactly its own place in the stacking order, with the export
  reporting which happened. A test CAD sheet exports as 644 objects, all vector,
  86KB, curves preserved as true Béziers. Markups become real SVG, and links
  become real `<a href>` — which no other export in RyDF preserves. There is a
  strict mode that refuses rather than falling back, for when you need a
  guarantee of pure vector.
- **Crop**, for PDF pages and pictures alike, through one implementation.
  Non-destructive: it moves a boundary rather than discarding anything, reset
  restores the page exactly, and undo covers it.
- **Transparent pictures show their transparency**, over a checkerboard rather
  than composited onto white — which used to make white artwork invisible. The
  backdrop colour, pattern and check size are configurable in Preferences.

### Fixed
- **Saving a marked-up document could destroy a non-PDF file.** The guard that
  refuses to write PDF bytes over something that is not a PDF sat on only one of
  the two save paths, and not the one a document with markups takes — so RyDF
  refused to overwrite your photograph only while there was nothing to write,
  and did it as soon as there was. Save & Quit hit the same path for any open
  picture. Exporting SVG could destroy a PDF the same way, and now cannot.
- **Cropping is on the undo timeline.** It was not, so Cmd+Z undid the step
  before it while the page stayed cropped, and markups were left displaced.
- **Reset crop put markups back in the wrong place**, measuring from the page
  box's bottom while display space is anchored to its top — and on the wrong
  axis for a rotated page.
- **"Native" image export was 72dpi**, so a 300dpi scan was exported at 24% of
  its pixels, through both saving and exporting.
- **Printing a cropped page printed what you had cropped away**, because the
  vector print path removed the crop box.
- **Photo orientation was ignored** for every format the macOS backend reads, so
  a portrait iPhone photo opened on its side. Measured: `sips` neither reports
  nor applies EXIF orientation, contrary to the assumption in the code.
- **A JPEG's resolution is read from EXIF** when it carries no JFIF header, so a
  scan no longer opens four times its true size.
- **The comment popup** no longer grows as you zoom out, run off the screen, or
  stay on screen after the note is deleted — three symptoms of it being an SVG
  `foreignObject` counter-scaled by the zoom.
- Crop no longer takes the **R** shortcut from Rectangle, draws a preview while
  you drag, and ignores a stray click instead of showing a raw error.

### Changed
- **The OCR dialog asks how hard to work, instead of asking about rotation.**
  Two knobs that traded the same thing — how finely a big sheet is cut up, and
  whether it is read turned each way — are now one choice. **Most accurate**
  (the default) reads large sheets in finer pieces and looks at each one every
  way round; **Fast** does one upright pass. On a test A1 scan carrying 155
  labels, accurate found 153 and fast found 77, at about four times the time.
  The **Read rotated text** checkbox is gone, since it was half of this choice.
- **Most accurate reads more of a drawing than any previous version.** The
  finer tiling is new: what shipped through 0.8.2 found 129 of those same 155
  labels, and 117 of the 140 set at 4pt, against 139 now.

## [0.8.2] — 2026-08-31

### Fixed
- **Opening a PDF from Finder brings RyDF back from the Dock.** Double-clicking
  a file while the app was minimized loaded the document into a window that
  stayed minimized, so the double-click looked like it had done nothing. The
  window now comes forward.

## [0.8.1] — 2026-08-31

### Added
- **Search says when a result can't be seen.** Some PDFs carry text nothing
  paints — a page whose layout was replaced by a flattened image, with the
  original text left underneath. Search finds those words, because they really
  are in the file, and used to put a highlight over whatever artwork covers
  them, which reads as though the highlight were pointing at that artwork.
  Results like this now carry a **Not visible** tag, and their highlight on the
  page is drawn as a dashed outline rather than a solid block. Nothing changes
  for an ordinary document: across two very different test files and 430
  matches, not one was tagged.

## [0.8.0] — 2026-08-25

### Added
- **OCR reads rotated text.** Title-block labels and vertical dimension strings
  were being missed almost every time — the recognizer only reads text running
  left-to-right in the image it is given, and nothing was turning the page
  round. RyDF now reads each page all three ways. On a test sheet carrying six
  vertical labels, it found one of them before and all six now. There's a
  **Read rotated text** checkbox in the OCR dialog, on by default; turning it
  off is roughly twice as fast and right for a page of ordinary prose.
- **Clear marks from selected text.** Select text that already carries a
  highlight, underline or strikeout and the bar offers a **Clear** button that
  removes them, so changing your mind no longer means hunting for the mark with
  the select tool and deleting it by hand. It appears only when there is
  something to remove, the text stays selected so you can mark it a different
  way straight away, and it leaves anything drawn over the top — notes, clouds,
  arrows — alone.

### Changed
- **OCR reads large sheets in tiles, and finds the fine print.** Recognition
  works at a fixed internal resolution, so on a big drawing the small type came
  out below what it could read at all — and rendering the page at a higher
  resolution did not help, because the whole sheet was still being scaled back
  down. On 4-5pt dimension text on an A1 sheet, RyDF used to recover about one
  label in twenty; it now recovers nineteen. Letter and A4 pages are read whole
  as before and are no slower.
- **OCR no longer locks the window up while it works.** It used to recognize a
  whole page as one indivisible piece of work, which on a drawing was several
  seconds during which nothing else could render and Stop did nothing. It now
  works in small pieces, so the page keeps drawing and Stop takes effect in a
  fraction of a second.

### Fixed
- **The Highlight / Underline / Strikeout / Copy bar appears in full.** Selecting
  text at any zoom above 100% showed a clipped sliver of it — at 274% zoom, just
  the word "Highl" — and its buttons would not have worked once visible.
- **OCR text sits at the angle the text does.** A recognized line was previously
  laid down flat across its own bounding box, so on the rare occasions a rotated
  label was found, selecting it highlighted the wrong part of the page.
- **A failure to start OCR is reported instead of swallowed.** On a build
  without text recognition, RyDF used to work through every page rasterizing it
  at 200 DPI on the way to failing on each one, and say nothing.

## [0.7.1] — 2026-08-24

### Added
- **A print queue in the toolbar.** A job RyDF sends goes into the same queue as
  any other app's, but macOS only raises its printer window for jobs sent the
  Cocoa way — so a print from RyDF had no window attached to it, and once the
  progress bar said "sent" there was no way to stop a long drawing set from
  anywhere. There's now a queue button beside Print, showing what's waiting
  across every printer. Open it to see each printer's jobs and cancel one job,
  everything on one printer, or everything at once. It appears only when
  something is queued.

### Changed
- **Print closes the dialog immediately.** Progress lives in the queue now, so
  there's nothing to wait around for.

### Fixed
- **Content prints centred on a larger sheet.** Printing at actual size on paper
  bigger than the drawing put it in the bottom-left corner, because the sheet
  placement was left to the printing system, which anchors an undersized page at
  the corner. Save as PDF is unchanged.
- **The paper size starts at the size the drawing is actually drawn at**, when
  the printer can load it — rather than always starting at the printer's
  default and needing to be corrected.

## [0.7.0] — 2026-08-23

### Added
- **Saved files now show their markups in every PDF viewer.** ⌘S used to write
  a record only RyDF could read, so a drawing you marked up and emailed arrived
  looking blank in Preview, Acrobat and Bluebeam — Save a Copy was the only way
  to send markups. Saving now writes both: real PDF annotations for everyone
  else, and RyDF's own exact record for itself. Reopening still gives you one
  editable markup per markup, and saving again replaces what it wrote instead
  of stacking a new copy on top each time. Annotations somebody else added in
  Acrobat are left alone. Lines, arrows and polylines carry across too — they
  used to be dropped, so an arrow pointing at the thing you were commenting on
  simply wasn't there for whoever you sent it to.
- **Layers.** A drawing set that arrives with its producer's layers — Balloons,
  Dimensions, Titleblock, Revision Tables — now shows them in a **Layers** tab
  in the sidebar, with a switch for each. Turn off the balloons and dimensions
  to get a clean sheet to redline, then turn them back on. Nothing is changed
  in your file: layer visibility is a view setting, like zoom, and is not saved
  back or added to undo. Documents without layers say so.
- **A filled form now reads as filled everywhere.** Values typed into a PDF's
  form fields are written into the document's own fields on save, with the
  appearance the field needs to draw them — so a W-9 you fill in here opens
  filled in Preview, Acrobat, and anything that reads form data. Tick boxes use
  the state name the form itself defines rather than a guess, and ticking one
  radio button clears its group.
- **Filling in forms.** RyDF used to display every interactive form as blank —
  a filled-in W-9 opened empty, and printed empty, with nothing to say the
  values were there. Fields and their values now show, and you can click one
  and type, or tick a box. What you type is kept alongside your markups rather
  than written into the PDF's own fields, and it prints as it looks.
- **Drawings that state their own scale now use it.** If a sheet carries a
  measurement scale in the file, RyDF reads it and the status bar says so —
  *Scale (p.1, from the drawing): 1 mm = 0.71 pt* — with no calibrating. Your
  own calibration always wins: set one and it replaces the drawing's for that
  page, permanently. Measuring against a scale that came from the drawing
  records it on the document at that moment, so the dimension prints the same
  number it shows.
  Where the drawing states a scale but not which unit it means — which is the
  common case, and reads identically as 1:5 in millimetres or 1:50 in
  centimetres — RyDF offers the reading rather than applying it, because those
  two differ by ten times in every dimension taken from the sheet.
- **Links in the page now work.** Clicking an entry in a PDF's table of contents
  jumps to that page, as do cross references elsewhere in a document; links to a
  web address or an email open in your browser or mail app. RyDF read a
  document's outline but ignored the links drawn on the page itself, so a
  proposal or spec whose contents page is built from links — the common case,
  since those documents often have no outline at all — had dozens of dead
  entries. Hovering a link highlights it and shows where it goes; drawing over
  one still draws.
- **Page numbers the document itself prints.** A drawing set that numbers its
  sheets "FP-111.00" — or a report with roman-numeral front matter — now shows
  those numbers under the page thumbnails and beside the page box, instead of
  only RyDF's own 1..N position. You can type a sheet number into the page box
  to jump straight to it; a partial number like `FP-111` is enough.

### Fixed
- **"Check for Updates" now names the version you're on** — *RyDF 0.7.0 is the
  current version* rather than just telling you there's nothing newer.
- **No more `.markups.json` files next to your drawings.** Autosave used to
  write its crash-recovery copy beside the document, so marking up anything in
  a synced folder scattered stray JSON files through it — files you never asked
  for, that sync and get shared along with the drawing. The recovery copy now
  lives in RyDF's own Application Support folder and nothing is written next to
  your documents at all; markups go in the PDF, where an explicit save puts
  them. Old stray files are still read when you open a drawing, so nothing is
  lost, and saving that drawing clears them away.
- **Saving over the file you already have open no longer destroys it.** Save a
  Copy, Export Flattened and Extract Pages all wrote straight to the chosen
  path — and because RyDF keeps reading the open document from disk as you
  work, choosing the open file as the destination overwrote the pages out from
  under it mid-write. On a 1.7 MB proposal this left a 76 KB file with 2 of its
  29 images and 5 of its 15 fonts, and it still opened, so nothing said
  anything was wrong. Every save now writes alongside the destination and moves
  it into place in one step, which also means an interrupted save (a crash, a
  full disk) can no longer leave a half-written file where your document was.
- **Switching tabs while a document opens no longer loses its markups.** The
  rest of the open — loading the markups, recording the file's fingerprint for
  the external-change watcher — followed whichever tab was in front by the time
  it ran, so moving to another tab mid-open left the new document with no
  markups at all and nothing said so. Saving after that would then write the
  emptied set back over the file's real markups.
- **Markups drawn while a save is running are no longer treated as saved.** A
  save wrote the document as it stood when it started, but cleared the unsaved
  marker when it finished — so anything drawn in between was left unwritten
  with nothing indicating it. Quitting then closed without asking. The window
  is as long as the write takes, which on a Dropbox or network folder is not
  brief.
- **Page operations follow the tab they were started from.** Rotating, deleting,
  moving or inserting pages and then switching tabs before it finished applied
  the result to the tab you moved to, and filed the undo step there as well.
- **↑/↓ keep stepping through search results after you click one.** They worked
  while the cursor was still in the search box, but clicking a result moved the
  keyboard focus onto that row, and from then on the arrows just scrolled the
  results list instead of moving between matches. Clicking a result no longer
  takes focus off the search box, and ↑/↓ now step through matches from anywhere
  in the Search panel. Arrow keys still scroll the page as usual when you're
  working in the document itself.

## [0.6.0] — 2026-08-24

### Added
- **Undo now covers page edits, not just markups.** ⌘Z takes back a page
  deletion — the pages come back exactly as they were, not a re-imported
  approximation — and the same goes for rotating, moving, inserting pages and
  running Recognize Text. Markup edits and page edits share one timeline, so ⌘Z
  always undoes whatever you did last, whichever kind it was, and ⇧⌘Z redoes it.
  A short note tells you what was undone (“Undid Delete 2 pages”).
  Undo is per tab and covers anything that would change the file if you saved
  right now; it deliberately leaves alone what you're merely *looking* at —
  zoom, scrolling, selection, the tool you have picked. **⌘Z never touches the
  disk: it does not un-save a file you already exported or saved a copy of.**
  Password-protected documents keep markup undo but get no page-edit history,
  because that would mean writing an unprotected copy of the file to disk.
- **Tabs can be dragged to reorder them.** Pick a tab up and the others part
  around it; drop it wherever you want it. Grabbing a tab brings its drawing
  forward, as in a browser, and a plain click still just switches to it. With
  more tabs open than fit the strip, holding one against either end scrolls the
  strip along so you can drop it somewhere currently out of view.

### Fixed
- **Text is properly sharp again.** Pages were being rendered at a slightly
  higher resolution than the screen actually needed and then scaled back down
  to fit — and a downscale of a few percent is the worst kind, enough to soften
  every letter without looking obviously wrong. It was most visible on a dense
  price list or schedule next to the same page in Acrobat. Once you settle on a
  zoom, the page is now rendered at exactly the screen's resolution, so one
  rendered pixel lands on one screen pixel and text is as crisp as the display
  allows. Zooming itself is unchanged — the page still scales smoothly under
  your fingers and sharpens the moment you stop.
- **Search finds text on large-format sheets again.** On drawing sets whose
  sheets are exported with the page origin at the centre rather than the corner
  — common for CAD plots — searching for text you could plainly see (and even
  copy) returned “No results”. The background index that decides which pages are
  worth searching was reading each page through a window anchored at the corner,
  so on those sheets it only ever saw a quarter of the page and skipped the rest.
  Nothing was wrong with the pages or your query.
- **Reloading a file changed on disk no longer sits on “Opening…”.** Choosing
  **Reload** loaded the new version immediately but left the opening overlay up
  for a further fifteen seconds before revealing it. The viewer signals “this
  page is on screen” once per document, and it recognised a new document by its
  revision number — but a freshly reopened file always starts at revision zero,
  the same as the one it replaced, so the signal never fired and the overlay
  waited for its own timeout. It now clears as soon as the page is actually
  drawn, about a second.

## [0.5.1] — 2026-08-17

### Fixed
- **Documents that mix page sizes no longer freeze when you scroll between
  them.** On a set with, say, 8.5×11 spec sheets and 34×22 plots, scrolling
  from one size to another could snap the view back and leave the large sheets
  unreachable — the window looked like it had frozen or gone blank. Fit width
  still fits whichever page you're looking at, as it should, so a wide plot
  sizes itself to the window as you reach it; what was broken was the re-fit
  itself. Zooming out to fit a wide page shrinks the document, and the scroll
  position was being clamped to the shorter document *before* the view had been
  re-anchored — so the app re-anchored to the wrong place, landed back on the
  previous page, refit again, and the two zooms fought each other. The re-fit
  now anchors from where you actually were, so it settles immediately on the
  page you scrolled to.
- **A mixed-size document opens centred.** Pages are laid out centred in a
  canvas as wide as the widest page, so a narrow page used to sit off to the
  side — invisible until you scrolled sideways to find it. The view now centres
  on open and stays centred as you scroll between sizes, until you deliberately
  pan sideways.

## [0.5.0] — 2026-08-13

### Added
- **RyDF notices when a drawing is re-saved by another program.** If a PDF
  you have open is changed on disk — re-exported from your CAD app, updated in
  a synced folder — a prompt offers to reload it to the latest version. If you
  have unsaved markups, it warns that reloading discards them and lets you
  **Save a Copy** first to keep your work, **Discard & Reload**, or **Keep
  working** with the version you have. RyDF's own saves never trigger it, and
  each open tab is watched independently.

### Fixed
- **Search highlights and text selection land on the right words again.** On
  some PDFs — ones exported with the page origin shifted off (0,0), like certain
  price sheets — every search highlight and text-selection box sat about a
  word to the left of the text it belonged to, and dragging to copy grabbed the
  wrong words. RyDF was reading text positions in the page's own coordinate
  space but drawing them in the cropped display space without accounting for
  the offset between the two. The same shift also nudged snapped measurements,
  the OCR text layer, and native annotations written into an exported copy —
  all now corrected. Ordinary PDFs (origin at 0,0) are unaffected.

## [0.4.1] — 2026-08-09

### Fixed
- **Copying selected text works again.** With the Select-text tool, dragging
  over text and choosing Copy — or pressing ⌘C — now actually puts the text on
  the clipboard. Two things were wrong: the copy went through a browser API the
  app's webview doesn't reliably expose (so it silently did nothing), and ⌘C had
  no handler at all, because the selection is the app's own highlight rather than
  a normal text selection the system knows about. Copying now uses a path that
  works in the webview, and ⌘C copies whatever text is currently selected.

## [0.4.0] — 2026-08-09

### Added
- **The Print window remembers your printer.** Whichever printer (or “Save as
  PDF”) you last printed to is selected again the next time you open Print,
  instead of always resetting to the system default. If that printer isn't
  connected any more, it quietly falls back to the default.
- **The paper-size list is searchable.** Large-format plotters report dozens of
  sizes; the picker now opens with a search box — type a name (“arch e”) or a
  dimension (“34”) to filter, and use ↑/↓ and Enter to pick without the mouse.

## [0.3.2] — 2026-08-09

### Changed
- **A page looks far less fuzzy while a big sheet is still rendering.** Opening or
  scrolling to a large sheet shows a quick stand-in image until the sharp,
  full-resolution tiles finish rendering. That stand-in was the small sidebar
  thumbnail stretched across the whole page — on an E-size drawing that's about
  a tenfold enlargement, so it read as a blurry mush for the second or two the
  real render took. The stand-in is now drawn at about two-and-a-half times the
  resolution, so it's legible enough to start reading from immediately, and the
  sidebar page thumbnails are a touch crisper too. The final, fully sharp render
  is unchanged and arrives just as fast.

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
