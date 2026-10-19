---
phase: 02-launch-and-run-verification
plan: 04
subsystem: testing
tags: [runtests, run-02, run-04, save, html-output, gui-checkpoint, build-05]

# Dependency graph
requires:
  - phase: 02-launch-and-run-verification
    provides: running built app PID 46275 (plan 02-01), RUN-02 fixture and expected hash (plan 02-03), RUN-04 command candidate (plan 02-02)
provides:
  - RUN-02.row recorded PASS from the byte check (sha256 0524aa99...), with its log
  - RUN-04.row recorded PASS for the rendering criterion, from Brian's report, with its note
  - BUILD-05 site a recorded as an open item for Brian in the defect ledger
affects: [02-05, 02-06]

# Actuals (#2632): chars/4 over the harness files actually changed (outside the repo).
actuals:
  tokens: 600
  tasks: 4
  commits: 0

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "RUN-02 pass rule is byte equality via shasum -a 256 -c on the file Brian saved in place"
    - "GUI observations are recorded as reported, with the source named, and never inferred by Claude"

key-files:
  created:
    - ~/runtests/logs/RUN-02-save.log
    - ~/runtests/logs/RUN-02.row
    - ~/runtests/logs/RUN-04-gui.txt
    - ~/runtests/logs/RUN-04.row
  modified:
    - .planning/WINDOWS.md (new, untracked before this plan; one unrun-verify entry for BUILD-05 site a)

key-decisions:
  - "RUN-02 PASS: shasum -a 256 -c exited 0 on the saved file, the line count is 2, and the file is 34 bytes. The file was not edited by hand."
  - "RUN-04 PASS for the rendering criterion only: the window showed rendered content, not a blank window. Site a is not exercised and stays an open item for Brian."
  - "No tracked repo file was changed. The only repo file added is this SUMMARY.md, plus the defect ledger entry."

patterns-established:
  - "A GUI report that reaches Claude through the coordinator is recorded as the coordinator's relay of Brian's reply, with the source named in the file."

requirements-completed: [RUN-02, RUN-04]

# Coverage metadata (#1602)
coverage:
  - id: D1
    description: "RUN-02: the saved runtest-utf8.txt matches the expected sha256 byte for byte, and has two lines."
    requirement: "RUN-02"
    verification:
      - kind: other
        ref: "shasum -a 256 -c ~/runtests/runtest-utf8.expected.sha256 (exit 0, runtest-utf8.txt: OK); wc -l = 2; log ~/runtests/logs/RUN-02-save.log"
        status: pass
    human_judgment: true
    rationale: "The edit, save, and reopen were Brian's GUI steps. Brian reported the reopened window shows two lines, and the shell checked the bytes."
  - id: D2
    description: "RUN-04: the HTML output window of the first HTML-output command opens and shows rendered content, not a blank window."
    requirement: "RUN-04"
    verification:
      - kind: other
        ref: "~/runtests/logs/RUN-04-gui.txt (Brian's report, relayed by the coordinator); ~/runtests/logs/RUN-04.row"
        status: pass
    human_judgment: true
    rationale: "Visual observation by Brian at the Mac. The rendering criterion is his report. Claude did not inspect the window."
  - id: D3
    description: "BUILD-05 site a (OakHTMLOutputView.mm:199) is exercised by a link click in the RUN-04 output."
    requirement: "BUILD-05"
    verification: []
    human_judgment: true
    rationale: "Not exercised. No link was seen in the captured area, and the page was not fully scrolled. Open item for Brian."

# Metrics
duration: not recorded
completed: 2026-10-19
status: complete
---

# Phase 2 Plan 04: RUN-02 Save and RUN-04 HTML Output Summary

**RUN-02 PASS by byte hash (sha256 0524aa99..., shasum exit 0, two lines). RUN-04 PASS for rendering: the About Persistent Includes window showed rendered content, not blank. Site a stays open because no link was seen in the captured area.**

## Performance

- **Duration:** not recorded (paused at two Brian GUI checkpoints)
- **Completed:** 2026-10-19
- **Tasks:** 4 of 4 (Tasks 1 and 3 were Brian's GUI steps; Tasks 2 and 4 were auto)
- **Files modified in repo:** 0 tracked files. This SUMMARY.md and the new `.planning/WINDOWS.md` ledger entry are the only repo files touched.

## Accomplishments

- **Task 1 (Brian, GUI):** opened `runtest-utf8.txt` in the built app, typed `runtest line two` on line 2, pressed Return once, saved with Cmd-S, and reopened the file. The reopened window shows two lines.
- **Task 2 (auto):** `shasum -a 256 -c runtest-utf8.expected.sha256` printed `runtest-utf8.txt: OK` and exited 0. The actual hash is `0524aa99317b263e747d1b4ac10e7236d4b59eb609c2a4464def61301ab00530`. The line count is 2, and the file is 34 bytes. Wrote `RUN-02-save.log` and `RUN-02.row` (PASS).
- **Task 3 (Brian, GUI):** ran Bundles > HTML > Includes > Help: Persistent Includes. The window titled "About Persistent Includes" shows rendered output and is not blank. No hyperlink is visible in the captured area. The page continues below the captured area, and it was not scrolled.
- **Task 4 (auto):** wrote `RUN-04-gui.txt` with Brian's reply and a screenshot description, and `RUN-04.row` (PASS, GUI). Verify passed on both.
- Recorded BUILD-05 site a as an open item in the defect ledger.
- The built app (PID 46275) was left running. No TextMate process was started, quit, or killed.

## Task Commits

No per-task commits. The plan changes no tracked repo files. The SUMMARY commit is the only commit from this plan.

**Plan metadata:** the commit that adds this SUMMARY.md (`docs(02-04)`). The commit ledger (`gsd-plan-head-before-02-04`) has base `b073ef82` and 0 plan commits before it. Zero is legitimate here, because the plan made no code changes.

## Files Created/Modified

Outside the repository (`~/runtests`):
- `~/runtests/logs/RUN-02-save.log` - shasum output with exit status, actual hash, line count, byte count, and the reopen report
- `~/runtests/logs/RUN-02.row` - `RUN-02`, `PASS`, `0`, log path, note
- `~/runtests/logs/RUN-04-gui.txt` - Brian's RUN-04 reply and the screenshot description
- `~/runtests/logs/RUN-04.row` - `RUN-04`, `PASS`, `GUI`, GUI report path, note

Inside the repository:
- `.planning/WINDOWS.md` - new ledger, one `unrun-verify` entry (id 1, open) for BUILD-05 site a at `Frameworks/HTMLOutput/src/OakHTMLOutputView.mm:199`
- this SUMMARY.md

## Decisions Made

- **RUN-02 is PASS.** The pass rule is byte equality on the saved file. `shasum -c` exited 0, and the line count is 2. Brian's reopen report also shows two lines. The file was not edited by hand.
- **RUN-04 is PASS for the rendering criterion.** The window opened and showed rendered content with code blocks, not a blank window. The PASS covers only that criterion.
- **Site a is not exercised.** No link was seen in the captured area, and the page was not fully scrolled. Site a stays an open item for Brian, as the plan states. The row note says so.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Wrong file path in the new defect ledger entry**
- **Found during:** Task 4, after the entry was written
- **Issue:** The `windows append` call used `Sources/OakHTMLOutputView.mm`. The file is at `Frameworks/HTMLOutput/src/OakHTMLOutputView.mm`.
- **Fix:** Edited the path in `.planning/WINDOWS.md` (table row and JSON entry). This is the ledger's only file, and it was created in this plan.
- **Files modified:** `.planning/WINDOWS.md`
- **Commit:** the SUMMARY commit below.

**Total deviations:** 1 auto-fixed (Rule 1).
**Impact on plan:** none on results. The ledger entry now points to the correct file.

## Issues Encountered

- **Brian's RUN-04 reply reached Claude through the coordinator, not directly.** `RUN-04-gui.txt` records the coordinator's relay of Brian's text, and the file names that source. The text is not a direct transcript of Brian's message.
- **The RUN-04 page was not fully scrolled.** The rendering criterion is met. Whether the rest of the page has a link is unknown.
- **Headless HTML run had errors.** The earlier headless run of this command printed `ruby18: command not found` and `Markdown.pl` errors. Brian's GUI report shows rendered content, so the errors did not blank the window. The headless exit status is still a note only.

## Known Stubs

None. No code was written.

## Threat Flags

None. No new network endpoint, auth path, or file access beyond the plan's threat model.

## Open Items for Brian

- **Site a** (`Frameworks/HTMLOutput/src/OakHTMLOutputView.mm:199`, NSURLConnection in the policy delegate). To exercise it, click a link on the About Persistent Includes page, or add a link case to RUN-04. Until then it stays open. It is recorded in `.planning/WINDOWS.md`.
- **Scroll the RUN-04 page** to see whether it has any link below the captured area. This is optional and does not change the rendering PASS.

## Self-Check

- `shasum -a 256 -c ~/runtests/runtest-utf8.expected.sha256` exited 0 (`runtest-utf8.txt: OK`).
- `RUN-02.row` and `RUN-04.row` each hold one 5-field line with status PASS. Both verify commands passed.
- `RUN-02-save.log`, `RUN-04-gui.txt` are non-empty. `RUN-04-gui.txt` contains the word "rendered".
- PID 46275 still runs from the built app path. Nothing was started, quit, or killed.
- Repo gate: `git status --porcelain -- . ':(exclude).planning'` is empty.
- Ledger: base `b073ef82`, 0 commits measured before this SUMMARY.

## Self-Check: PASSED

## Next Phase Readiness

- Plan 02-05 can use the running built app and the RUN-04 result. RUN-04's HTML window is still open.
- Plan 02-06 handles any migration after all GUI sessions. No RUN test failed here, so no migration is triggered by this plan.
- The `REQUIREMENTS.md`, `STATE.md`, and `ROADMAP.md` files were not edited here. The orchestrator owns those writes.

---
*Phase: 02-launch-and-run-verification*
*Completed: 2026-10-19*
