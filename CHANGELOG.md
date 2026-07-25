# Changelog

All notable changes to RyDF are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/); versions are `MAJOR.MINOR.PATCH`
matching `src-tauri/Cargo.toml` and `src-tauri/tauri.conf.json`.

The in-app updater reads this file — at each release tag — to show the user
exactly what changed between the version they have and the one on offer, so keep
the `## [x.y.z] — YYYY-MM-DD` heading format stable.

## [Unreleased]

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
