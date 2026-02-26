#+TITLE: cooViewer (SoraTakai fork)
#+OPTIONS: toc:2

* Overview
This repository is a maintenance fork of cooViewer.

The purpose of this fork is to keep reader behavior stable for daily use,
improve launch/focus behavior when opened from other apps, and improve
compatibility with automation tools that use macOS accessibility data.

This README intentionally documents only what is in this fork.
Old notes about historical fork merge sources were removed.

* Fork delta (upstream/master -> this repo main)
As of 2026-02-26, this fork is 5 commits ahead of upstream/master.

** 7c263f8 - NFC path normalization for history lookup
Problem:
- macOS paths can look identical but use different Unicode composition forms.
- History lookups (`RecentItems`, `LastPages`, `BookSettings`) could fail,
  so last-page restore sometimes failed.

Change:
- Added path normalization to NFC (`precomposedStringWithCanonicalMapping`).
- Normalization is applied on path ingest, alias/path conversion boundaries,
  and lookup comparisons.

Result:
- Last-page and saved settings lookups are reliable for filenames that differ
  only by Unicode normalization form.

** c3553f9 - Close actions now match Cmd+Q behavior
Change:
- Changed menu/action paths that used window close to app terminate.
- `Close` behavior now quits the app, matching the intended workflow in this fork.

Result:
- Close actions are consistent with quitting behavior.

** db20112 - Moved-file settings auto-apply + page recovery fix
Change:
- Added preference:
  `Auto-apply the same setting for moved files`
  (Japanese: `移動されたファイルに同じ設定を自動で適用する`).
- Added `COAutoAcceptMissingSettingKey` with default ON.
- Added one-time migration key so existing users are moved to ON once.
- Added basename-based fallback remap for moved files in `RecentItems` and
  `LastPages` when original paths are missing.
- Updates stored `temppath` and `alias` to new location during remap.

Result:
- Reopening moved files restores saved page/settings more reliably.
- User can keep auto-apply ON or turn it OFF in Preferences.

** deef0a1 - Bring cooViewer to front when opened by third-party app
Problem:
- When opened by external frontends (for example ComicShelf), cooViewer could
  open behind the current app.

Change:
- Added `[NSApp activateIgnoringOtherApps:YES]` before
  `makeKeyAndOrderFront:` in `openPage:last:`.

Result:
- cooViewer becomes the active front app on open.

Extra housekeeping in same commit:
- Added `cooViewer/` to `.gitignore`.

** 42187bc - Accessibility standard-window fix for main viewer window
Problem:
- Automation tools (for example Hammerspoon) may classify windows by AX subrole.
- Main viewer window could be treated as non-standard in some flows.

Change:
- `CustomWindow` now returns `NSAccessibilityStandardWindowSubrole`.

Result:
- Main viewer window is consistently reported as a standard window.

* Accessibility notes
- Main viewer window: reported as `AXStandardWindow`.
- Some auxiliary windows are still non-standard by design
  (for example password dialog panel, accessory/overlay windows).
- During destroy/teardown events, AX properties may temporarily read as empty;
  automation logs can therefore show a final `standard=false` on destruction.
  That does not imply a persistent hidden non-standard main window.

* Build
Local build example:
#+begin_src sh
git clone --recursive https://github.com/SoraTakai/cooViewer.git
cd cooViewer
xcodebuild -project cooViewer.xcodeproj -target cooViewer -configuration Default build CODE_SIGNING_ALLOWED=NO
open build/Default/cooViewer.app
#+end_src

* Branch and upstream
- Default branch: `main`
- Upstream reference: `https://github.com/tak758/cooViewer` (branch `master`)
