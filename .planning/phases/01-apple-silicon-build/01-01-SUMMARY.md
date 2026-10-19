---
phase: 01-apple-silicon-build
plan: 01
subsystem: build
tags: [homebrew, rave, configure, readme, deployment-target, submodules]

# Dependency graph
requires:
  - phase: none
    provides: plan 01-01 starts from develop at b46e5ae7 (planning commit 0ae5e5cf)
provides:
  - APP_MIN_OS set to 26.7.1 in default.rave (BUILD-04)
  - Submodule init command in the README Building section (BUILD-03)
  - configure Homebrew prefix lookup with /opt/homebrew fallback and pre-write checks (BUILD-02)
  - Recorded Homebrew package versions in the README Building section (BUILD-02)
affects: [01-02 clean-checkout build verification]

# Actuals (#2632) — chars/4 over the realized diff of the three source files
actuals:
  tokens: 894
  tasks: 3
  commits: 3

commits: 3
plan_head_before: 0ae5e5cf4bf3c6552f923a31313584563cdc481b
plan_head_after: 4be57f0d331863b11e73850efce60711bcb39db6

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "configure resolves the Homebrew prefix with brew --prefix and falls back to /opt/homebrew"
    - "configure checks exact .dylib files and headers before its first local.rave write"
    - "Each missing Homebrew formula prints its brew install line through error"

key-files:
  created: []
  modified:
    - default.rave
    - configure
    - README.md

key-decisions:
  - "APP_MIN_OS is 26.7.1 on default.rave line 1 (D-05). Info.plist is not edited (D-06)."
  - "-std=c++2a stays on line 5 and no -arch flag is added (D-07, D-08)."
  - "configure falls back to /opt/homebrew when brew is missing or prints nothing (D-01)."
  - "configure writes -I and -L paths only after every check passes, so a failed run leaves no partial local.rave (D-02, D-03)."
  - "Library checks test the exact libcapnp.dylib and libkj.dylib files. A glob breaks test -f on Homebrew."
  - "Package versions come from brew list --versions and match the planner's values exactly (D-04)."
  - "One commit per requirement ID, in the order BUILD-04, BUILD-03, BUILD-02 (D-12)."

patterns-established:
  - "Missing-tool errors name the formula: error \"... Install with: brew install <formula>\""
  - "Pre-write checks: a failed configure run never leaves a partial local.rave behind"

requirements-completed: [BUILD-02, BUILD-03, BUILD-04]

duration: not recorded (record_start_time step was not run)
completed: 2026-10-19
status: complete
---

# Phase 1 Plan 01: Source edits for the Apple Silicon build Summary

**APP_MIN_OS set to 26.7.1, submodule init documented before configure, and configure resolves Homebrew under brew --prefix with pre-write checks and per-formula install hints**

## Performance

- **Duration:** not recorded (the start timestamp was not captured at plan start)
- **Started:** not recorded
- **Completed:** 2026-10-19
- **Tasks:** 3 of 3 completed
- **Files modified:** 3 (default.rave, configure, README.md)

## Accomplishments

- `default.rave` line 1 now reads `set APP_MIN_OS    "26.7.1"`. The generated build file carries 592 occurrences of `mmacosx-version-min=26.7.1` and zero of `10.12`.
- The README clone block now runs `git submodule update --init --recursive` between `cd textmate` and `./configure && ninja TextMate/run`.
- `configure` finds the Homebrew prefix from `brew --prefix` with a `/opt/homebrew` fallback. It checks the headers and the exact `libcapnp.dylib` and `libkj.dylib` files before writing `local.rave`. It prints a `brew install <formula>` line for each missing tool.
- The README records six package versions from `brew list --versions`, in a text block after the install block.

## Task Commits

Each task was committed atomically:

1. **Task 1: BUILD-04 set deployment target to 26.7.1** - `bb05e9cc` (default.rave only)
2. **Task 2: BUILD-03 document submodule init before configure** - `c71a25b8` (README.md, one added line)
3. **Task 3: BUILD-02 find Homebrew dependencies under the brew prefix** - `4be57f0d` (configure and README.md)

**Plan metadata:** committed separately with the state files (see the final commit below).

## Verify Output

**Task 1 (BUILD-04)**

- `grep -c -F 'set APP_MIN_OS    "26.7.1"' default.rave` = 1
- `grep -c -F -- '-std=c++2a' default.rave` = 1 (unchanged; baseline 1)
- `bin/rave -crelease -tTextMate -b"$HOME/build/textmate-rave-check/build" -o"$HOME/build/textmate-rave-check/build.ninja"` exited 0. Its output showed only the expected submodule warnings ("reference is missing", "Unknown target 'Dialog2/debug'"). The generated build file had 592 `mmacosx-version-min=26.7.1` lines and 0 `mmacosx-version-min=10.12` lines. The baseline 10.12 count at HEAD was 592, so the flag count matches.
- `git diff --quiet b46e5ae7 -- Applications/TextMate/Info.plist` exits 0.

**Task 2 (BUILD-03)**

- `grep -c -F 'git submodule update --init --recursive' README.md` = 1 (baseline 0)
- Line numbers: clone 45, submodule 47, configure 48. The submodule line falls strictly between them.
- `git show --numstat` for `c71a25b8` on README.md: `1	0	README.md`

**Task 3 (BUILD-02)**

- `sh -n configure` exits 0.
- `grep -v '^#' configure | grep -c -F '/usr/local'` = 0 (baseline 4)
- `brew --prefix` and `/opt/homebrew` and `lib${lib}.dylib` are each present in configure (baseline 0 each).
- `PATH=/usr/bin:/bin sh ./configure`: exit 1, printed `*** dependency missing: ‘capnp’. Install with: brew install capnp`, and no `local.rave` was written.
- Success path, run in a scratch copy of configure (not the checkout) with Homebrew on PATH: all header and library checks passed. `local.rave` contained `add FLAGS    "-I/opt/homebrew/include"` and `add LN_FLAGS "-L/opt/homebrew/lib"`. The script then stopped at the missing `bin/rave`, which is expected in the scratch directory.
- `brew list --versions boost capnp google-sparsehash multimarkdown ragel ninja` matched all six README lines.

**Plan-level verification (after all three commits)**

- `git log --format=%s b46e5ae7..HEAD -- . ':(exclude).planning'` lists the BUILD-02, BUILD-03, and BUILD-04 subjects.
- `git diff --name-only b46e5ae7 HEAD -- . ':(exclude).planning'` prints `README.md`, `configure`, `default.rave`.
- `grep -c -F 'set APP_MIN_OS    "26.7.1"' default.rave` = 1
- `grep -c -F 'git submodule update --init --recursive' README.md` = 1
- `grep -v '^#' configure | grep -c -F '/usr/local'` = 0
- `Applications/TextMate/Info.plist` and `local-orig.rave` are unchanged.
- No `local.rave` or `build.ninja` exists in the checkout.

## Files Created/Modified

- `default.rave` - line 1: `APP_MIN_OS` changed from `10.12` to `26.7.1`. No other line changed.
- `configure` - dependency checks split into two loops; prefix lookup added; header and library checks moved ahead of the `local.rave` writes; exact-file dylib checks; `-I` and `-L` paths built from the prefix.
- `README.md` - one submodule line in the clone block; one sentence and a six-line version block after the install block.

## Decisions Made

- Used the decisions in 01-CONTEXT.md (D-01 through D-12). No new decisions were needed.
- The planner's version values matched `brew list --versions` exactly on 2026-10-19, so no README values were changed.

## Deviations from Plan

None - plan executed exactly as written.

The commit ledger from protocol 0c (`gsd-plan-head-before-*`) was not written before the first commit. The base for the commit count was reconstructed as the first commit's parent, `bb05e9cc^ = 0ae5e5cf`, which was HEAD at plan start. The count of 3 is measured from that base.

## Issues Encountered

- `bin/rave` prints "reference is missing" warnings and "Unknown target 'Dialog2/debug'" because submodules are not initialized in this planning checkout. The plan expected these warnings.
- Pre-existing and out of scope (not changed, per the plan): `configure` line 32 tests `$builddir`, which is never set in `configure`, so the `-b` build directory is only honored when the caller exports `builddir`.
- `actuals.tokens` is 894. This is chars/4 over the 3574-character realized diff of the three files. It is well below the plan's estimate of 20000. Future estimates for source-edit plans of this size should use the realized value.

## Known Stubs

None.

## Threat Flags

None. All changes stay within the three named files. `configure` runs no `sudo` and uses no `eval`. `local.rave` is not copied or hand-edited.

## Self-Check: PASSED

- FOUND: default.rave, configure, README.md
- FOUND commits: bb05e9cc (BUILD-04), c71a25b8 (BUILD-03), 4be57f0d (BUILD-02)

## Next Phase Readiness

- Plan 01-02 can build a clean checkout of develop. It must include the three commits above.
- Still to verify in 01-02: the built `TextMate.app` plist value (`plutil -extract LSMinimumSystemVersion raw ...` should print `26.7.1`), the clean-checkout 10.12 absence check, and the no-Developer-ID check on the generated `local.rave`.

---
*Phase: 01-apple-silicon-build*
*Completed: 2026-10-19*
