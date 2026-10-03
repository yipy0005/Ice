# Ice on macOS 27 "Golden Gate" (27.0.1, build 26A434): notes

## Base and attribution
- Branch `macos-27-support` = `main` (11edd39) merged with upstream PR #995 "macOS 27 support"
  (head 4f1d700, author RabenkoYevhenii, https://github.com/jordanbaird/Ice/pull/995, GPLv3).
  PR #995 is itself built on upstream `macos-26`. Merge commit: 20754ef. No files conflicted.
- PR credits: Thaw PlatformRuntimeKit (https://github.com/thaw-app/Thaw) via Barometer (https://github.com/mackid1993/Barometer).
- Issue #1006 asks for a seeded default `MacOS27Layout` and for ad hoc builds without hardened runtime. The seeding is already
  in the PR (`Concealer27.swift`, `macOS27LayoutSeeded`). No extra code was written for it.

## Toolchain
Xcode 27.0 (27A266a), MacOSX27.0.sdk, macOS 27.0.1 (26A434). Deployment target unchanged (not required to raise).

## What was run
- `swift test` (repo root Package.swift, IceMacOS27Core): 120 tests in 23 suites passed.
- `xcodebuild -project Ice.xcodeproj -scheme Ice -configuration Release -derivedDataPath /tmp/ice-build CODE_SIGN_IDENTITY=- ENABLE_HARDENED_RUNTIME=NO build`:
  ** BUILD SUCCEEDED **. Only warnings (ApplicationServices @preconcurrency, CGWindowListCreateImage deprecated in 14.0 in ScreenCapture.swift). No source fixes were needed.
  `codesign --verify --deep --strict` on Ice.app: passed (no output). The "Copy to Applications" phase is a no-op echo.
- `swift Scripts/macos27/ax-items.swift` (read only): lists ~20 AXMenuExtra items by owner bundle ID (Rectangle, Dropbox, OneDrive, etc.)
  plus the MenuBarAgent "Show Hidden Menu Bar Items" element. AX enumeration of menu bar items works on 27.0.1.

## Runtime status (checklist)
A different Ice (/Applications/Ice.app, pid 5073, the user's installed copy) was running. I did not quit or replace it, and did not launch
the new build next to it, because two instances would both rewrite the menu bar. Not exercised, so **needs manual verification**:
- Launch of the ad hoc build (Sparkle Team ID issue in #1006; built with hardened runtime off), Accessibility / Screen Recording prompts
- Hide/show by click, hover and hotkey (`verify-conceal.sh`, `verify-hover.sh`)
- Control items and dividers; Ice Bar (`verify-icebar*.sh`); Layout editor (`verify-layout.sh`); system item clicks (`verify-clock.sh`)
- SwiftUI settings panes legibility; variable-length item squash (#1007); multi-display and notch
Verified only: build, unit tests, AX item enumeration.

## Known limitations
- The assessment-mode assertion allowlist is honoured only for Developer ID-signed apps, so a locally signed Ice hides its own icon while concealing.
- The assertion is a private API and may change in later 27.x releases.
- For local runs: `defaults write com.jordanbaird.Ice SUAutomaticallyUpdate -bool false`, quit the installed Ice, re-grant Accessibility.
