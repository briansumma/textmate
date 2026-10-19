---
phase: 02-launch-and-run-verification
plan: 02
subsystem: testing
tags: [runtests, run-03, bundles, headless, inventory, decision-record]

# Dependency graph
requires:
  - phase: 02-launch-and-run-verification
    provides: ~/runtests harness and RUN-01 row (plan 02-01)
provides:
  - bundle-set (9 rows) and managed-only (60 rows, 5 NOFILE) inventories under ~/runtests/lists
  - run-command.sh headless runner (shellcheck clean)
  - RUN-03 headless table with 9 rows, each with exit status, classification, and log path
  - decisions-run03.txt recording Brian's answers for plan 05
affects: [02-04, 02-05, 02-06]

# Actuals (#2632): chars/4 over the harness files actually changed (outside the repo).
actuals:
  tokens: 6917
  tasks: 3
  commits: 0

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Headless run uses env -i with the plan's fixed PATH and a 120-second alarm; a headless exit status is a note, the GUI check decides"
    - "First command = first merged Commands file in LC_ALL=C order (managed base plus user layer, same-uuid user file replaces base)"

key-files:
  created:
    - ~/runtests/bin/run-command.sh
    - ~/runtests/lists/bundle-set.tsv
    - ~/runtests/lists/managed-only.tsv
    - ~/runtests/lists/managed-excluded.txt
    - ~/runtests/lists/run03-commands.txt
    - ~/runtests/lists/run03-bodies/ (8 exact first-command bodies)
    - ~/runtests/lists/decisions-run03.txt
    - ~/runtests/logs/RUN-03.headless.tsv
    - ~/runtests/logs/RUN-03-<bundle>.log (8 logs)
  modified: []

key-decisions:
  - "Decision A approved as proposed (Brian, via coordinator): the eight first commands, including C's managed-base command. Themes is NO-COMMAND."
  - "Decision B1 applied: the five NOFILE bundles (Bundle Support, Markdown (GitHub) Font Settings, Math, SCM, SCM Diff Gutter) are FAIL with the cause 'no grammar, theme, or snippet file'. Phase 2 stays OPEN for SC3."
  - "The fixed PATH from the plan is kept. Every headless result is labeled headless-only. The GUI decides each verdict."
  - "Avian, Shell Script, and Python run later in the GUI on a throwaway scratch file in ~/runtests/work, never on Brian's documents."

patterns-established:
  - "Headless class (HEADLESS-PASS, ENV-FAIL, HEADLESS-FAIL, NO-COMMAND) is a note only"
  - "Decision record written before the first command runs (decisions-run03.txt)"

requirements-completed: []

# Coverage metadata (#1602)
coverage:
  - id: D1
    description: "Bundle-set (9 rows), managed-only (60 rows, 7 excluded, 5 NOFILE), and the first-command choice for each bundle are built from the filesystem and printed."
    requirement: "RUN-03"
    verification:
      - kind: other
        ref: "Task 1 verify command: bundle-set 9 rows, managed-excluded 7, managed-only 60, NOFILE 5, support path present, shellcheck clean on run-command.sh (rc 0)"
        status: pass
    human_judgment: false
  - id: D2
    description: "Eight approved first commands ran headlessly, one run each, with exit status, classification, and log path recorded; Themes recorded NO-COMMAND; no managed-only command ran."
    requirement: "RUN-03"
    verification:
      - kind: other
        ref: "Task 3 verify command: RUN-03.headless.tsv has 9 rows, 5 fields each, numeric status or '-' on NO-COMMAND (rc 0)"
        status: pass
    human_judgment: false
  - id: D3
    description: "Brian's decisions (Decision A, B1, GUI decides, PATH kept, GUI-risk commands on a scratch file) recorded in decisions-run03.txt before any command ran."
    requirement: "RUN-03"
    verification: []
    human_judgment: true
    rationale: "Recorded from the coordinator's relay of Brian's answers. The coordinator message did not restate BUILD-05 A1 or B-confirm; those are recorded from the plan text and flagged for Brian."

# Metrics
duration: not recorded (plan paused at the Task 2 checkpoint)
completed: 2026-10-19
status: complete
---

# Phase 2 Plan 02: RUN-03 Bundle-Set Headless Run Summary

**Eight approved first commands ran headlessly: 5 ENV-FAIL (ruby18 and python are off the fixed PATH), 1 HEADLESS-FAIL (Perl), 2 HEADLESS-PASS (HTML exit 0 with tool-missing output, and Shell Script 2). Themes is NO-COMMAND. No managed-only command ran. Every verdict is left to the GUI check in plan 05.**

## Performance

- **Duration:** not recorded (paused at the Task 2 checkpoint)
- **Completed:** 2026-10-19
- **Tasks:** 3 of 3 (Task 2 resolved by Brian's answers, relayed by the coordinator)
- **Files modified in repo:** 0 tracked files. This SUMMARY.md is the only repo file added.

## Accomplishments

- Built the bundle-set inventory: 9 rows (8 user bundles plus Avian from the built app). Each row has a folder, the first command file, and the command kind.
- Built the managed-only inventory: 67 managed folders, 7 excluded by exact name (C, HTML, Perl, Python, Shell Script, Text, Themes), 60 managed-only. 55 have a load file. 5 are NOFILE (Bundle Support, Markdown (GitHub) Font Settings, Math, SCM, SCM Diff Gutter).
- Chose each first command by the stated rule. Printed the choices and the exact bodies (`~/runtests/lists/run03-bodies/`).
- Confirmed the RUN-04 candidate: About Persistent Includes.tmCommand, the expected result.
- Wrote `run-command.sh` (fixed PATH, env -i, 120-second alarm, stdin from /dev/null, TM_SUPPORT_PATH set to the measured folder). Shellcheck clean.
- Wrote `decisions-run03.txt` before any command ran.
- Ran the eight approved commands headlessly, one run each. Recorded nine rows in `RUN-03.headless.tsv`. Read the first and last 20 lines of each log.

## Headless Table (headless-only; GUI decides)

| Bundle | Command file | Exit | Class | Log |
| ------ | ------------ | ---- | ----- | --- |
| Avian | Compress Selected Items.tmCommand | 127 | ENV-FAIL | RUN-03-Avian.log |
| C | C & C++ Library Completions.tmCommand (managed base) | 127 | ENV-FAIL | RUN-03-C.log |
| HTML | About Persistent Includes.tmCommand (managed base) | 0 | HEADLESS-PASS | RUN-03-HTML.log |
| Perl | Check With Perl-Critic.tmCommand (managed base) | 1 | HEADLESS-FAIL | RUN-03-Perl.log |
| Python | Check Python Syntax.plist (managed base) | 127 | ENV-FAIL | RUN-03-Python.log |
| Shell Script | Execute Current File in Window.plist (managed base) | 127 | ENV-FAIL | RUN-03-Shell_Script.log |
| Shell Script 2 | New Shell Script.tmCommand (user) | 0 | HEADLESS-PASS | RUN-03-Shell_Script_2.log |
| Text | Convert Character : Selection to Char.tmCommand (managed base) | 127 | ENV-FAIL | RUN-03-Text.log |
| Themes | none | - | NO-COMMAND | - |

Log detail:
- **Avian, C, Shell Script, Text:** `env: ruby18: No such file or directory`. The ruby18 shim exists at `Bundle Support.tmbundle/Support/shared/bin/ruby18`, but that folder is not on the fixed PATH.
- **HTML:** exit 0, but the output contains `ruby18: command not found` (webpreview.sh) and `Markdown.pl` / `SmartyPants.pl: command not found`. The exit status is not evidence that the HTML was produced. The GUI check decides.
- **Perl:** `LoadError: cannot load such file -- .../Bundles/Perl.tmbundle/Support/perlcritic.rb`. The managed-base command resolves TM_BUNDLE_SUPPORT to the user Perl copy, which has no Support folder.
- **Python:** `python: command not found`. `~/.local/bin/python` is not on the fixed PATH.
- **Shell Script 2:** exit 0, and the log holds the 179-line generated script template (stdout only, no document touched).

## Task Commits

No per-task commits. The plan changes no tracked repo files. The harness, lists, logs, and decision record are under `~/runtests`, outside the repository.

**Plan metadata:** the commit that adds this SUMMARY.md (`docs(02-02)`). The commit ledger (`gsd-plan-head-before-02-02`) has base `de98eccb` and 0 plan commits before it. Zero is legitimate here because the plan made no code changes.

## Files Created/Modified

Outside the repository (`~/runtests`):
- `bin/run-command.sh` - headless runner (fixed PATH, env -i, 120-second alarm)
- `lists/bundle-set.tsv` - 9 rows: name, folder used, first command file, kind
- `lists/managed-only.tsv` - 60 rows: name, load file (5 NOFILE)
- `lists/managed-excluded.txt` - 7 names
- `lists/run03-commands.txt` - 9 rows, plus a `#` RUN-04 candidate comment (skipped by the run)
- `lists/run03-bodies/*.txt` - exact bodies of the eight first commands (sha256 prefixes were printed at write time, not saved)
- `lists/decisions-run03.txt` - Decision A, B1, GUI decides, PATH kept, GUI-risk scratch rule, A1 and B-confirm as recorded in the plan
- `logs/RUN-03.headless.tsv` - 9 rows
- `logs/RUN-03-<bundle>.log` - 8 logs (one per command run)
- `work/cmd-<bundle>.sh` - the 8 command bodies as run (chmod 700)

Inside the repository: this file only.

## Decisions Made

- **Decision A approved as proposed (Brian, via coordinator).** The eight first commands are used as listed. C's first command comes from the managed base copy, because the user C bundle has no Commands folder. Themes is NO-COMMAND.
- **Decision B1 applied.** The five NOFILE bundles are FAIL with the cause "no grammar, theme, or snippet file". Phase 2 stays OPEN for SC3. REQUIREMENTS.md and ROADMAP.md are not amended.
- **PATH kept.** The plan's fixed PATH is not changed. Headless results that depend on a tool outside it are labeled headless-only.
- **GUI-risk commands move to a scratch file.** Avian, Shell Script, and Python run later in the GUI on a throwaway scratch file in `~/runtests/work`, not on Brian's documents.

## Deviations from Plan

### Auto-fixed Issues

None under the deviation rules. No code changed, and no bug or blocker needed a fix.

### Documented Deviations

**1. [Documented choice] `requirements-completed` left empty.**
- **Found during:** Final frontmatter (the workflow asks for a verbatim copy of the plan's `requirements`).
- **Issue:** The plan declares RUN-03. This plan does not complete RUN-03. The GUI verdicts (plan 05) and the 60 managed-only loads are still open.
- **Fix:** The field is empty, so no tool marks RUN-03 complete. The status is explained in this summary.
- **Files modified:** this SUMMARY.md only.
- **Commit:** the SUMMARY commit below.

**Total deviations:** 0 under the deviation rules, 1 documented choice.
**Impact on plan:** RUN-03 stays open until plan 05 records the GUI verdicts and the managed-only loads are checked.

## Issues Encountered

- **Pre-commit guard check.** A first check ran in a shell without `gsd_run` on PATH and reported the branch-protection query as unavailable, which fell back to the five-name list. That list includes `develop`. Rerunning through the real tool (`node gsd-tools.cjs git.base-branch --is-protected develop`) returned `false`, because the repo's default branch is `master`. The guard passes, and no config was changed.
- **Coordinator message did not restate BUILD-05 A1 or B-confirm.** Both are recorded in `decisions-run03.txt` from the plan text, and the record says so. Brian should confirm them: site a (`OakHTMLOutputView.mm:199`) is not covered by A1 and stays open.
- **Headless results that look wrong.** HTML exits 0 with error output, and Perl fails on a path the managed-base layering does not supply. Both are recorded as they ran. Neither is a verdict.

## Known Stubs

None. No code was written.

## Threat Flags

None. No new network endpoint, auth path, or file access beyond the plan's threat model. The approved commands ran with stdin from /dev/null, an explicit PATH, a 120-second alarm, and a working directory in `~/runtests/work`. None received a document path.

## Self-Check

Checked after the run and before this summary was written.

- Task 3 verify command: PASS (9 rows, 5 fields each, numeric status or `-` on NO-COMMAND).
- `decisions-run03.txt`, `run-command.sh`, and `RUN-03.headless.tsv` exist and are non-empty.
- No managed-only command ran. The only log that names a managed path is `RUN-03-HTML.log`, and that is the HTML command's own path inside its error output.
- Repo gate: `git status --porcelain -- . ':(exclude).planning'` is empty.
- Ledger: base `de98eccb`, 0 commits measured.

## Self-Check: PASSED

## Next Phase Readiness

- Plan 05 takes the GUI verdicts for each bundle-set command. The GUI-risk commands (Avian, Shell Script, Python) run on a scratch file in `~/runtests/work`.
- Open items for Brian: confirm BUILD-05 A1 and B-confirm as recorded; decide site a (`OakHTMLOutputView.mm:199`).
- The five NOFILE bundles are FAIL under B1. Record them in the run log in plan 05.
- Plan 06 runs after all GUI sessions.

---
*Phase: 02-launch-and-run-verification*
*Completed: 2026-10-19*
