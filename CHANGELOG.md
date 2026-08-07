# Changelog

All notable changes to Yomu are documented here. Versioning follows
[Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`), the same
scheme used by GitHub release tags.

## v1.0.0 — first tracked release

The app had no version number before this release, so this is the baseline
everything below is measured against. It bundles an accessibility overhaul,
a new upload review step, and a set of performance and correctness fixes.

### Added
- **Editable upload review step.** Picking or dropping files no longer uploads
  them immediately — you first see each file's detected series name and
  chapter number in editable fields (chapter number supports decimals, series
  name autocompletes against your existing library), with a live "New series"
  / "Adding to existing series X" hint, before anything is added.
- App version is now shown next to the "Yomu" wordmark on the library screen
  and in Settings → About.
- `CHANGELOG.md` (this file).

### Fixed — performance
- **Zip web workers were explicitly disabled** (`useWebWorkers: false`),
  forcing every archive to decompress single-threaded on the main thread.
  Re-enabled; this is the single biggest fix for slow uploads.
- **Zip entry extraction was fully sequential** (one page at a time), which
  meant only one worker in zip.js's pool was ever busy. Extraction is now
  parallelized, with correct handling of wrong/retried passwords across
  concurrent entries.
- **IndexedDB writes used one transaction per page** instead of batching.
  Directly benchmarked: 216ms vs 55ms for a 150-page chapter (~4x) in this
  environment; the gap is typically larger on real device storage. Fixed by
  writing all pages for a chapter in a single transaction.
- Whole-series deletion ran one full transaction per chapter, sequentially;
  now batched into a single transaction.
- PDF pages nested inside a `.zip`/`.cbz` didn't report progress during
  rendering (only stand-alone `.pdf` uploads did) — now consistent.

### Fixed — accessibility
- The chapter options menu (⋯) was a non-focusable `<span role="button">`
  nested inside another `<button>` — invalid HTML, and completely unreachable
  by keyboard. Rebuilt as a real, sibling `<button>`.
- The "Add chapters" dropzone was a `<div>` with no keyboard affordance at
  all. It's now a real `<button>`.
- **Escape did not close any dialog** unless the reader happened to be open
  behind it. It now closes the topmost open dialog.
- Open dialogs had no focus containment — Tab could pass straight through
  them into the page underneath. Dialogs now use `inert` on everything
  behind the active layer (background chrome, the reader, or a
  lower-stacked dialog), and move focus in on open / restore it on close.
- Three form fields had `<label>` elements not associated with their
  `<input>` (password prompt, export password, rename series), and the
  auto-advance toggle had no accessible name at all.
- The search input set `outline:none` with no replacement, leaving keyboard
  focus completely invisible on it; fixed, and extended visible
  `:focus-visible` styling to the rest of the app, which had none.
- Muted/secondary text failed WCAG AA contrast — measured at 3.24:1 (dark
  theme) and 2.86:1 (light theme) against a 4.5:1 requirement. Both palettes
  adjusted to land between 4.8:1–6.4:1.
- Reader page images had no `alt` text; the page slider had no accessible
  name; theme/reading-mode toggle buttons didn't expose their selected state
  (`aria-pressed`) to assistive tech.
- Toasts and upload progress were visually-only; both are now announced via
  `aria-live` regions.
- The reader's auto-hiding top/bottom chrome, and any app chrome sitting
  behind the full-screen reader, remained keyboard-focusable while hidden.
  Both now use `inert` while hidden.

### Fixed — other bugs
- Searching the library, switching tabs, then returning to Library left the
  grid silently filtered by the old search text — the search bar itself was
  hidden, with no indication a filter was still active. Series that didn't
  match were effectively invisible with no explanation. Fixed by clearing
  the search state whenever the bar is hidden.
- A rapid double-tap in the reader could land in a ~40ms gap between the
  single-tap and double-tap timers and trigger both the chrome toggle and
  the zoom toggle at once; tightened the guard.
- Reaching the last page in paged mode and pressing "next" again could fire
  the end-of-chapter prompt twice (vertical/scroll mode already guarded
  against this; paged mode didn't).
- Zero-byte storage usage displayed as "0 MB" instead of "0 B".
- Deleting a series' last remaining chapter left an empty "ghost" series
  behind (0 chapters, a disabled-looking "No chapters yet" primary button,
  an Export button with nothing to export) sitting in the library
  indefinitely. Deleting a series' only chapter now removes the series too.
- The rename-series and export dialogs' text/password fields didn't respond
  to Enter — only clicking the button worked. Both now submit on Enter,
  matching the password-unlock prompt elsewhere in the app.

### Changed
- `service-worker.js` cache version bumped so existing installs pick up all
  of the above.
