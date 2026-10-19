---
phase: 02-launch-and-run-verification
plan: 01
subsystem: testing
tags: [runtests, launch, process-check, crash-report, tracer]

# Dependency graph
requires:
  - phase: 01-apple-silicon-build
    provides: built TextMate.app at /Users/bfs/build/textmate/release/Applications/TextMate/TextMate.app
provides:
  - ~/runtests harness (snapshot.sh, launch-check.sh) outside the repository
  - pre-run01 bundle and defaults snapshots, the baseline for RUN-03 and RUN-05
  - RUN-01 row recorded as PASS (exit 0, with a documented launch-step caveat)
affects: [02-02, 02-03, 02-04, 02-05, 02-06]

# Actuals (#2632): chars/4 over the harness files actually changed (outside the repo).
# No tracked repo file changed in this plan.
actuals:
  tokens: 1830
  tasks: 2
  commits: 0

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Process check matches the command path of each TextMate PID to the built app, never a bare PID"
    - "Marker file touched before open, so find -newer reports only crash reports from this launch"
    - "check mode is read-only: it re-verifies a running app and never launches or kills"

key-files:
  created:
    - ~/runtests/bin/snapshot.sh
    - ~/runtests/bin/launch-check.sh
    - ~/runtests/snapshots/pre-run01.bundles.txt
    - ~/runtests/snapshots/pre-run01.defaults.plist
    - ~/runtests/run01.marker
    - ~/runtests/logs/RUN-01-launch.log
    - ~/runtests/logs/RUN-01.row
  modified: []

key-decisions:
  - "RUN-01 recorded as PASS with exit 0 on Brian's decision (option a): check-mode evidence stands in for the killed launch step"
  - "run01 was not re-run, because re-running while the built app runs would refuse with exit 2 and truncate the transcript"
  - "The installed /Applications/TextMate.app was quit by Brian with Cmd-Q before the launch. The executor did not kill any process"

patterns-established:
  - "Harness lives in ~/runtests, outside the repository; no test file, script, or log enters the repo"
  - "Launch exit is recorded only from the launch step; a killed launch is recorded as its real exit status with a note"

requirements-completed: [RUN-01]

# Coverage metadata (#1602)
coverage:
  - id: D1
    description: "The built app launches and runs. The process command path matches the built app, and no new TextMate crash report appears."
    requirement: "RUN-01"
    verification:
      - kind: other
        ref: "~/runtests/bin/launch-check.sh <built app> check (rc 0, run twice: before and after window confirmation)"
        status: pass
    human_judgment: false
  - id: D2
    description: "The main window of the built app is visible, with no launch dialog."
    requirement: "RUN-01"
    verification: []
    human_judgment: true
    rationale: "Visual observation by Brian at the Mac. Brian confirmed the window is visible and no dialog appears."

# Metrics
duration: not recorded (plan paused at two checkpoints)
completed: 2026-10-19
status: complete
---

# Phase 2 Plan 01: RUN-01 Tracer Launch Summary

**Built TextMate launches as PID 46275 with a command path matching the built app, no new crash report, and the main window confirmed by Brian**

## Performance

- **Duration:** not recorded (paused at two checkpoints)
- **Completed:** 2026-10-19
- **Tasks:** 2 (tracer launch, window confirmation)
- **Files modified in repo:** 0 tracked files. This SUMMARY.md is the only repo file added.

## Accomplishments

- Built `~/runtests` harness with `snapshot.sh` and `launch-check.sh`. Both are shellcheck-clean with `--exclude=SC1090`.
- Took the pre-run snapshot `pre-run01`: 75 bundle entries (user and managed) and the `com.macromates.TextMate` defaults export. RUN-03 and RUN-05 compare against it.
- Verified the built app is the running process. PID 46275 has the command path `/Users/bfs/build/textmate/release/Applications/TextMate/TextMate.app/Contents/MacOS/TextMate`. A bare PID was not accepted as proof.
- Verified no new `TextMate*` crash report appeared after `~/runtests/run01.marker`.
- Brian confirmed the main window of the built app is visible. No dialog appeared. The app stayed running.
- Wrote `~/runtests/logs/RUN-01.row` with one tab-separated line: `RUN-01`, `PASS`, `0`, log path, and note.

## Task Commits

No per-task commits. The plan changes no tracked repo files. The harness, snapshots, logs, and row are under `~/runtests`, which is outside the repository.

**Plan metadata:** the commit that adds this SUMMARY.md (`docs(02-01)`). The commit ledger shows 0 plan commits before it, which is legitimate for a plan with no code changes.

## Files Created/Modified

Outside the repository (`~/runtests`):
- `~/runtests/bin/snapshot.sh` - writes `LABEL.bundles.txt` (scope, name, mtime, sorted with `LC_ALL=C`) and `LABEL.defaults.plist`
- `~/runtests/bin/launch-check.sh` - `run01` and `relaunch` launch and verify; `check` is read-only
- `~/runtests/snapshots/pre-run01.bundles.txt` and `pre-run01.defaults.plist` - the pre-launch baseline
- `~/runtests/run01.marker` - touched before `open`
- `~/runtests/logs/RUN-01-launch.log` - transcript of the run01 launch step. It ends at `WAITING: 30s`, because the step was killed (see Issues Encountered).
- `~/runtests/logs/RUN-01.row` - the RUN-01 row, `PASS`

Inside the repository: this file only.

## Decisions Made

- **RUN-01 is PASS with exit 0 on Brian's decision (option a).** The run01 launch step was killed by the tool harness with exit 137, so its own exit 0 was never produced. Brian chose to accept check mode as the evidence for steps (e) and (f). The row note documents this, so the substitution stays visible.
- **run01 was not re-run.** While the built app runs, a second run01 refuses with exit 2 and truncates `RUN-01-launch.log`. Re-running would have destroyed the only transcript.
- **No process was killed.** The installed `/Applications/TextMate.app` (PID 94680) was quit by Brian with Cmd-Q, which was the precondition. The built app (PID 46275) was left running for the window check.

## Deviations from Plan

### Auto-fixed Issues

None under the deviation rules. No code changed, and no bug or blocker needed a fix.

### User-directed substitution

**1. [User decision] Launch exit 0 recorded from check mode**
- **Found during:** Task 1 (tracer launch)
- **Issue:** The run01 launch step was killed by the harness during its 30-second wait. Its steps (e) and (f) and its exit status were never recorded. The plan's tracer done-criterion says `launch-check.sh run01` exits 0.
- **Fix:** Brian chose option (a). The row records exit 0 with a note. The evidence is a read-only check-mode run, executed twice: once right after the launch and once after Brian confirmed the window. Both returned exit 0.
- **Files modified:** `~/runtests/logs/RUN-01.row`
- **Verification:** The tracer verify command returned exit 0. Check mode reports PID 46275 matching the built app's command path, and no new crash report.
- **Committed in:** this SUMMARY.md commit.

**Total deviations:** 0 under the deviation rules, 1 user-directed substitution.
**Impact on plan:** The window and process evidence is real. The launch step's own exit status is not, and the row says so.

## Issues Encountered

- **Harness killed the run01 launch.** The Bash tool returned exit 137 and reported that the output file had been replaced during startup cleanup. The launch itself succeeded: the app started and logged `WAITING: 30s`. The transcript therefore has no steps (e) and (f), no `EXIT STATUS` line, and no head/tail output. The check-mode results above are the complete record.
- **Check mode writes no transcript.** Its results are recorded here and in the row note, not in a log file. This matches the plan's rule that check mode writes no file.

## Checkpoints

1. **Precondition (Task 1):** no TextMate process before launch. Not met at first (installed copy running). Resolved when Brian quit it with Cmd-Q and the coordinator confirmed `pgrep -x TextMate` printed nothing.
2. **Window confirmation (Task 2):** Brian confirmed the main window of the built app is visible, with no dialog. Recorded as `window yes, dialog none`.

**Launch dialog text:** none.

## User Setup Required

None. The plan installs nothing and changes no configuration.

## Next Phase Readiness

- The `~/runtests` harness and the `pre-run01` snapshots are ready for plans 02-02 through 02-05.
- The built app is still running as PID 46275 for plan 02-04. Plan 02-04 needs it running, and its open windows are the ones under test.
- The `RUN-01` requirement is marked complete in this plan's frontmatter. `REQUIREMENTS.md` was not edited here, and `STATE.md` and `ROADMAP.md` are left to the orchestrator.

## Self-Check: PASSED

Checked after the row was written:
- `~/runtests/bin/snapshot.sh` and `~/runtests/bin/launch-check.sh` exist and are executable. Shellcheck exit 0.
- `pre-run01.bundles.txt` (75 lines) and `pre-run01.defaults.plist` exist and are non-empty.
- `~/runtests/logs/RUN-01.row` holds one 5-field line: `RUN-01`, `PASS`, `0`, log path, note.
- Tracer verify command (check mode, the row's exit field, non-empty log) returned exit 0.
- Repo tree outside `.planning` is clean before this SUMMARY commit. Plan commit ledger: 0.

---
*Phase: 02-launch-and-run-verification*
*Completed: 2026-10-19*
