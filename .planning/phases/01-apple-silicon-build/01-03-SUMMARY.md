---
phase: 01-apple-silicon-build
plan: 03
subsystem: build
tags: [test-baseline, regression, gen_test, runners, arm64, checkpoint]

# Dependency graph
requires:
  - phase: 01-apple-silicon-build (plan 02)
    provides: clean checkout at 1c0ca4f9 and build products under /Users/bfs/build/textmate/release
provides:
  - Spike: io runner built and run outside the build system (23 passed, 1 failed, log holds the count line)
  - Test baseline table for the 25 test directories: 15 ran, 10 un-baselined
  - Checkpoint: 15 counted rows, below the 20-row minimum
affects: [phase 3 regression comparison]

# Actuals (#2632): no tracked file changed, so chars/4 over the tracked diff is 0
actuals:
  tokens: 0
  tasks: 3
  commits: 0

# Commit ledger (#3968), measured from HEAD at plan start
commits: 0
plan_head_before: c22b37b7d5d716e2962e39fd3bc5e5d420768a0b
plan_head_after: c22b37b7d5d716e2962e39fd3bc5e5d420768a0b

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Runner method (option b): gen_test output compiled with flags from one object of the directory, linked with the closure objects of the TextMate link line"
    - "One log per directory; Pass and Fail copied from the run section of that log"

key-files:
  created:
    - /Users/bfs/build/textmate-baseline/baseline.sh (driver, outside the repository)
    - /Users/bfs/build/textmate-baseline/logs/ (25 run logs, outside the repository)
    - /Users/bfs/build/textmate-baseline/runners/ (runner .cc, .o, and binaries, outside the repository)
  modified: []

key-decisions:
  - "Plan stops at a checkpoint. 15 rows ran, the 20-row minimum is not met. The method was not changed to reach 20."
  - "Baseline runs on the post-BUILD tree at 1c0ca4f9, not on 346b52b1 (see Test baseline)."

patterns-established:
  - "Driver baseline.sh <dir> runs one directory: gen_test, compile, link, run, all logged; prints one row"

requirements-completed: []

duration: not recorded (record_start_time step was not run)
completed: 2026-10-19
status: checkpoint
---

# Phase 1 Plan 03: Test baseline Summary

**Test baseline on the post-BUILD tree at 1c0ca4f9 counted 15 of 25 test directories with runners built outside the build system. The 20-row minimum is not met, so the plan stops at a checkpoint.**

## Performance

- **Duration:** not recorded (the start timestamp was not captured at plan start)
- **Tasks:** 3 of 3 executed. Task 3 wrote the table and stopped at the minimum-count checkpoint.
- **Tracked files modified:** 0 in the clean checkout, 0 in the planning checkout outside `.planning`

## Checkpoint

**CHECKPOINT REACHED.** Counted rows (Result `ran` with integer Pass and Fail): **15**. Required: **20**.

The 10 rows that are not counted, with their causes:

| Cause | Rows | Count |
| ----- | ---- | ----- |
| Harness flag selection (fixed-up flags still miss an include path, or an Objective-C object supplied C++ flags). Not diagnosed further. | buffer, layout, HTMLOutput | 3 |
| Runner C++ flags cannot come from the directory's C objects (`-std=c99`) | vendor/Onigmo | 1 |
| gen_test wraps `.mm` test bodies in a namespace, and Objective-C declarations are not allowed there | ns, OakAppKit | 2 |
| Runtime exit in product code (`NSInvalidArgumentException` from `command::runner_t::launch`, or SIGBUS) | command, editor, file, cf | 4 |

Fixing the three harness flag cases would give 18 counted rows, still below 20. Reaching 20 would need a change to the plan's rules: how the `cxx_tests` `.mm` files are handled, or how C-only directories get C++ flags. That is a decision for the user. The method was not changed to reach the threshold.

## Spike

**Target:** `Frameworks/io`. File set (`tests/*.cc`, 7 files, gen_test run from the clean checkout):

```
/Users/bfs/src/textmate-clean/Frameworks/io/tests/t_attributes.cc
/Users/bfs/src/textmate-clean/Frameworks/io/tests/t_disambiguate_paths.cc
/Users/bfs/src/textmate-clean/Frameworks/io/tests/t_escape.cc
/Users/bfs/src/textmate-clean/Frameworks/io/tests/t_events.cc
/Users/bfs/src/textmate-clean/Frameworks/io/tests/t_path.cc
/Users/bfs/src/textmate-clean/Frameworks/io/tests/t_resolve.cc
/Users/bfs/src/textmate-clean/Frameworks/io/tests/t_unique.cc
```

**gen_test** (cwd `/Users/bfs/src/textmate-clean`, stdout to `runners/Frameworks_io.cc`, stderr to the log):

```
/Users/bfs/src/textmate-clean/bin/gen_test <the 7 absolute paths above>
```

Two Ruby ERB deprecation warnings appear on stderr. They are expected.

**Compile** (flags from `release/_CompileClang/Frameworks/io/src/entries.o`, the last line of `ninja -C /Users/bfs/src/textmate-clean -t commands <object>`, with `-MMD -MF` and the source removed and `-c` kept, as the runner has to be compiled to an object):

```
xcrun --sdk macosx clang -funsigned-char -Wall -Wwrite-strings -Wformat -Winit-self -Wmissing-include-dirs -Wno-parentheses -Wno-sign-compare -Wno-switch -Wno-c99-designator -DNULL_STR=\"\\uFFFF\" -DREST_API=\"https://api.textmate.org\" -IShared/include -mmacosx-version-min=26.7.1 -I/opt/homebrew/include -Os -DNDEBUG -flto=thin -std=c++2a -include /Users/bfs/build/textmate/release/_PCH/Shared/PCH/prelude.cc -I/Users/bfs/build/textmate/release/_Include/{OakFoundation,Onigmo,cf,crash,io,ns,plist,regexp,text} -I/Users/bfs/build/textmate/release/_Include/io -c -o /Users/bfs/build/textmate-baseline/runners/Frameworks_io.o /Users/bfs/build/textmate-baseline/runners/Frameworks_io.cc
```

The last `-I` (`_Include/io`) is the directory's own include path. It was added after the first loop pass (see Deviations).

**Link** (flags from the TextMate link line, non-object words kept, `-o` replaced; objects from the require closure of io, which is io, text, cf, ns, regexp, crash, OakFoundation, plist, and vendor/Onigmo; no `Applications/` object):

```
xcrun --sdk macosx clang -o /Users/bfs/build/textmate-baseline/runners/Frameworks_io -mmacosx-version-min=26.7.1 -fobjc-link-runtime -L/opt/homebrew/lib -flto=thin -Wl,-cache_path_lto,/Users/bfs/build/textmate/.cache -Wl,-dead_strip -Wl,-dead_strip_dylibs -Wl,-U,__Z15revoked_serialsv -framework AddressBook -framework AppKit -framework ApplicationServices -framework AudioToolbox -framework Carbon -framework Cocoa -framework CoreFoundation -framework Foundation -framework Quartz -framework Security -framework SystemConfiguration -framework UserNotifications -framework WebKit -lc++ -lcapnp -lcurl -liconv -lkj -lsqlite3 -lz <closure framework flags> <74 object paths, listed in the log under "link objects: 74", in that order> /Users/bfs/build/textmate-baseline/runners/Frameworks_io.o
```

- **Object count:** 74 link objects from the closure, plus the runner object. Of the 74, 9 are io's own sources.
- **First error line:** none. Compile and link exit 0.
- **Run:** `perl -e 'alarm shift; exec @ARGV' 120 runners/Frameworks_io --verbose --no-parallel`, cwd `/Users/bfs/src/textmate-clean`. Output: `Frameworks_io: 1 of 24 tests failed:` with `t_attributes.cc:26: Expected (path::attributes(file).size() == 2), found (3 != 2)`. Exit 1. Count line present, so Result `ran`, Pass 23, Fail 1.
- **Spike result:** PASS. The runner linked, ran, and wrote a count line. No tracked file changed.

The exact `$ ` command lines for each step are in `/Users/bfs/build/textmate-baseline/logs/Frameworks_io.log`.

## Test baseline

- **Clean checkout HEAD:** `1c0ca4f93e2ae99f5104b4b2e0c9b8828b8aeac5` (`git -C /Users/bfs/src/textmate-clean rev-parse HEAD`).
- **Develop tip at write time:** `c22b37b7`. The commits since `1c0ca4f9` are `.planning`-only (`git log 1c0ca4f9..develop -- . ':(exclude).planning'` is empty), so the code under test is unchanged.
- **Tree:** the baseline runs on the post-BUILD tree at `1c0ca4f9`, not on baseline commit `346b52b1`.
- **Difference from 346b52b1:** BUILD-04 changes `APP_MIN_OS` from `10.12` (default.rave line 1 at 346b52b1) to `26.7.1`. The compile flags carry `-mmacosx-version-min=26.7.1` where 346b52b1 has `-mmacosx-version-min=10.12`. BUILD-02 changes the include and library paths in `local.rave` from `/usr/local` to `/opt/homebrew`; the flags carry `-I/opt/homebrew/include` and `-L/opt/homebrew/lib`. BUILD-02 and BUILD-03 change `configure` and `README.md`, which compile no source.
- **Method:** option b. The runners that `bin/gen_test` writes are built and run outside the build system, with no tracked change.

**Table** (Pass and Fail copied from each directory's run section in `/Users/bfs/build/textmate-baseline/logs/`; `n/a` marks rows with no count line):

| Directory | Test globs | Result | Exit | Pass | Fail |
|---|---|---|---|---|---|
| Frameworks/authorization | `tests/t_*.cc` | ran | 0 | 1 | 0 |
| Frameworks/buffer | `tests/*.{cc,mm}` | compile-fail | 1 | n/a | n/a |
| Frameworks/bundles | `tests/*.cc` | ran | 0 | 5 | 0 |
| Frameworks/cf | `tests/*.cc` | no-count | 138 | n/a | n/a |
| Frameworks/command | `tests/*.cc` | no-count | 134 | n/a | n/a |
| Frameworks/document | `tests/t_*.{mm,cc}` | ran | 0 | 2 | 0 |
| Frameworks/editor | `tests/*.cc` | no-count | 134 | n/a | n/a |
| Frameworks/file | `tests/*.cc` | no-count | 134 | n/a | n/a |
| Frameworks/HTMLOutput | `tests/*.cc` | compile-fail | 1 | n/a | n/a |
| Frameworks/io | `tests/*.cc` | ran | 1 | 23 | 1 |
| Frameworks/layout | `tests/t_*.cc`; cxx `tests/gui_*.mm` | compile-fail | 1 | n/a | n/a |
| Frameworks/network | `tests/*.cc` | ran | 1 | 0 | 1 |
| Frameworks/ns | `tests/t_*.{cc,mm}`; cxx `tests/gui_*.mm` | compile-fail | 1 | n/a | n/a |
| Frameworks/OakAppKit | (no tests key); cxx `tests/gui_*.mm` | compile-fail | 1 | n/a | n/a |
| Frameworks/parse | `tests/t_*.cc` | ran | 0 | 4 | 0 |
| Frameworks/plist | `tests/*.cc` | ran | 0 | 24 | 0 |
| Frameworks/regexp | `tests/*.cc` | ran | 0 | 41 | 0 |
| Frameworks/scm | `tests/t_*.cc` | ran | 1 | 29 | 3 |
| Frameworks/scope | `tests/*.cc` | ran | 0 | 13 | 0 |
| Frameworks/selection | `tests/*.cc` | ran | 0 | 24 | 0 |
| Frameworks/settings | `tests/*.cc` | ran | 0 | 9 | 0 |
| Frameworks/SoftwareUpdate | `tests/t_*.mm` | ran | 0 | 3 | 0 |
| Frameworks/text | `tests/*.cc` | ran | 0 | 34 | 0 |
| Frameworks/theme | `tests/*.cc` | ran | 0 | 1 | 0 |
| vendor/Onigmo | `tests/t_*.cc` | compile-fail | 1 | n/a | n/a |

Counted rows: **15** (authorization, bundles, document, io, network, parse, plist, regexp, scm, scope, selection, settings, SoftwareUpdate, text, theme).

**Un-baselined** (10 rows, Result not `ran`; first error line from each log):

1. Frameworks/buffer: compile-fail, exit 1. `/Users/bfs/build/textmate/release/_Include/buffer/buffer/buffer.h:9:10: fatal error: 'scope/scope.h' file not found`. Flags came from an io `.mm` object, whose requires do not include `scope`. Harness artifact.
2. Frameworks/cf: no-count, exit 138. `Bus error: 10` during the run. No count line.
3. Frameworks/command: no-count, exit 134. `*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '*** -[__NSPlaceholderArray initWithObjects:count:]: attempt to insert nil object from objects[1]'`, thrown from `command::runner_t::launch`.
4. Frameworks/editor: no-count, exit 134. The same NSInvalidArgumentException.
5. Frameworks/file: no-count, exit 134. The same NSInvalidArgumentException, from `command::runner_t::launch` via `file::save`.
6. Frameworks/HTMLOutput: compile-fail, exit 1. `error: Objective-C was enabled in precompiled file '/Users/bfs/build/textmate/release/_PCH/Shared/PCH/prelude.mm.gch' but is currently disabled`. HTMLOutput has no `.cc` object, so the flags came from an Objective-C object. Harness artifact.
7. Frameworks/layout: compile-fail, exit 1. `/Users/bfs/build/textmate/release/_Include/layout/layout/ct.h:4:10: fatal error: 'theme/theme.h' file not found`. Flags came from an OakFoundation `.mm` object. Harness artifact.
8. Frameworks/ns: compile-fail, exit 1. `/Users/bfs/src/textmate-clean/Frameworks/ns/tests/gui_key_events.mm:5:12: error: Objective-C declarations may only appear in global scope`. gen_test places the `.mm` body inside a namespace.
9. Frameworks/OakAppKit: compile-fail, exit 1. `/Users/bfs/src/textmate-clean/Frameworks/OakAppKit/tests/gui_dictionary.mm:3:12: error: Objective-C declarations may only appear in global scope`. Same cause as ns.
10. vendor/Onigmo: compile-fail, exit 1. `error: invalid argument '-std=c99' not allowed with 'C++'`. The only objects are C, so their flags cannot compile a C++ runner.

**Regression check wording.** The Regression check says `ninja <framework>/test`. This checkout generates no test targets (`bin/rave` reads the `tests` keys but emits no test target, and `bin/gen_test` has no caller in the repo), so the runners were built outside the build system under option b. The check's wording differs from this method.

## Deviations from Plan

**Plan-level result:** the plan stopped at the minimum-count checkpoint, as its rules require. The spike passed.

### Auto-fixed issues (harness, during Task 2)

**1. [Rule 1 - Bug] Own `_Include` path missing from compile flags.** The first loop pass gave 3 counted rows. Tests include their own framework headers as `<NAME/header.h>`. The flags of an object carry only the `_Include` paths of the directory's requires, so authorization, cf, editor, and regexp failed on `file not found`. Fix: add `-I<release>/_Include/<NAME>` when that directory exists. Files: `/Users/bfs/build/textmate-baseline/baseline.sh` (outside the repository). The spike io row was unchanged by this fix (23 pass, 1 fail).

**2. [Rule 1 - Bug] Objective-C mode flags for `.mm` file sets.** In the first pass, a file set with `.mm` files used a `.cc` object's flags (`prelude.cc.gch`), which conflicts with `-x objective-c++`. Fix: take the flags from an Objective-C++ object of the directory or its closure. Remaining issue (not fixed): these closure flags can miss include paths (rows 1 and 7 above), and HTMLOutput has no `.cc` object (row 6).

### Plan interpretation

- **Spike compile step.** The plan says to remove `-c` and then add `-o` for the runner object. Without `-c` the step would link, not compile, so `-c` was kept.
- **Link objects.** The plan says to exclude objects that define `main`. The link set comes from the closure directories, and no `Applications/` object is in it, so the check was not run with `nm`. The io link succeeded, so no second `main` was present for io.
- **`-Wl,-U,__Z15revoked_serialsv`** was copied from the TextMate link line. It makes that one symbol a dynamic-lookup symbol.

### Environment

- **Root pin.** This dispatch carried no `<project_root_pin>` block, so the sequential root-pin guard could not be run. The fallback (warning, continue) was used. Before each commit, the planning checkout's toplevel was checked against `/Users/bfs/bitbucket/textmate`.
- **Protected branch.** The planning checkout is on `develop`. `gsd_run query git.base-branch --is-protected develop` returns `false`, so the HEAD assertion passes.

### Not done (by checkpoint)

- ROADMAP progress was not updated. The plan did not complete, and `roadmap.update-plan-progress` would count this SUMMARY as complete.
- The plan counter was not advanced (`state.advance-plan` not run).
- STATE.md received a blocker (`state.add-blocker`) and a session record (`state.record-session`) for this checkpoint.

## Issues Encountered

- 10 of 25 rows did not count; see Checkpoint.
- Earlier plan 01-02 notes still apply: deprecated-API warnings are BUILD-05 (Phase 2). README line 79 is out of scope.

## Known Stubs

None.

## Threat Flags

None. The runners were compiled from the clean checkout's file set, outside the repository. Each run had a 120-second alarm (T-01-12). No tracked file changed in either checkout (T-01-11). The baseline records the clean HEAD and the post-BUILD difference (T-01-13).

## Self-Check

Checked before this SUMMARY was written:

- FOUND: /Users/bfs/build/textmate-baseline/baseline.sh
- FOUND: /Users/bfs/build/textmate-baseline/runners/Frameworks_io (executable), log holds `1 of 24 tests failed`
- FOUND: 25 logs under /Users/bfs/build/textmate-baseline/logs/, one per directory
- Clean checkout `git status --porcelain`: empty
- `bin/rave` and `bin/gen_test` differ from 346b52b1: none
- Non-planning files differing from 346b52b1: `README.md configure default.rave`, unchanged from plan 01-02

## Next Phase Readiness

- The baseline needs a user decision before it can reach 20 rows. Options: (a) change the handling of the `cxx_tests` `.mm` files, (b) give C-only directories C++ flags, or (c) record the 15-row baseline as final and let Phase 3 compare against it.
- Phase 2 (BUILD-05) owns the deprecated-API warnings.

---
*Phase: 01-apple-silicon-build*
*Completed: 2026-10-19 (checkpoint)*
