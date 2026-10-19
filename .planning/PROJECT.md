# TextMate (Personal Fork)

## What This Is

A personal, local-only fork of TextMate 2.0.23, the macOS text editor by MacroMates. The goal is to keep it running as a native Apple Silicon app on the current macOS for one user. The upstream project has had no commits since 2021-10-12.

## Core Value

TextMate builds for Apple Silicon, launches, opens and saves files, and runs its bundle commands on the current macOS. Everything else is secondary.

## Requirements

### Validated

- ✓ Upstream TextMate 2.0.23 editing, bundles, and commands are inherited from the existing code. Not yet re-verified on this machine (inferred from codebase map, 2026-10-19).

### Active

- [ ] Build `TextMate.app` for arm64 on the current macOS with the current Xcode from a clean checkout.
- [ ] `configure` finds Homebrew dependencies under `/opt/homebrew` without hand-editing `local.rave`.
- [ ] Submodules initialize with one documented command.
- [ ] The built app launches on the current macOS.
- [ ] Open, edit, and save a file work.
- [ ] Existing bundle commands run.
- [ ] The online update check is removed, and the app no longer contacts `api.textmate.org` for updates.
- [ ] The built app is ad-hoc signed and runs locally without notarization.
- [ ] Deprecated APIs are migrated only where the build or the run fails on the current macOS.
- [ ] Deployment target is the current macOS only. The 10.12 floor is removed.

### Out of Scope

- New features. The user does not want any at this time.
- Intel (x86_64) builds. The target Mac is Apple Silicon only.
- Notarization and Developer ID signing. Personal use needs ad-hoc signing only.
- Public distribution or an upstream release.
- Remote hosting of the fork. The repository stays local for now.
- Security hardening from the concerns audit (rmate remote mode, privileged helper, ATS). Reason: personal machine. Revisit if the fork leaves this Mac.
- Replacing legacy WebView or ODB code. Reason: change only if the build or run requires it.

## Context

- Upstream repository: `github.com/textmate/textmate`. Newest release is 2.0.23, dated 2021-10-12. The last upstream commit is `346b52b1`, by Allan Odgaard, dated 2021-10-12.
- Local branch `develop` is the upstream tip plus one docs commit, dated 2026-10-19.
- Build environment: arm64 Mac, macOS 26.7.1, Xcode 27.1, Homebrew at `/opt/homebrew`.
- Codebase map: `.planning/codebase/` (complete, 2026-10-19).
- Known build blockers, from the static audit. A build has not run yet:
  - `configure` looks only in `/usr/local`.
  - Submodules are uninitialized in this checkout.
  - `default.rave` hardcodes the deployment target `10.12` and `-std=c++2a`.
  - Deprecated APIs are in use. These include `WebView`, `NSTask.launchPath`, `NSURLConnection`, and `AuthorizationExecuteWithPrivileges`.
- `local-orig.rave` already contains arm64 and x86_64 target logic. Use it as a reference.

## Constraints

- **Platform:** Apple Silicon (arm64) only. Reason: the target Mac is arm64, and Intel support ends in the next macOS release.
- **OS floor:** the current macOS only. Reason: personal use on one machine.
- **Feature freeze:** no new features. Reason: user request.
- **Signing:** ad-hoc only. Reason: personal use, no distribution.
- **Hosting:** local only. Reason: no hosting decision yet.
- **Dependencies:** Boost, Cap'n Proto, google-sparsehash, MultiMarkdown, and Ragel, installed through Homebrew. Versions are not pinned yet.

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
*Last updated: 2026-10-19 after initialization*
