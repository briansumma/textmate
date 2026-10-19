---
phase: 02-launch-and-run-verification
plan: 03
subsystem: testing
tags: [runtests, run-02, build-05, fixture, preflight, compat-h]

# Dependency graph
requires:
  - phase: 02-launch-and-run-verification
    provides: ~/runtests harness and RUN-01 row (plan 02-01)
provides:
  - RUN-02 fixture with a pre-test initial hash and a distinct expected hash
  - BUILD-05 call-site table with tracked, object, reached-by, and default disposition per site
  - compat.h baseline hashes for lines 8-16 and 18-21 for plan 06
affects: [02-04, 02-05, 02-06]

# Actuals (#2632): chars/4 over the harness files actually changed (outside the repo).
# No tracked repo file changed in this plan.
actuals:
  tokens: 954
  tasks: 2
  commits: 0

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "RUN-02 pass rule is byte equality via shasum -a 256; expected hash fixed before the GUI edit"
    - "Read-only preflight: build objects and tracked status checked against the built commit 1c0ca4f9"

key-files:
  created:
    - ~/runtests/runtest-utf8.txt
    - ~/runtests/runtest-utf8.initial.sha256
    - ~/runtests/runtest-utf8.expected.sha256
    - ~/runtests/expected/runtest-utf8.expected.txt
    - ~/runtests/logs/RUN-02.pre.txt
    - ~/runtests/lists/build05-sites.tsv
    - ~/runtests/logs/BUILD-05.compat-baseline.txt
    - ~/runtests/logs/BUILD-05.preflight.txt
  modified: []

key-decisions:
  - "The RUN-02 expected hash is computed from the exact two-line byte string before the GUI edit, so a failure cannot be argued after the fact"
  - "Site c (Gestalt) is recorded as exercised by RUN-02; RUN-01 reach is recorded as unresolved, not guessed"
  - "No BUILD-05 call site is migrated in this plan; all defaults stay unchanged unless a RUN test fails"

patterns-established:
  - "Fixture hashes use relative names (runtest-utf8.txt) so shasum -c checks the saved file directly"

requirements-completed: [RUN-02, BUILD-05]

# Coverage metadata (#1602)
coverage:
  - id: D1
    description: "The RUN-02 fixture is ASCII-only, has an initial hash that verifies, and has an expected hash that differs from it."
    requirement: "RUN-02"
    verification:
      - kind: other
        ref: "shasum -a 256 -c ~/runtests/runtest-utf8.initial.sha256 plus the hash-difference and non-ASCII byte-count checks (rc 0)"
        status: pass
    human_judgment: false
  - id: D2
    description: "Every BUILD-05 call site (a to i) has a row with tracked status, object, reached-by, and disposition; compat.h baselines are recorded."
    requirement: "BUILD-05"
    verification:
      - kind: other
        ref: "BUILD-05 verify command in 02-03-PLAN.md Task 2 (rc 0): 10 table lines, RUN-01 (unresolved) present, ODBEditorSuite.o present, compat.h lines 18-21 match baseline"
        status: pass
    human_judgment: false

# Metrics
duration: not recorded
completed: 2026-10-19
status: complete
---

# Phase 2 Plan 03: RUN-02 Fixture and BUILD-05 Preflight Summary

**RUN-02 fixture with a fixed expected hash (0524aa99...) recorded before the GUI edit, and a BUILD-05 site table covering sites a to i with compat.h baselines**

## Performance

- **Duration:** not recorded
- **Completed:** 2026-10-19
- **Tasks:** 2 (fixture, preflight)
- **Files modified in repo:** 0 tracked files. This SUMMARY.md is the only repo file added.

## Accomplishments

- Wrote `~/runtests/runtest-utf8.txt` with the single line `runtest line one`. Its sha256 (`98ab0ad6...`) matches `runtest-utf8.initial.sha256`.
- Wrote `~/runtests/expected/runtest-utf8.expected.txt` with `runtest line one` and `runtest line two`. Its sha256 (`0524aa99...`) is recorded as `runtest-utf8.expected.sha256` under the name `runtest-utf8.txt`, so `shasum -c` checks the saved file after the GUI edit.
- The initial and expected hashes differ. The fixture is ASCII-only (non-ASCII byte count 0).
- Recorded both hashes and the pre-test note in `~/runtests/logs/RUN-02.pre.txt` before the GUI edit.
- Confirmed the code tree matches the built commit: `git diff --name-only 1c0ca4f9 -- . ':(exclude).planning'` is empty. The clean checkout `~/src/textmate-clean` is at 1c0ca4f9.
- Confirmed tracked status for all nine call-site files. Confirmed `ODBEditorSuite.o`, `path_info.o`, `user_agent.o`, `resource.o`, and the other object files exist under `~/build/textmate/release/_CompileClang`.
- Recorded the compat.h baselines: lines 8-16 `f917b112...`, lines 18-21 `dfa67021...`. Both match the planning-time values.
- Wrote the site table `~/runtests/lists/build05-sites.tsv` (header plus nine rows, a to i).

## Task Commits

No per-task commits. The plan changes no tracked repo files. Fixture, site table, and logs are under `~/runtests`, outside the repository.

**Plan metadata:** the commit that adds this SUMMARY.md (`docs(02-03)`). The commit ledger shows 0 plan commits before it, which is legitimate for a plan with no code changes.

## Files Created/Modified

Outside the repository (`~/runtests`):
- `~/runtests/runtest-utf8.txt` - RUN-02 input fixture, one line, 17 bytes
- `~/runtests/runtest-utf8.initial.sha256` - initial hash, relative name
- `~/runtests/runtest-utf8.expected.sha256` - expected post-edit hash, relative name `runtest-utf8.txt`
- `~/runtests/expected/runtest-utf8.expected.txt` - the two expected lines
- `~/runtests/logs/RUN-02.pre.txt` - pre-test record of both hashes
- `~/runtests/lists/build05-sites.tsv` - BUILD-05 site table, header plus sites a to i
- `~/runtests/logs/BUILD-05.compat-baseline.txt` - compat.h line-range hashes
- `~/runtests/logs/BUILD-05.preflight.txt` - code-diff result (empty), HEAD values, rebuild command (not run)

Inside the repository: this file only.

## Decisions Made

- **Expected hash fixed before the GUI edit.** The expected file is the exact two-line byte string, so the RUN-02 result is a byte comparison, not a judgment.
- **Site c reach recorded as partly unresolved.** `path_attributes` (open.cc:263, save.cc:183) reaches Gestalt for RUN-02. RUN-01 reach through `user_agent.cc:33` or the `AppController.mm:716` menu action was not traced. The row says so.
- **No call site migrated.** Every default stays unchanged unless a RUN test fails (REQUIREMENTS Decisions 3 and 8 to 12).

## Site Table (BUILD-05 preflight)

| Site | Call | Tracked | Object | Reached by | Default |
|------|------|---------|--------|-----------|---------|
| a | NSURLConnection canHandleRequest, OakHTMLOutputView.mm:199 | yes | yes | RUN-04 only via policy delegate; output has no link, not exercised | unchanged; OPEN ITEM for Brian (A1 does not cover) |
| b | NSTask launchPath, SoftwareUpdate.mm:557 | yes | yes | none (update install action) | unchanged; not exercised; ACCEPTED-DEVIATION under A1 |
| c | Gestalt, compat.h:8-16 | yes | yes | RUN-02 (exercised); RUN-03, RUN-05 inferred from shared open path; RUN-01 (unresolved) | unchanged (Decision 8) |
| d | Get1Resource, resource.cc:34 | yes | yes | none (no caller outside resource.* files) | unchanged (Decision 9); not exercised; ACCEPTED-DEVIATION under A1 |
| e | GetIconRef, TMFileReference.mm:257-260 | yes | yes | none (RUN-02 does not check icon) | unchanged (Decision 10); not exercised; ACCEPTED-DEVIATION under A1 |
| f | WebView registerURLSchemeAsLocal, OakCommand.mm:679 | yes | yes | RUN-01, RUN-04 | unchanged unless RUN-01 or RUN-04 fails (Decision 11) |
| g | Apple Events and ODB, ODBEditorSuite.mm | yes | yes | none (no RUN test sends an ODB event) | unchanged (Decision 12); not exercised; ACCEPTED-DEVIATION under A1 |
| h | AuthorizationExecuteWithPrivileges wrapper, compat.h:18-21 | yes | yes | RUN-01 only if mate install prompts at launch | compile-only verified; unchanged (Decision 3) |
| i | AuthorizationExecuteWithPrivileges call sites, TerminalPreferences.mm:50, server.cc:26 | yes | yes | RUN-01 only if mate install prompts at launch | compile-only verified; unchanged (Decision 3) |

## Deviations from Plan

None. The plan's read-only steps, fixture steps, and verify commands ran as written. Every verify command passed.

## Issues Encountered

None.

## Open Items for Brian

- **Site a** (`OakHTMLOutputView.mm:199`, NSURLConnection) is not exercised by RUN-04 as written, because the RUN-04 output has no link. A1 does not cover it. Decide whether to add a link case to RUN-04 or accept the gap.
- **Site c, RUN-01 reach** is unresolved. A Gestalt migration would need a RUN-01 retest in plan 06 if it were ever made.

## User Setup Required

None. The plan installs nothing and changes no configuration.

## Next Phase Readiness

- Plan 02-04 (GUI edit) can proceed. Brian opens `~/runtests/runtest-utf8.txt`, types `runtest line two` on a new line, and saves. Then `shasum -a 256 -c ~/runtests/runtest-utf8.expected.sha256` decides RUN-02.
- Plan 06 compares the compat.h baselines in `BUILD-05.compat-baseline.txt` against any change.
- `REQUIREMENTS.md`, `STATE.md`, and `ROADMAP.md` were not edited here. The orchestrator owns those writes.

## Self-Check: PASSED

Checked before the SUMMARY commit:
- `shasum -a 256 -c ~/runtests/runtest-utf8.initial.sha256` returned OK. The initial and expected hashes differ. The non-ASCII byte count is 0.
- `~/runtests/lists/build05-sites.tsv` has 10 lines (header plus a to i) and contains `RUN-01 (unresolved)`.
- `ODBEditorSuite.o` exists in the build tree.
- `compat.h` lines 18-21 match the baseline `dfa67021...`.
- No tracked repo file changed: `git status --porcelain -- . ':(exclude).planning'` is empty.
- Plan commit ledger: 0 commits before this SUMMARY, base `450702221d6f4eb918267ef7bfea0228503ef3a4`.

---
*Phase: 02-launch-and-run-verification*
*Completed: 2026-10-19*
