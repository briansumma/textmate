# TextMate (Personal Fork)

## What This Is

A personal, local-only fork of TextMate 2.0.23, the macOS text editor by MacroMates. The goal is to keep it running as a native Apple Silicon app on the current macOS for one user. The upstream project has had no commits since 2021-10-12.

## Core Value

TextMate builds for Apple Silicon, launches, opens and saves files, and runs its bundle commands on the current macOS. Other working features keep working.

## Requirements

### Validated

- (None yet. RUN-02 and RUN-03 verify the inherited editing and bundle behavior on this machine.)

### Active

- [ ] **BUILD-01**: Build `TextMate.app` for arm64 on the current macOS with the current Xcode from a clean checkout.
- [ ] **BUILD-02**: `configure` finds Homebrew dependencies under `/opt/homebrew` without hand-editing `local.rave`.
- [ ] **BUILD-03**: Submodules initialize with one documented command.
- [ ] **BUILD-04**: Deployment target is macOS 26.7.1 only. The 10.12 floor is removed. The `-std=c++2a` flag is unchanged unless the build fails.
- [ ] **BUILD-05**: Deprecated APIs are migrated only where the build or a RUN test fails on the current macOS. A migration must not change behavior.
- [ ] **RUN-01**: The built app launches on the current macOS.
- [ ] **RUN-02**: Opening, editing, and saving a file work.
- [ ] **RUN-03**: Each bundle in the bundle set runs its commands. Each managed-only bundle loads.
- [ ] **RUN-04**: A bundle command whose output is HTML shows that output in the HTML output window.
- [ ] **RUN-05**: The About window and the Preferences window open and show content. A changed preference persists.
- [ ] **UPD-01**: The app contacts no `api.textmate.org` host for updates. The app update check, the bundle index poll and its checkbox, the bundle updater, the `bl` update path, and the `can_reach_host` probe are removed.
- [ ] **PRIV-01**: The crash reporter uploads no report, and the "Submit to MacroMates" preference is removed.
- [ ] **SIGN-01**: The built app is ad-hoc signed and runs locally without notarization.

### Out of Scope

- New features. The user does not want any at this time.
- Intel (x86_64) builds. The target Mac is Apple Silicon only.
- Notarization and Developer ID signing. Personal use needs ad-hoc signing only.
- Public distribution or an upstream release.
- Remote hosting of the fork. The repository stays local for now.
- Security hardening from the concerns audit (rmate remote mode, privileged helper, ATS). Reason: personal machine. Revisit if the fork leaves this Mac.
- `disable-library-validation` entitlement. Reason: stays unchanged, same as security hardening.
- Any `CONCERNS.md` item that no requirement names. Reason: in scope only if a requirement or an Open decision names it.
- Replacing legacy WebView or ODB code. Reason: change only if a BUILD-05 test requires it.

## Context

- Upstream repository: `github.com/textmate/textmate`. Newest release is 2.0.23, dated 2021-10-12. The last upstream commit is `346b52b1`, by Allan Odgaard, dated 2021-10-12.
- Local branch `develop` is the upstream tip plus one docs commit, dated 2026-10-19.
- Build environment: arm64 Mac, macOS 26.7.1, Xcode 27.1, Homebrew at `/opt/homebrew`.
- Codebase map: `.planning/codebase/` (complete, 2026-10-19).
- Known build blockers, from the static audit. A build has not run yet:
  - `configure` looks only in `/usr/local`.
  - Submodules are uninitialized in this checkout.
  - `default.rave` hardcodes the deployment target `10.12`.
  - Deprecated APIs are in use. These include `WebView`, `NSTask.launchPath`, `NSURLConnection`, and `AuthorizationExecuteWithPrivileges`. SDK 27.0 declares them deprecated, not removed, so they do not block the build. BUILD-05 governs them.
  - The `-std=c++2a` flag is not a blocker. A probe compiled with it on this toolchain. It stays unless the build fails (BUILD-04).
- `local-orig.rave` already contains arm64 and x86_64 target logic. Use it as a reference.

## Constraints

- **Platform:** Apple Silicon (arm64) only. Reason: the target Mac is arm64, and Intel support ends in the next macOS release.
- **OS floor:** the current macOS only. Reason: personal use on one machine.
- **Feature freeze:** no new features. Reason: user request.
- **Signing:** ad-hoc only. Reason: personal use, no distribution.
- **Hosting:** local only. Reason: no hosting decision yet.
- **Dependencies:** Boost, Cap'n Proto, google-sparsehash, MultiMarkdown, Ragel, and ninja, installed through Homebrew. Versions are not pinned yet. BUILD-02 requires them recorded in the README Building section.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Target the current macOS only | Personal use on one machine; drops the 10.12 floor | — Pending |
| Remove the online update check | `api.textmate.org` may no longer respond; removal adds no feature | — Pending |
| Ad-hoc signing only | Personal use; no notarization needed | — Pending |
| Local-only repository | No hosting decision yet | — Pending |
| Skip domain research; use the codebase map | The map is complete, and the task is maintenance, not a new domain | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? Move to Out of Scope with reason.
2. Requirements validated? Move to Validated with phase reference.
3. New requirements emerged? Add to Active.
4. Decisions to log? Add to Key Decisions.
5. "What This Is" still accurate? Update if drifted.

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections.
2. Core Value check: still the right priority?
3. Audit Out of Scope: reasons still valid?
4. Update Context with current state.

---
*Last updated: 2026-10-19 after red-team revision of REQUIREMENTS.md*
