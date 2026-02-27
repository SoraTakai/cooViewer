# cooViewer (Sora fork)

This repository is a maintenance fork of cooViewer.

The focus of this fork is practical reliability for day-to-day reading:

- better history/settings recovery when files move
- predictable app activation when opened from third-party frontends
- better compatibility with automation tools that use macOS accessibility metadata

This README documents the current fork delta only.

## Fork delta from upstream

Functional changes in this fork:

- [`7c263f8`](https://github.com/SoraTakai/cooViewer/commit/7c263f88cc97e2d2c29cb768da62a56e272f24ee): Normalize paths to NFC so history/settings lookup stays stable across Unicode normalization differences. Example: if a filename looks identical but is stored as decomposed text in one place and precomposed text in another, last-page restore still works.
- [`c3553f9`](https://github.com/SoraTakai/cooViewer/commit/c3553f902ff8acbbf067c612a835111bef15d47a): Make close actions terminate the app (quit behavior). Example: using the app's close action now exits cooViewer like `Cmd+Q` instead of only closing the reader window.
- [`db20112`](https://github.com/SoraTakai/cooViewer/commit/db2011292b891b7352374274b30f2c923e39cb19): Add moved-file auto-apply preference and basename-based remap for `RecentItems`/`LastPages` when the original path is missing. Example: move `Series01.cbz` to another folder, reopen it there, and previous page/settings are recovered instead of resetting to page 1.
- [`deef0a1`](https://github.com/SoraTakai/cooViewer/commit/deef0a1e0a36a8e255bd897f8cb1ff12cfc46d94): Activate cooViewer on open so it comes to front when launched by third-party frontends. Example: opening a comic from ComicShelf brings cooViewer to foreground immediately.
- [`42187bc`](https://github.com/SoraTakai/cooViewer/commit/42187bc46ad26c5e216ee19243b607b9617df6c3): Mark the main viewer window as `AXStandardWindow` for automation compatibility. Example: Hammerspoon `win:isStandard()` reports `true` for the main reader window.
- [`b9e8a77`](https://github.com/SoraTakai/cooViewer/commit/b9e8a773d1a1a11c551f0b1843c9b843c529036a): Replace basename matching with SHA-256 signature identity (`historySignature`) sampled from file bytes (`COHistorySignatureSampleBytes`, default `1 MiB` head+tail), build hash indexes for lookup, and canonicalize duplicate entries so latest close state wins across `RecentItems`, `LastPages`, and `BookSettings`. `RecentItems`/`LastPages` fallback is gated by `AutoAcceptMissingSetting`.

## Accessibility notes

- Main viewer window is `AXStandardWindow`.
- Auxiliary windows are still non-standard by design (for example password dialog and accessory/overlay windows).
- During destroy/teardown events, AX reads may transiently return empty values; automation logs can show a final `standard=false` at window destruction. That does not imply a persistent hidden non-standard main window.

## Build

```sh
git clone --recursive https://github.com/SoraTakai/cooViewer.git
cd cooViewer
xcodebuild -project cooViewer.xcodeproj -target cooViewer -configuration Default build CODE_SIGNING_ALLOWED=NO
open build/Default/cooViewer.app
```

## Branch and upstream

- Default branch: `main`
- Fork remote: `https://github.com/SoraTakai/cooViewer`
- Upstream reference: `https://github.com/tak758/cooViewer` (branch `master`)
