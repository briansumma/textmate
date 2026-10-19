---
phase: 01-apple-silicon-build
plan: 02
subsystem: build
tags: [clean-checkout, arm64, xcode, homebrew, submodules, deployment-target, verification]

# Dependency graph
requires:
  - phase: 01-apple-silicon-build (plan 01)
    provides: BUILD-04, BUILD-03, BUILD-02 source edits at bb05e9cc, c71a25b8, 4be57f0d
provides:
  - Clean-checkout build proof of develop 1c0ca4f9 (arm64 TextMate.app, minos 26.7.1)
  - configure proof with /opt/homebrew include and library paths, and no legacy prefix
  - Non-recursive clone proof that the README submodule command initializes all 7 submodules
affects: [01-03 test baseline]

# Actuals (#2632) — chars/4 over the realized tracked source diff. No tracked source file changed.
actuals:
  tokens: 0
  tasks: 3
  commits: 0

# Commit ledger (#3968) — measured from HEAD at plan start
commits: 0
plan_head_before: 1c0ca4f93e2ae99f5104b4b2e0c9b8828b8aeac5
plan_head_after: 1c0ca4f93e2ae99f5104b4b2e0c9b8828b8aeac5

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Clean proof: git clone --recursive of develop into an empty directory outside the repository"
    - "Long ninja build runs in the background with a log file, then is polled"

key-files:
  created: []
  modified: []

key-decisions:
  - "Proof passes builddir= to configure as an environment override. README line 79 does not document it, so README stays out of scope."
  - "Deprecated-API warnings from the build are left unmigrated. BUILD-05 is Phase 2 work."

patterns-established:
  - "Commit-under-test gate: subjects since b46e5ae7 (excluding .planning) must be exactly the three BUILD subjects before any clone"

requirements-completed: [BUILD-01, BUILD-02, BUILD-03, BUILD-04]

duration: not recorded (record_start_time step was not run)
completed: 2026-10-19
status: complete
---

# Phase 1 Plan 02: Clean-checkout build proof Summary

**A clean `git clone --recursive` of develop at 1c0ca4f9 configures with /opt/homebrew paths and builds an arm64 TextMate.app with minos 26.7.1; a clone made without `--recursive` shows the README submodule command takes 7 uninitialized submodules to 0.**

## Performance

- **Duration:** not recorded (the start timestamp was not captured at plan start)
- **Completed:** 2026-10-19
- **Tasks:** 3 of 3 verified, no checkpoint
- **Tracked files modified:** 0

## Accomplishments

- **BUILD-01:** `ninja TextMate` exits 0 in a clean recursive clone of develop. `lipo -archs` prints `arm64`.
- **BUILD-02:** `configure` exits 0 with `builddir` set. Its `local.rave` has the `/opt/homebrew` include and library paths, with no `/usr/local` and no `APPLE_DEVELOPER_ID`.
- **BUILD-03:** In a clone made without `--recursive`, `submodule status | grep -c '^-'` is 7 before the README command and 0 after it.
- **BUILD-04:** The built `Info.plist` reads `LSMinimumSystemVersion` 26.7.1. `vtool` reports `minos 26.7.1`. The clean `build.ninja` has 702 lines with `mmacosx-version-min=26.7.1` and 0 with `10.12`.

## Clean checkout

| Item | Value |
| ---- | ----- |
| Clean checkout path | /Users/bfs/src/textmate-clean (outside the repository, untracked) |
| Clone command | `git clone --recursive --branch develop /Users/bfs/bitbucket/textmate /Users/bfs/src/textmate-clean` |
| Clean checkout HEAD | 1c0ca4f93e2ae99f5104b4b2e0c9b8828b8aeac5 |
| develop tip, planning checkout | 1c0ca4f93e2ae99f5104b4b2e0c9b8828b8aeac5 (equal) |
| Submodule status after recursive clone | 7 lines, 0 starting with `-`, `+`, or `U` |
| `git status --porcelain` in clean checkout after build | empty (`build.ninja` and `local.rave` are ignored) |

**Commit under test.** The subjects since `b46e5ae7`, excluding `.planning`, are exactly the three BUILD subjects. `b46e5ae7` is an ancestor of HEAD.

- BUILD-02: find Homebrew dependencies under the brew prefix
- BUILD-03: document submodule init before configure
- BUILD-04: set deployment target to 26.7.1

Per-commit file sets: BUILD-04 touches `default.rave`. BUILD-03 touches `README.md`. BUILD-02 touches `configure` and `README.md`.

The develop tip (1c0ca4f9) is a planning-only commit (`docs(01-01)`). The `.planning` pathspec excludes it, so the code under test is the three BUILD commits.

## Toolchain

Verbatim output, run before configure and before the build:

```
ProductName:		macOS
ProductVersion:		26.7.1
BuildVersion:		25G241
```

```
Xcode 27.1
Build version 27A9275
```

This matches macOS 26.7.1 (build 25G241) and Xcode 27.1 (build 27A9275), the values in REQUIREMENTS.md.

## Configure

- **Command:** `cd /Users/bfs/src/textmate-clean && builddir="$HOME/build/textmate" ./configure`
- **Exit status:** 0 (log: /Users/bfs/build/textmate-phase1-configure.log)
- **Override note:** This proof passes `builddir=` as an environment override. README line 79 does not document it. README stays out of scope and was not edited.
- **`local.rave` (verbatim):**

  ```
  add FLAGS    "-I/opt/homebrew/include"
  add LN_FLAGS "-L/opt/homebrew/lib"
  ```

- **grep counts in `local.rave`:** `/usr/local` = 0. `APPLE_DEVELOPER_ID` = 0. `-I/opt/homebrew/include` present. `-L/opt/homebrew/lib` present.
- **`build.ninja` checks:** `builddir = /Users/bfs/build/textmate` matched once. `mmacosx-version-min=10.12` count = 0. `mmacosx-version-min=26.7.1` count = 702.
- **Output location:** bin/rave's release configuration places products under `/Users/bfs/build/textmate/release/`, which is inside `~/build/textmate` as the plan expects.

## Build

- **Command:** `ninja TextMate`, run from the clean checkout in the background, with output to /Users/bfs/build/textmate-phase1-ninja.log.
- **Exit status:** 0. The background wrapper echoed `ninja exit: 0` from `$?`.
- **Failure markers:** 0 `FAILED` lines. No `ninja: build stopped` line.
- **Code signing:** The `Sign ... TextMate.app` step completed (`replacing existing signature`). No timestamp request failed.
- **Warnings:** The build prints deprecated-API warnings (for example `LSItemInfoRecord` and `launchApplicationAtURL`). They were not migrated. They are BUILD-05 and belong to Phase 2.
- **App:** /Users/bfs/build/textmate/release/Applications/TextMate/TextMate.app

**Built-app checks**

- `lipo -archs` on `Contents/MacOS/TextMate`: `arm64`
- `plutil -extract LSMinimumSystemVersion raw` on `Contents/Info.plist`: `26.7.1`
- `vtool -show-build` on `Contents/MacOS/TextMate` (minos line and surrounding block):

  ```
  Load command 11
        cmd LC_BUILD_VERSION
    cmdsize 32
   platform MACOS
      minos 26.7.1
        sdk 27.0
     ntools 1
       tool LD
    version 27037.1
  ```

- Numeric minos check (major 26, minor 7, patch 1): pass.

**Last 20 lines of the build log (verbatim):**

```
[834/853] Copy ‘'Frameworks/SoftwareUpdate/resources/Update Badge.tiff'’…
[835/853] Copy ‘Frameworks/Preferences/icons/Variables.png’…
[836/853] Copy ‘Frameworks/DocumentWindow/resources/TouchBarQuickOpenTemplate.png’…
[837/853] Copy ‘'Frameworks/Preferences/icons/Variables@2x.png'’…
[838/853] Copy ‘'Applications/TextMate/resources/TextMate Help/images/tm_small.png'’…
[839/853] Copy ‘Frameworks/DocumentWindow/resources/TouchBarNewTabTemplate.png’…
[840/853] Copy ‘Applications/TextMate/icons/XML.icns’…
[841/853] Copy ‘Applications/TextMate/icons/YAML.icns’…
[842/853] Copy ‘'Applications/TextMate/support/Bundles/Avian.tmbundle/Commands/Compress Selected Items.tmCommand'’…
[843/853] Copy ‘'Frameworks/OakTextView/resources/warning Template.pdf'’…
[844/853] Copy ‘'Applications/TextMate/support/Bundles/Avian.tmbundle/Commands/Encrypt on Save.tmCommand'’…
[845/853] Copy ‘'Applications/TextMate/support/Bundles/Avian.tmbundle/Commands/Import GZip.tmCommand'’…
[846/853] Copy ‘Applications/TextMate/resources/WKWebView.js’…
[847/853] Copy ‘'Applications/TextMate/support/Bundles/Avian.tmbundle/Commands/Import PNG Image.tmCommand'’…
[848/853] Copy ‘'Applications/TextMate/support/Bundles/Avian.tmbundle/Commands/Show Images.tmCommand'’…
[849/853] Copy ‘'Applications/TextMate/support/Bundles/Avian.tmbundle/Commands/Import BZip2.tmCommand'’…
[850/853] Copy ‘Applications/TextMate/support/Bundles/Avian.tmbundle/info.plist’…
[851/853] Copy ‘Applications/TextMate/support/Bundles/Avian.tmbundle/Commands/Decrypt.tmCommand’…
[852/853] Sign ‘/Users/bfs/build/textmate/release/Applications/TextMate/TextMate.app’…
/Users/bfs/build/textmate/release/Applications/TextMate/TextMate.app: replacing existing signature
```

## Non-recursive BUILD-03 check

- **Clone:** `git clone --branch develop /Users/bfs/bitbucket/textmate /Users/bfs/src/textmate-nonrec` (no `--recursive`), exit 0.
- **Before the README command:** `git submodule status | grep -c '^-'` = 7.
- **README command run:** `git submodule update --init --recursive`, the command at README.md line 58, exit 0.
- **After the README command:** `grep -c '^-'` = 0. `grep -c '^[-+U]'` = 0.

## Feature freeze

- **Changed non-planning files since 346b52b1:** `README.md`, `configure`, `default.rave`. This is exactly the named set.
- **Non-planning commit subjects since 346b52b1:**
  - BUILD-02: find Homebrew dependencies under the brew prefix
  - BUILD-03: document submodule init before configure
  - BUILD-04: set deployment target to 26.7.1
- **Subjects that do not start with `BUILD-0`:** 0.
- **Pre-phase tree:** the non-planning tree at b46e5ae7 equals the one at 346b52b1.

## Working tree notes

The planning checkout has uncommitted state that predates this plan and is not part of it. It was not committed here:

- Modified: `.planning/config.json`
- Untracked: `.planning/milestone.lock`, `.planning/phases/01-apple-silicon-build/01-PATTERNS.md`, `.planning/state.json`

## Deviations from Plan

None. The plan executed as written, with these observations:

- The protocol 0c ledger was not written at plan start. The base for the commit count is HEAD at plan start, 1c0ca4f9. Measured against that base, `git rev-list --count 1c0ca4f9..HEAD` = 0 before this SUMMARY commit. This plan made no code commits, so `commits: 0` is correct.
- `actuals.tokens` is 0 because no tracked source file changed. chars/4 over the realized source diff is 0.

## Issues Encountered

- The build prints deprecated-API warnings. They are BUILD-05 work, which is Phase 2. No build step failed, so no checkpoint was needed.
- README line 79 has documentation drift, which is out of scope. It was not edited.

## Known Stubs

None.

## Threat Flags

None. No tracked file changed, and the build ran from an untracked clone outside the repository.

## Self-Check: PASSED

- FOUND: /Users/bfs/src/textmate-clean/local.rave
- FOUND: /Users/bfs/src/textmate-clean/build.ninja
- FOUND: /Users/bfs/src/textmate-nonrec/configure
- FOUND: /Users/bfs/build/textmate-phase1-configure.log
- FOUND: /Users/bfs/build/textmate-phase1-ninja.log
- FOUND commits: bb05e9cc (BUILD-04), c71a25b8 (BUILD-03), 4be57f0d (BUILD-02)

## Next Phase Readiness

- Plan 01-03 can run the test baseline against clean checkout HEAD 1c0ca4f93e2ae99f5104b4b2e0c9b8828b8aeac5 at /Users/bfs/src/textmate-clean. The build products are under /Users/bfs/build/textmate/release/.
- Phase 2 (BUILD-05) owns the deprecated-API warnings listed in Build.

---
*Phase: 01-apple-silicon-build*
*Completed: 2026-10-19*
