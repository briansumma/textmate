# Roadmap: TextMate (Personal Fork)

## Overview

This fork keeps TextMate 2.0.23 running as a native Apple Silicon app on the current macOS, for one user. The work is maintenance only. No new features are added.

Phase 1 makes a clean arm64 build from a fresh checkout. Phase 2 proves that the built app launches and keeps its working features, and records that run as the baseline. Phase 3 removes the update and crash-report contacts, repeats the run checks on the changed build, and ad-hoc signs the final app.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work

- [ ] **Phase 1: Apple Silicon Build** - A clean checkout builds a runnable arm64 TextMate.app with documented setup
- [ ] **Phase 2: Launch and Run Verification** - The built app launches and keeps its editing, bundle, HTML output, and preference features
- [ ] **Phase 3: Remove Network Contacts and Sign** - Update and crash-upload contacts are removed, and the final build is ad-hoc signed and re-verified

## Phase Details

### Phase 1: Apple Silicon Build

**Goal**: A clean checkout of `develop` builds `TextMate.app` for arm64 on macOS 26.7.1 with Xcode 27.1. The build uses Homebrew dependencies under `/opt/homebrew` and no hand-edited `local.rave`.
**Depends on**: Nothing (first phase)
**Requirements**: BUILD-01, BUILD-02, BUILD-03, BUILD-04
**Success Criteria** (what must be TRUE):
  1. `ninja TextMate` exits 0 in a clean checkout, and `lipo -archs` on the built app binary prints `arm64`
  2. `./configure` exits 0 with the six Homebrew packages installed, and `grep -c /usr/local local.rave` prints `0`. The package versions are recorded in the README "Building" section.
  3. The README "Building" section names `git submodule update --init --recursive`. After that command, `git submodule status | grep -c '^-'` prints `0`.
  4. `plutil -extract LSMinimumSystemVersion raw` prints `26.7.1`, and `vtool -show-build` reports minimum OS 26.7.1. The 10.12 floor is gone.

**Plans**: 1/3 plans executed
- [x] 01-01-PLAN.md
- [ ] 01-02-PLAN.md
- [ ] 01-03-PLAN.md

### Phase 2: Launch and Run Verification

**Goal**: The built app launches on the current macOS and keeps its working features. The run results are recorded as the baseline that Phase 3 is checked against.
**Depends on**: Phase 1 (a built app must exist before the run tests can run)
**Requirements**: RUN-01, RUN-02, RUN-03, RUN-04, RUN-05, BUILD-05
**Success Criteria** (what must be TRUE):
  1. After `open`, the built app is still running 30 seconds later (`pgrep -x TextMate` prints a process ID). No new `TextMate*` report appears in `~/Library/Logs/DiagnosticReports`.
  2. A user can open `runtest-utf8.txt`, type a line, and save. The `shasum -a 256` result matches the expected hash, and reopening the file shows both lines.
  3. Each command in the bundle set exits 0 and its output appears at the declared target. All 60 managed-only bundles load (60 of 60).
  4. A command from `HTML.tmbundle` shows its rendered output in the HTML output window, not a blank window. The About and Preferences windows show content, and a changed preference holds after relaunch.
  5. Each BUILD-05 call site passes its RUN test. A site is either unchanged or migrated with no change in behavior. Compile-only sites are recorded as compile-only verified.

**Plans**: TBD

### Phase 3: Remove Network Contacts and Sign

**Goal**: The built app contacts no `api.textmate.org` host and uploads no crash report. The final build is ad-hoc signed and still passes the Phase 2 run checks.
**Depends on**: Phase 2 (run tests must exist and pass before the removals are checked against them)
**Requirements**: UPD-01, PRIV-01, SIGN-01
**Success Criteria** (what must be TRUE):
  1. `grep -rn 'REST_API' Frameworks Applications` prints nothing. The menu bar has no "Check for Update" item.
  2. The Preferences pane has no "Check Now" button, no bundle-update checkbox, and no "Submit to MacroMates" checkbox.
  3. A DNS capture during RUN-01 to RUN-05 shows no `api.textmate.org` lookup. `grep -rn 'REST_API' Frameworks/CrashReporter Applications/TextMate/src` prints nothing.
  4. The Phase 2 run checks pass again on the changed build, so no working feature breaks.
  5. `codesign -dv --verbose=4` shows `Signature=adhoc` and no `TeamIdentifier` line. `codesign --verify --deep --strict` exits 0. `grep -c APPLE_DEVELOPER_ID local.rave` prints `0`, and the entitlements diff is empty.

**Plans**: TBD
**UI hint**: yes

## Cross-cutting checks

These checks come from REQUIREMENTS.md. They apply across phases and have no requirement IDs.

- First successful build (Phase 1): run `ninja <framework>/test` for each framework that has a `tests` directory. Record the pass and fail list against baseline commit `346b52b1`. Phase 3 must show no new failures.
- Feature freeze (Phase 3 close): `git diff 346b52b1 -- . ':(exclude).planning'` lists the changed files. Each file maps to one requirement ID in its commit message.
- Local only (Phase 3 close): `git push` to `origin` fails, or `origin` is removed.

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Apple Silicon Build | 1/3 | In Progress | - |
| 2. Launch and Run Verification | 0/TBD | Not started | - |
| 3. Remove Network Contacts and Sign | 0/TBD | Not started | - |
