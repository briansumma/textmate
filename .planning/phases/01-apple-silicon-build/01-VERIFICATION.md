---
phase: 01-apple-silicon-build
verified: 2026-10-19T19:45:23Z
status: passed
score: 24/24 must-haves verified (4 ROADMAP success criteria + 20 plan truths); 4/4 requirements satisfied
behavior_unverified: 0
overrides_applied: 0
covered_files: [".planning/phases/01-apple-silicon-build/01-01-PLAN.md", ".planning/phases/01-apple-silicon-build/01-01-SUMMARY.md", ".planning/phases/01-apple-silicon-build/01-02-PLAN.md", ".planning/phases/01-apple-silicon-build/01-02-SUMMARY.md", ".planning/phases/01-apple-silicon-build/01-03-PLAN.md", ".planning/phases/01-apple-silicon-build/01-03-SUMMARY.md", "README.md", "configure", "default.rave"]
covered_digest: "v3:sha256:37c7bfbbd3e769cca2629512ebb002264a9b163327e35592dca4c96064dd6e99"
must_haves:
  - truth: "SC1: ninja TextMate exits 0 in a clean checkout; lipo -archs prints arm64"
    status: VERIFIED
  - truth: "SC2: configure exits 0 with six Homebrew packages; grep -c /usr/local local.rave prints 0; versions recorded in README"
    status: VERIFIED
  - truth: "SC3: README Building names git submodule update --init --recursive; submodule status shows zero uninitialized"
    status: VERIFIED
  - truth: "SC4: LSMinimumSystemVersion and vtool minos both 26.7.1; 10.12 floor gone"
    status: VERIFIED
  - truth: "01-01 T1: default.rave line 1 APP_MIN_OS 26.7.1; generated build flags carry 26.7.1"
    status: VERIFIED
  - truth: "01-01 T2: configure uses brew --prefix with /opt/homebrew fallback; no legacy prefix on non-comment lines"
    status: VERIFIED
  - truth: "01-01 T3: configure exits non-zero, prints brew install line, writes no local.rave when a tool is missing"
    status: VERIFIED
  - truth: "01-01 T4: configure checks headers and exact libcapnp.dylib and libkj.dylib before any local.rave write"
    status: VERIFIED
  - truth: "01-01 T5: README fenced block has submodule command between clone and configure"
    status: VERIFIED
  - truth: "01-01 T6: README lists the six brew list --versions package versions"
    status: VERIFIED
  - truth: "01-02 U1: recursive clone has no uninitialized or drifted submodules; HEAD equals develop tip"
    status: VERIFIED
  - truth: "01-02 U2: commits since b46e5ae7 (excluding .planning) are exactly the three BUILD subjects with their file sets"
    status: VERIFIED
  - truth: "01-02 U3: configure with builddir exits 0 and writes /opt/homebrew include and library paths"
    status: VERIFIED
  - truth: "01-02 U4: build.ninja carries -mmacosx-version-min=26.7.1 and no 10.12 flag"
    status: VERIFIED
  - truth: "01-02 U5: toolchain recorded as macOS 26.7.1 (25G241) and Xcode 27.1 (27A9275)"
    status: VERIFIED
  - truth: "01-02 U6: ninja TextMate exits 0; TextMate executable reports arm64"
    status: VERIFIED
  - truth: "01-02 U7: Info.plist LSMinimumSystemVersion 26.7.1; vtool minos 26.7.1"
    status: VERIFIED
  - truth: "01-02 U8: non-recursive clone plus README command leaves zero uninitialized submodules"
    status: VERIFIED
  - truth: "01-02 U9: changed non-planning file set is exactly README.md, configure, default.rave, each mapped to a BUILD commit"
    status: VERIFIED
  - truth: "01-03 V1: baseline from real runs of framework runners, outside the build system"
    status: VERIFIED
  - truth: "01-03 V2: Pass and Fail integers copied from logs (spot-checked)"
    status: VERIFIED
  - truth: "01-03 V3: directories with no count line carry n/a and are listed un-baselined"
    status: VERIFIED
  - truth: "01-03 V4: Test baseline names clean HEAD SHA, post-BUILD tree statement, and both differences from 346b52b1"
    status: VERIFIED
  - truth: "01-03 V5: no tracked file changes in either checkout (checked as of verification time)"
    status: VERIFIED
requirements:
  - id: BUILD-01
    status: SATISFIED
    plans: [01-02, 01-03]
  - id: BUILD-02
    status: SATISFIED
    plans: [01-01, 01-02]
    note: "Clean-checkout path satisfied. Existing-checkout upgrade path not (CR-01, WARNING)."
  - id: BUILD-03
    status: SATISFIED
    plans: [01-01, 01-02]
  - id: BUILD-04
    status: SATISFIED
    plans: [01-01, 01-02]
---

# Phase 1: Apple Silicon Build Verification Report

**Phase Goal:** A clean checkout of `develop` builds `TextMate.app` for arm64 on macOS 26.7.1 with Xcode 27.1. The build uses Homebrew dependencies under `/opt/homebrew` and no hand-edited `local.rave`.
**Verified:** 2026-10-19T19:45:23Z
**Status:** passed
**Re-verification:** No (no prior VERIFICATION.md in the phase directory)

## Goal Achievement

The goal holds. The clean checkout at `/Users/bfs/src/textmate-clean` (HEAD `1c0ca4f9`) produced an arm64 `TextMate.app` at `/Users/bfs/build/textmate/release/Applications/TextMate/TextMate.app`. Its generated build flags and linked libraries point at `/opt/homebrew`. Its `local.rave` is the two-line file that `configure` writes, not a hand edit. Its `APP_MIN_OS` is 26.7.1 throughout.

### Observable Truths

Each row was checked against artifacts or a reproduction, not against the SUMMARY claims alone.

| # | Truth | Status | Evidence |
| - | ----- | ------ | -------- |
| SC1 | `ninja TextMate` exits 0; `lipo -archs` prints arm64 | VERIFIED | `lipo -archs` on the app binary prints `arm64`. `ninja -n TextMate` in the clean checkout exits 0. It lists only steps 38/39 (plugin copy and app sign), with no compile steps pending. |
| SC2 | `configure` exits 0 with six packages; `grep -c /usr/local local.rave` is 0; versions in README | VERIFIED | Clean `local.rave` is 74 bytes: `add FLAGS "-I/opt/homebrew/include"` and `add LN_FLAGS "-L/opt/homebrew/lib"`. `grep -c /usr/local` prints 0. The README versions block matches `brew list --versions` for all six packages exactly. `configure` was not re-run in the clean tree. Its artifacts (`local.rave`, `build.ninja`, both 14:25) are the output of that run. |
| SC3 | README names the submodule command; zero uninitialized submodules after it | VERIFIED | README Building fence has `git submodule update --init --recursive` between the clone line and the configure line. Clean checkout `git submodule status` has 0 uninitialized lines. `/Users/bfs/src/textmate-nonrec` has 0 lines starting with `-`. The network clone itself was not re-run. |
| SC4 | `LSMinimumSystemVersion` and `vtool` minos both 26.7.1; 10.12 gone | VERIFIED | Info.plist `LSMinimumSystemVersion` = `26.7.1` (read with PlistBuddy; `plutil` not run). `vtool -show-build` reports `minos 26.7.1`, `sdk 27.0`. `build.ninja` has 702 `-mmacosx-version-min=26.7.1` flags and 0 `10.12`. `default.rave` has no `10.12`. |
| 01-01 T1 | `default.rave` line 1 is `APP_MIN_OS 26.7.1`; build flag carries it | VERIFIED | Freeze diff shows only line 1 changed (`10.12` to `26.7.1`). Line 5 keeps `-std=c++2a`. |
| 01-01 T2 | `configure` uses `brew --prefix`, falls back to `/opt/homebrew`; no legacy prefix on non-comment lines | VERIFIED | `configure` lines 17-24. `git grep /usr/local` over non-planning tracked files has no hit in `configure`. The brew-absent fallback path was checked by code reading only. |
| 01-01 T3 | Missing tool: non-zero exit, brew install line, no `local.rave` | VERIFIED | Reproduced in a scratch copy with `PATH=/usr/bin:/bin`. Output: `*** dependency missing: 'capnp'. Install with: brew install capnp`. Exit 1. No `local.rave` created. |
| 01-01 T4 | Header and exact dylib checks run before any `local.rave` write | VERIFIED | Checks at `configure` lines 29-43. Writes at lines 45-46. The checked paths exist under `/opt/homebrew` (`boost/crc.hpp`, `capnp/message.h`, `sparsehash/dense_hash_map`, `libcapnp.dylib`, `libkj.dylib`). |
| 01-01 T5 | README submodule line sits between clone and configure | VERIFIED | README fenced block: clone, `cd textmate`, `git submodule update --init --recursive`, `./configure && ninja TextMate/run`. |
| 01-01 T6 | README lists the six brew versions | VERIFIED | Versions match `brew list --versions` for boost, capnp, google-sparsehash, multimarkdown, ninja, ragel. |
| 01-02 U1 | Recursive clone has no uninitialized or drifted submodules; HEAD equals develop tip | VERIFIED | Clean checkout has 0 uninitialized submodules. HEAD is `1c0ca4f9`; develop is now `6f19ff37`. `git log 1c0ca4f9..develop -- . ':(exclude).planning'` is empty, so the code tree is identical. The equality holds for code, not for the planning-only tip. |
| 01-02 U2 | Commits since b46e5ae7 (excluding `.planning`) are the three BUILD subjects | VERIFIED | `git log b46e5ae7..develop -- . ':(exclude).planning'` lists `4be57f0d` (BUILD-02), `c71a25b8` (BUILD-03), `bb05e9cc` (BUILD-04). Stats: `default.rave` only; `README.md` only; `configure` and `README.md`. |
| 01-02 U3 | `configure` with builddir exits 0 and writes `/opt/homebrew` paths | VERIFIED | `local.rave` content as in SC2. `build.ninja` has 2 references to `/Users/bfs/build/textmate`. Exit status is from artifact evidence, not a re-run. |
| 01-02 U4 | `build.ninja` carries 26.7.1 and no 10.12 flag | VERIFIED | 702 occurrences of `-mmacosx-version-min=26.7.1`; 0 of `10.12`. |
| 01-02 U5 | Toolchain recorded as macOS 26.7.1 (25G241) and Xcode 27.1 (27A9275) | VERIFIED | Host: `sw_vers` ProductVersion 26.7.1, BuildVersion 25G241. `xcodebuild -version`: Xcode 27.1, build 27A9275. Both match `01-02-SUMMARY.md` lines 99-107. |
| 01-02 U6 | `ninja TextMate` exits 0; executable is arm64 | VERIFIED | `ninja -n` exits 0 with no compile work pending. `lipo -archs` prints `arm64`. The real build was not re-run, to avoid changing the build tree. |
| 01-02 U7 | Info.plist 26.7.1; vtool minos 26.7.1 | VERIFIED | As in SC4. |
| 01-02 U8 | Non-recursive clone plus README command leaves zero uninitialized submodules | VERIFIED | On-disk state of `/Users/bfs/src/textmate-nonrec`: 0 lines starting with `-`. The clone and update step is recorded in `01-02-SUMMARY.md` (lines 179-184) and was not re-run. |
| 01-02 U9 | Changed non-planning file set is exactly README.md, configure, default.rave | VERIFIED | `git diff --stat 346b52b1 -- . ':(exclude).planning'` lists exactly those three files (40 insertions, 7 deletions). |
| 01-03 V1 | Baseline from real runs outside the build system | VERIFIED | 25 run logs in `/Users/bfs/build/textmate-baseline/logs/` with real output, e.g. `Frameworks_io: 1 of 24 tests failed`. |
| 01-03 V2 | Pass and Fail copied from logs | VERIFIED | Spot-checked. `io` row 23/1 matches `1 of 24`. `network` row 0/1 matches `1 of 1`. Full 25-row re-derivation was not repeated. |
| 01-03 V3 | Directories with no count line are `n/a` and un-baselined | VERIFIED | 10 logs have no count line. The table marks `n/a` for the matching rows (buffer, cf, command, editor, file, HTMLOutput, layout, ns, and the others). |
| 01-03 V4 | Baseline section names clean HEAD SHA, post-BUILD statement, both differences | VERIFIED | `01-03-SUMMARY.md` lines 136-142 name `1c0ca4f93e2ae99f5104b4b2e0c9b8828b8aeac5`, the post-BUILD tree, the `-mmacosx-version-min` change, and the `/usr/local` to `/opt/homebrew` change. |
| 01-03 V5 | No tracked file changed in either checkout | VERIFIED | Clean checkout `git status --short` is empty now. The run window itself cannot be re-observed. |

**Score:** 24/24 truths verified.

### Requirements Coverage

| Requirement | Source Plan | Status | Evidence |
| ----------- | ----------- | ------ | -------- |
| BUILD-01 | 01-02, 01-03 | SATISFIED | Clean checkout builds an arm64 app with the host toolchain (macOS 26.7.1, Xcode 27.1). |
| BUILD-02 | 01-01, 01-02 | SATISFIED, with WARNING (CR-01) | Clean-checkout `configure` finds Homebrew deps under `/opt/homebrew` with no hand-edited `local.rave`. Existing checkouts with a stale `local.rave` keep `/usr/local`; see CR-01 below. |
| BUILD-03 | 01-01, 01-02 | SATISFIED | README names the submodule command. Non-recursive clone reaches zero uninitialized submodules. |
| BUILD-04 | 01-01, 01-02 | SATISFIED | `APP_MIN_OS` is 26.7.1. Info.plist and vtool agree. `-std=c++2a` is unchanged. |

Orphan check: `REQUIREMENTS.md` traceability maps only BUILD-01 to BUILD-04 to Phase 1. BUILD-05 belongs to Phase 2. No orphaned Phase 1 requirement.

### Feature Freeze

`git diff 346b52b1 -- . ':(exclude).planning'` in the clean checkout lists exactly `README.md`, `configure`, and `default.rave`. The non-planning tree is identical between `1c0ca4f9` and develop.

### CR-01 Assessment (01-REVIEW.md)

**Verdict: does not break the phase goal. It is a code-quality and upgrade-path defect, recorded as a WARNING.**

Evidence for "does not break the goal":

1. The goal and SC2 are scoped to "a clean checkout of `develop`". `local.rave` is gitignored (`.gitignore:2`) and is absent before the first `configure` run. The clean checkout's `local.rave` is exactly what the new `configure` writes, with `/opt/homebrew` paths and 0 `/usr/local` matches. The CR-01 branch is never reached.
2. CR-01 needs a `local.rave` that already exists. Reviewer's claim: it came from the pre-change `configure`. That makes it a stale generated file, not a hand edit. CR-01 does not itself introduce a hand edit.
3. The clean build's binary links `/opt/homebrew/opt/capnp/lib/libcapnp.1.5.0.dylib` and `libkj.1.5.0.dylib`. `build.ninja` has 0 `/usr/local` references.

Evidence the defect is real:

- Reproduced in scratch with a two-line `/usr/local` `local.rave`. `configure` printed `Skipping writing 'local.rave': File already exists`, ran no prefix or header checks, and left both `/usr/local` lines. (The reproduction exited 127 only because `bin/rave` is absent from the scratch copy. The skip behavior is the finding.)

Why it is still a real gap in BUILD-02's wording:

- BUILD-02 says `configure` finds the deps under `/opt/homebrew` "without hand-editing `local.rave`". For an existing checkout, the only way to reach `/opt/homebrew` is to delete the stale file by hand. That is a manual step the requirement does not allow. The planning checkout (`/Users/bfs/bitbucket/textmate`) has no `local.rave` today, so the defect does not bite there now.

Recommendation (not a blocker for this phase): apply the reviewer's fix before the next checkout that predates this change, either by failing loudly on a `/usr/local` `local.rave` or by regenerating a file that holds only the two generated lines. Decision for Brian: fix now, or accept the upgrade-path gap and record it.

### Other Review Findings (not status-affecting)

- WR-01 (silent `brew --prefix` fallback): the fallback target is the intended prefix, so the goal is unaffected.
- WR-05 (new 26.7.1 floor with no README requirement): intended by BUILD-04. Documentation gap only.
- WR-02, WR-03, WR-04, IN-01 to IN-06: documentation and style. Not checked in depth; none change a must-have.

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
| ---- | ---- | ------- | -------- | ------ |
| `configure` | 26-27 | Silent skip when `local.rave` exists (CR-01) | Warning | Stale `/usr/local` flags kept on existing checkouts. See CR-01. |
| `configure` | 17-24 | Silent fallback when `brew --prefix` fails (WR-01) | Info | Lands on the intended prefix; no goal impact. |
| `README.md` | Setup | No platform floor or Xcode requirement stated (WR-05) | Info | Documentation gap for distributors. |

Debt-marker gate (`TBD`, `FIXME`, `XXX`, `TODO`, `HACK`, `PLACEHOLDER`) in `README.md`, `configure`, `default.rave`: no matches. Gate passes.

### Build Artifact Notes (Info)

- `ninja -n TextMate` reports two pending non-compile steps: a plugin copy and the app sign. A later `ninja TextMate` will redo them. This does not affect the arm64 or minimum-OS result.
- Help and changelog HTML in the app bundle mention `/usr/local`. These are documentation text, not build paths.
- Test baseline: 15 of 25 directories counted, against the 20-row minimum in the 01-03 task action. The user accepted 15 as the final baseline (`01-03-SUMMARY.md`, Minimum Decision). This is a recorded user decision and does not affect BUILD-01 to BUILD-04.

### Human Verification Required

None. Every must-have was checked against artifacts or a direct reproduction. The CR-01 fix-or-accept decision is a decision for Brian, not a verification item.

### Gaps Summary

No gaps. All four ROADMAP success criteria, all 20 plan truths, and all four requirement IDs are verified against the codebase and built artifacts. CR-01 is a WARNING.

---

_Verified: 2026-10-19T19:45:23Z_
_Verifier: Claude (gsd-verifier)_
