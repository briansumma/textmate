# Phase 2: Launch and Run Verification - Context

**Gathered:** 2026-10-19
**Status:** Ready for planning

<domain>
## Phase Boundary

The built `TextMate.app` from Phase 1 launches on the current macOS. It keeps its editing, bundle, HTML output, and preference features. The run results form the baseline that Phase 3 is checked against. BUILD-05 migrates deprecated APIs only where a RUN test fails.

Requirements in scope: RUN-01, RUN-02, RUN-03, RUN-04, RUN-05, BUILD-05.

</domain>

<decisions>
## Implementation Decisions

### GUI Checks (RUN-01, RUN-02, RUN-04, RUN-05)
- Brian performs the GUI steps. Claude runs the shell checks and records each result.
- Test files go in `~/runtests/`, outside the repo.
- Results go in `.planning/phases/02-launch-and-run-verification/02-RUN-LOG.md`, one row for each requirement ID.

### Bundle and Load Checks (RUN-03, RUN-04)
- Each bundle runs the first command that it declares. Claude lists that command for each bundle before the run.
- A missing interpreter records an environment failure. Do not install software.
- Each managed-only bundle loads one grammar, theme, or snippet file in a new document. Claude lists one file for each bundle.
- RUN-04 uses the first `HTML.tmbundle` command that produces HTML output.

### Failures and Commits (BUILD-05)
- A failed RUN test: record the failure and its log first. Migrate only the failing call site. Run the test again.
- Commit one change for each requirement ID. Each commit message names the ID.
- The run log keeps the exit status and the first and last 20 lines of output. The full log goes to `~/runtests/logs/`.
- A test that still fails after a fix stops the run. Phase 2 stays open.

### Claude's Discretion
- None. All grey areas were accepted as recommended.

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- The built app: `find ~/build/textmate -name TextMate.app -print -quit`. Phase 1 produced it at `~/build/textmate/release/Applications/TextMate/TextMate.app`.
- Test pass rules: `.planning/REQUIREMENTS.md`, sections RUN-01 to RUN-05 and BUILD-05.
- Codebase maps: `.planning/codebase/` (CONCERNS.md, TESTING.md, STRUCTURE.md).

### Established Patterns
- Phase 1 recorded each result in a summary file and kept commits to one requirement ID each.

### Integration Points
- BUILD-05 call sites, as listed in REQUIREMENTS.md:
  - `Frameworks/HTMLOutput/src/OakHTMLOutputView.mm:199` (NSURLConnection, RUN-04)
  - `Frameworks/SoftwareUpdate/src/SoftwareUpdate.mm:557` (NSTask.launchPath, RUN-01)
  - `Shared/include/oak/compat.h:8-16` (Gestalt, left unchanged by decision)
  - `Frameworks/io/src/resource.cc:34` (Get1Resource, left unchanged by decision)
  - `Frameworks/TMFileReference/src/TMFileReference.mm:257-260` (GetIconRef, left unchanged by decision)
  - `Frameworks/OakCommand/src/OakCommand.mm:679` (WebView registration, RUN-01)
  - `Applications/TextMate/src/ODBEditorSuite.mm` (not yet checked)
- Compile-only sites, no change planned: `Shared/include/oak/compat.h:18-21`, `Frameworks/Preferences/src/TerminalPreferences.mm:50`, `Frameworks/authorization/src/server.cc:26`.

</code_context>

<specifics>
## Specific Ideas

- The RUN-01 launch dialog does not fail the test. Record its text in the run log.
- RUN-03 counts 60 of 60 managed-only bundles. Missing interpreters can make the run fail. Phase 2 cannot pass until each cause is recorded.

</specifics>

<deferred>
## Deferred Ideas

- Migrating `Gestalt`, `Get1Resource`, `GetIconRef`, WebView registration, and the Apple Events and ODB sites. These stay unchanged unless a RUN test fails. See REQUIREMENTS.md decisions 8 to 12.
- Scripted bundle checks in place of GUI loads. Not chosen. It would change the RUN-03 pass rule.

</deferred>
