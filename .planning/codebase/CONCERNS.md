---
last_mapped_commit: 346b52b108b387462d4b3def481fb74983ae89f3
last_mapped_at: 2026-10-19
---
# Codebase Concerns

**Analysis Date:** 2026-10-19

**Scope and method:** Static reading of the tracked tree (`git ls-files`, 5,757 files; ~77k lines of C/C++/Objective-C++ under `Frameworks/`). No build or test run was performed. Each finding cites the file and line where it was observed. Items marked "verify" were inferred from code structure and should be reproduced before a fix is scheduled.

## Priority Summary

| Priority | Area | Item |
|----------|------|------|
| High | Security | Unauthenticated rmate listener in remote mode (`Applications/TextMate/src/RMateServer.mm`) |
| High | Security | Privileged auth helper accepts arbitrary root read/write (`Applications/PrivilegedTool/src/main.cc`) |
| High | Maintenance | No commits in the last 365 days; last commit 2021-10-12 |
| Medium | Security | ATS disabled globally (`Applications/TextMate/Info.plist`) |
| Medium | Security | Unverified update archive is streamed into `tar` before signature check (`Frameworks/SoftwareUpdate/src/OakDownloadManager.mm`) |
| Medium | Security | Hardened runtime weakened by `disable-library-validation` (`Applications/TextMate/Entitlements.plist`) |
| Medium | Performance | `nsRangeForRange:` copies the whole document on every call (`Frameworks/OakTextView/src/OakTextView.mm`) |
| Medium | Testing | 24 of 48 framework directories have no tests; CI runs no tests |
| Medium | Build | `configure` only looks in `/usr/local`; Submodules uninitialized in checkout |
| Low | Various | See individual sections |

---

## Security Considerations

### Unauthenticated rmate server in "remote" mode

- Risk: When the user sets `rmateServerListen` to `remote`, the rmate TCP server binds `in6addr_any` with no authentication. Any network peer can name arbitrary local paths. Opening a file with `path=` or `data=` reads it into TextMate. With `data-on-save` or `data-on-close` set, the content is written back over the socket to the peer when the document is saved or closed. This is an arbitrary-file-read path reachable from the network.
- Files:
  - `Applications/TextMate/src/RMateServer.mm` (bind at ~line 153: `iaddr.sin6_addr = listenForRemoteClients ? in6addr_any : in6addr_loopback;`; open handling ~lines 505-560: `documentWithPath:to_ns(args["path"])`, `dataWithContentsOfFile:to_ns(args["data"])`, `documentWithPath:to_ns(args["data"])` for write-back)
  - `Applications/TextMate/src/AppController.mm:479-482` (wires the setting into `setup_rmate_server`)
  - `Frameworks/Preferences/src/Keys.mm:95,99` (default is `localhost`; `remote` is opt-in)
- Current mitigation: Default is loopback only. The `token` argument is echoed back, never validated.
- Local-only mode also lets any local process of the same user open and read arbitrary files through `127.0.0.1`. The impact is limited because that process already runs as the user.
- Recommendations:
  1. Require a per-session secret. Generate it at startup, store it in a user-only file, and reject `open` requests whose `token` does not match.
  2. Refuse `path=`/`data=` values outside an allowlist, or require the file to already be open in TextMate for write-back.
  3. Until (1) lands, hide the `remote` option in `Frameworks/Preferences` behind a warning.

### Privileged authorization helper is a root file read/write oracle

- Risk: After a successful authorization check, the helper performs root-level `read` and `write` on any path. There is no path validation, so any client holding a valid authorization can read or overwrite any root-owned file. It also sets attributes (`path::set_attributes`) as root.
- Files:
  - `Applications/PrivilegedTool/src/main.cc:51-59` (`setup_socket`: `chmod(kAuthSocketPath, S_IRWXU|S_IRWXG|S_IRWXO)`, world-writable socket)
  - `Applications/PrivilegedTool/src/main.cc:74-110` (`read`/`write` actions with no path checks)
  - `Frameworks/authorization/src/constants.h:4-8` (`/var/run/com.macromates.auth_server.sock`, right `com.macromates.textmate.openfile`)
  - `Applications/PrivilegedTool/src/install.cc` (`launch_control`: `/bin/launchctl` executed as root)
- Current mitigation: Each request must present an authorization blob that passes `check_right`. The right is admin-group, 900 s timeout (`install.cc`, `add_policy`).
- Recommendations:
  1. Restrict the socket to the owning group or use `getpeereid()`/audit-token checks, rather than `S_IRWXO`.
  2. Allowlist writable paths (for example, files the user chose in an open panel) and reject everything else.
  3. Replace `AuthorizationExecuteWithPrivileges` and this custom socket protocol with `SMAppService` or XPC privileged helper APIs, which validate code signatures.

### Application Transport Security globally disabled

- Risk: `NSAllowsArbitraryLoads` is `true`, which disables ATS for every connection the app makes, including the WebKit views that render HTML output and browse arbitrary pages.
- Files: `Applications/TextMate/Info.plist:37-38`
- Recommendations: Remove the global flag. Add per-domain `NSExceptionDomains` only for hosts that truly need plain HTTP. Verify that `REST_API` (`default.rave`: `https://api.textmate.org`) and the bundle update hosts still work with ATS enforced.

### Hardened runtime weakened

- Risk: `disable-library-validation` lets the process load unsigned or differently-signed dylibs. The release build is signed with the hardened runtime (`default.rave`: `CS_FLAGS "--force --options runtime"`), so this entitlement undermines what the runtime flag provides.
- Files: `Applications/TextMate/Entitlements.plist:9-10`, `default.rave` (`CS_FLAGS`)
- Recommendations: Remove the entitlement unless a bundled plugin truly requires it. Confirm by launching a release build with it removed.

### Update archive streamed to `tar` before signature verification

- Risk: `OakDownloadManager` writes each received chunk to the `tar` extractor's pipe as it arrives (`writeData:`), and only checks the signature in `didCompleteWithError`. Unverified content is therefore unpacked to disk in an `NSItemReplacementDirectory`. The result is discarded on signature failure, and the directory is removed in `dealloc` (line ~88), but the window exists and the code path runs `tar -jxmkC` against attacker-controlled bytes.
- Files:
  - `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm:185-196` (`didReceiveData` writes to extractor)
  - `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm:239` (signature check after completion)
  - `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm:88` (temp directory cleanup)
- Contrast: `Frameworks/network/src/download_tbz.cc:179-184` verifies the signature before `rename_or_copy`, which is the safer ordering.
- Recommendations: Buffer the archive to a temporary file, verify it, and only then invoke `tar`. Reuse the verified-then-rename flow from `download_tbz.cc`.

### Load-error page interpolates the URL into a shell command string

- Risk: `ShowLoadErrorForURL` builds HTML containing JavaScript that calls `TextMate.system('/usr/bin/open ... "URL"')`. `EscapeHTML` only escapes `&`, `<`, and `"`. It does not escape `'`, backticks, or `$()`. A crafted URL can break out of the JS string, and `$(...)` inside the double quotes is expanded by `/bin/sh`.
- Files: `Frameworks/HTMLOutput/src/browser/HOBrowserView.mm:6-14`
- Current mitigation: The `TextMate` bridge is installed only in `Frameworks/HTMLOutput/src/OakHTMLOutputView.mm:132-145`, not in `HOBrowserView`, so the call is probably a `ReferenceError` in the browser window today (verify). The latent bug becomes exploitable if the error page is ever rendered in a bridged view.
- Recommendations: Remove the JavaScript and shell string. Open the URL with `[NSWorkspace.sharedWorkspace openURL:]` from a native action, and pass the URL through `NSURL` rather than text interpolation.

### HTML output exposes a shell bridge to local pages

- Risk: `TextMate.system()` runs `/bin/sh -c` with the given string (`Frameworks/HTMLOutput/src/helpers/HOJSBridge.mm:169-174`). It is installed for any page whose scheme is `file` or `tm-file`, and for any page while a command is running (`OakHTMLOutputView.mm:137-143`). Any local HTML file opened in HTML output can therefore run arbitrary shell commands.
- Current mitigation: Intended behavior for bundle commands. The bridge is disabled by `disableJavaScriptAPI`.
- Recommendations: Document the trust model. Consider restricting the bridge to pages produced by a running bundle command, not any `file://` page.

### Client-side license enforcement is not a security boundary

- Risk: License validation runs entirely in the client. It uses a hardcoded public key (`Frameworks/license/src/license.cc`, `public_key()`), a SHA-1 owner digest (`CC_SHA1` in `sha_digest`), and raw RSA "decrypt" (`kSecKeyAlgorithmRSAEncryptionRaw`), with a 1024-bit modulus in the key constant (verify the key size). The revocation list is weak-linked and gitignored, so the public tree never includes it.
- Files:
  - `Frameworks/license/src/license.cc:10` (`extern ... revoked_serials () WEAK_IMPORT_ATTRIBUTE;`)
  - `Frameworks/license/default.rave:7`
  - `.gitignore:3` (`Frameworks/license/src/revoked_serials.cc`)
  - `Frameworks/license/src/LicenseManager.mm:33-34` (revocation message)
- Recommendations: Treat this as a convenience check, not access control. If revocation matters, consult a server-side source. Replace SHA-1 with SHA-256 and raw RSA with PKCS#1 v1.5 or PSS verification when the next license format is introduced.

### Deprecated `AuthorizationExecuteWithPrivileges`

- Risk: Deprecated since macOS 10.7. It runs an arbitrary tool path as root and is wrapped in a compatibility header that suppresses the deprecation warning.
- Files:
  - `Shared/include/oak/compat.h:6-22` (`#pragma clang diagnostic ignored "-Wdeprecated-declarations"`)
  - `Frameworks/authorization/src/server.cc:26`
  - `Frameworks/Preferences/src/TerminalPreferences.mm:50`
- Recommendations: Migrate with the privileged-helper work above.

### Predictable temp socket and unchecked copy

- Risk: The mate/rmate Unix socket is `/tmp/textmate-<uid>.sock` (`Applications/TextMate/src/RMateServer.mm:99`). The server unlinks any existing file at that path before binding (line ~108). `/tmp` is shared, so another local user can pre-create the path, and the unlink-then-bind sequence is not atomic. The path is copied into `sockaddr_un.sun_path` with `strcpy` and no length check (line ~ `strcpy(addr.sun_path, _socket_path)`). The path is short, so the practical risk is low.
- Recommendations: Create the socket inside a per-user `$TMPDIR` or `~/Library/Caches` directory with mode 0700. Replace `strcpy` with `strlcpy`.

### Shell command strings from configuration

- Files: `Frameworks/CommitWindow/src/CommitWindow.mm:702` and `:758` (`/bin/sh -c cmdString`), `Frameworks/OakTextView/src/OakTextView.mm:3583` (`"#!/bin/sh\n" + commandString`). These run user-configured or bundle-supplied commands by design. Note them for any future work that composes these strings from file or network content.

---

## Tech Debt

### Project activity has stopped

- Issue: The last commit is `346b52b1`, dated 2021-10-12, and no commits have landed in the past 365 days. The newest release notes in `Applications/TextMate/about/Changes.md` are for v2.0.22 (2021-04-16).
- Impact: Security fixes and macOS compatibility changes are not being picked up. Known issues below remain open.
- Fix approach: Decide ownership before further feature work. If the project is maintained, establish a triage cadence and update the release notes. If not, mark the README and `.github/workflows/build.yml` accordingly.

### Stale CI configuration

- Issue: Two CI definitions disagree.
  - `.travis.yml` pins `osx_image: xcode7.2`, installs `libressl` and runs `brew upgrade boost`.
  - `.github/workflows/build.yml` runs on `macOS-latest` and installs `boost capnp google-sparsehash multimarkdown ninja ragel`.
- Neither workflow runs unit tests. Both run `ninja TextMate` only.
- Fix approach: Delete `.travis.yml` if Travis is no longer used. Add a test step (`ninja` test targets, see below) to `.github/workflows/build.yml`.

### Build variables hard-coded in `default.rave`

- Issue: `default.rave` hard-codes `APP_MIN_OS "10.12"`, `-std=c++2a`, `REST_API "https://api.textmate.org"`, and `CS_IDENTITY "-"`. Changing any of these requires editing the shared build file.
- Files: `default.rave`
- Fix approach: Move per-environment values into `local.rave`, which `configure` already creates (`configure:14-17`).

### Personal and release-process configuration committed to the tree

- Issue: `local-orig.rave` is tracked. It hardcodes `${HOME}/.nix-profile` paths, a `pretty_plist` install into `${HOME}/bin/pp`, and notarization via `APPLE_NOTARIZE_USER` and `APPLE_NOTARIZE_PASS` environment variables. The file holds no secret values, but it is one developer's environment.
- Files: `local-orig.rave`, `bin/notarize_await`, `bin/update_changes`
- Fix approach: Move it out of the tree or rename it into documentation. Keep credential handling in CI secret variables only.

### Duplicated import/export filter state machines

- Issue: The "run binary import hook" and "run text export hook" logic is copied between open and save. Both loops `break` after the first match, so the `filters` vector never holds more than one item. Both carry a `FIXME` saying a dialog is needed when there are multiple hooks, but that case cannot occur as written.
- Files:
  - `Frameworks/file/src/open.cc:296-318` and `:422-443`
  - `Frameworks/file/src/save.cc:245-265` and `:321-340`
- Fix approach: Extract one helper that takes the bundle event kind and the "already used" list. Remove the dead vector, or implement the dialog and then remove the `break`.

### Copy-pasted UI code

- Issue: `OakFileTableCellView` is duplicated between `Frameworks/OakFilterList/src/BundleItemChooser.mm:370` and `:383` (FIXME "Copy/paste from OakChooser.mm"). The same file-chooser stack is duplicated between `FileChooser.mm` and `OakChooser.mm`.
- Fix approach: Move the cell view into `Frameworks/OakFilterList/src/` as a shared class.

### God classes

- `Frameworks/OakTextView/src/OakTextView.mm`: 4,625 lines, about 264 method definitions.
- `Frameworks/DocumentWindow/src/DocumentWindowController.mm`: 2,885 lines, about 159 methods.
- `Frameworks/FileBrowser/src/FileBrowserViewController.mm`: 2,328 lines.
- `Frameworks/document/src/OakDocument.mm`: 1,866 lines, about 93 methods.
- `Frameworks/editor/src/editor.cc`: 1,548 lines.
- `Frameworks/selection/src/selection.cc`: 1,542 lines.
- Impact: Changes to one feature risk regressions in others. Little of this code is unit tested (see Test Coverage Gaps).
- Fix approach: Extract coherent subsystems (for example, the scroll and caret logic in `OakTextView.mm`, or the Find integration in `DocumentWindowController.mm`) behind their own classes, one at a time, with tests added first.

### Deprecated Apple APIs in active use

| API | Where | Note |
|-----|-------|------|
| `WebView` (legacy WebKit) | 10 files, e.g. `Frameworks/HTMLOutput/src/OakHTMLOutputView.mm` | Legacy WebKit, superseded by WKWebView |
| `NSURLConnection canHandleRequest:` | `Frameworks/HTMLOutput/src/OakHTMLOutputView.mm:199` | Deprecated since 10.11 |
| `NSTask.launchPath` | `Frameworks/SoftwareUpdate/src/SoftwareUpdate.mm:557`, `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm` | Use `executableURL` |
| `AuthorizationExecuteWithPrivileges` | `Shared/include/oak/compat.h` | See Security |
| Apple Events / ODB | `Applications/TextMate/src/ODBEditorSuite.mm` (189 lines), `ODBEditorSuite.h` | Legacy editor protocol |
| Carbon Resource Manager (`Get1Resource`) | `Frameworks/io/src/resource.cc` | Carbon-era |
| `Gestalt` | `Shared/include/oak/compat.h:8-16` | Use `NSProcessInfo` |

- Fix approach: Migrate `NSTask` and `NSURLConnection` first (small, low risk). Plan the WebKit migration separately because it affects HTML output behavior.

### Dead or incomplete code paths

- `Frameworks/document/src/Printing.mm:271` returns an empty array with `// TODO`.
- `Frameworks/BundlesManager/src/BundlesManager.mm:304` `// TODO Remove bundle's dependencies`.
- `Applications/gtm/src/gtm.cc:145-146` parses `-t` (trim) and `-d` (delimiters) with `// TODO`.
- `Applications/SyntaxMate/src/SyntaxMateImpl.mm:81` read-only UID access not provided.
- `Frameworks/DocumentWindow/src/DocumentWindowController.h:36-37` two IBActions marked "Move to AppController".

---

## Known Bugs

### Silent failure on read-only save

- Symptoms: Saving a read-only file gives no error. The `kFileTestReadOnly` case is a bare `break` with `// TODO show error`.
- Files: `Frameworks/file/src/save.cc:221-224`
- Trigger: Save a document whose file lacks write permission.
- Workaround: Check file permissions manually.
- Fix approach: Present the existing error sheet with the path and permission bits, and route the state machine to a terminal failure state.

### Three-way merge failure is only logged

- Symptoms: When a file changes on disk and no snapshot exists, `OakDocument` logs `NSLog` and does nothing else. The user is not told that the external change was not merged.
- Files: `Frameworks/document/src/OakDocument.mm:1783-1785`
- Trigger: An external edit to an open document whose snapshot was never captured.
- Workaround: Reload the document manually and reapply edits.
- Fix approach: Show the same "file changed on disk" prompt used for the no-merge case, and log through `os_log` to match the rest of the code.

### Overwrite not checked when opening multiple paths

- Symptoms: Dropping multiple paths onto the window can silently replace an existing file.
- Files: `Frameworks/DocumentWindow/src/DocumentWindowController.mm:1106` (`// FIXME check if paths[0] already exists (overwrite)`)
- Trigger: Multi-path drop with an existing destination.
- Fix approach: Check existence before the move and prompt.

### Spurious DELETE then WRITE events from the file watcher

- Symptoms: Overwriting a file emits DELETE followed by WRITE to observers, so observers can briefly treat the file as gone.
- Files: `Frameworks/FileBrowser/src/KEventManager.mm:359` (`// FIXME This may send DELETE followed by WRITE`)
- Fix approach: Coalesce events within a short window, or compare inode/fileID before reporting deletion.

### Non-atomic copy fallback when rename crosses devices

- Symptoms: If `rename` fails with `EXDEV`, the code falls back to `copyfile(... COPYFILE_MOVE)` directly into the destination. A failure partway through can leave the destination truncated.
- Files: `Frameworks/io/src/intermediate.mm:72-80` (`// TODO this should copy to dst under a new name, then re-run swap_files`)
- Trigger: Saving where the temporary file and the destination are on different volumes.
- Fix approach: Copy to a sibling temp file on the destination volume, fsync, then rename into place.

### Uncaught `std::out_of_range` in version comparison

- Symptoms: `std::stol` throws on overflow. A version component longer than the `long` range crashes the update check.
- Files: `Frameworks/SoftwareUpdate/src/OakCompareVersionStrings.mm:36`, `:63`, `:66` (guarded by `is_numeric`, which checks digits only, not length)
- Trigger: A version string with a very long numeric component from the update feed (verify with a unit test).
- Fix approach: Parse with `strtoll` and compare length first, or catch `std::out_of_range` around the comparison.

### Other incomplete handling

- `Frameworks/parse/src/grammar.cc:289` `// FIXME this is a kludge` compares plists on every check instead of registering a callback for bundle item changes.
- `Frameworks/editor/src/editor.cc:242` identifies the caret among snippet mirrors with a heuristic (`// FIXME`).
- `Frameworks/selection/src/selection.cc:631` and `:642`: with soft tabs, moving left or right can skip a full tab stop (`// FIXME`).
- `Frameworks/regexp/src/find.cc:352` an empty-range match returns a result, which also affects non-regex search.
- `Frameworks/io/src/path.cc:427` calls `fcntl(fd, F_GETPATH, buf)` twice in the same condition, and retries with `usleep(10)` only when the path check fails. If `fcntl` itself fails, the loop spins without sleeping (lines 418-430).
- `Frameworks/OakSystem/src/process.cc:12`: a process that exits on SIGINT may leave children running (`// TODO`).

---

## Performance Bottlenecks

### Whole-document copy on every UTF-8 to UTF-16 conversion

- Problem: `nsRangeForRange:` copies the document from 0 to the end of the range (`documentView->substr(0, to)`) on every call. This is O(N) in document size and is called from 14 sites, including the NSTextInputClient and accessibility paths.
- Files: `Frameworks/OakTextView/src/OakTextView.mm:1229-1238` (the existing TODO names an interval tree as the fix)
- Cause: The conversion is recomputed from scratch with no index.
- Improvement path: Reuse the `basic_tree_t` index already used for UTF-8 offsets (see `Shared/include/oak/basic_tree.h`). Cache the last conversion and invalidate it on edit.

### Choice-menu copies the full placeholder content per keystroke

- Problem: `realKeyDown:` copies the whole placeholder content twice per keypress (`Frameworks/OakTextView/src/OakTextView.mm:2218`, `:2227`) and then filters all choices with `find`.
- Files: `Frameworks/OakTextView/src/OakTextView.mm:2208-2235`
- Improvement path: Keep the prefix as a range over the buffer instead of a copied string, and filter the choices once per typed character rather than per candidate copy.

### Full-document spell check on load

- Problem: The spell checker does not delay work until the misspellings are requested, so loading a large file checks everything up front.
- Files: `Frameworks/buffer/src/spelling.cc:68-72` (`// TODO We should delay checking to the misspellings function`)
- Improvement path: Check only the visible range and the dirty ranges, as the TODO at line ~98 proposes.

### Width calculation scans all rows for wrapped layout

- Problem: The width used for the layout considers every row between `first` and `last`, and each paragraph's total width (not the wrapped row width).
- Files: `Frameworks/layout/src/layout.cc:286-294` (`// TODO paragraph_t::width covers the entire paragraph`)
- Improvement path: Use the row width directly once a wrapped row is known.

### Timers running at high frequency

- `Frameworks/OakTextView/src/OakTextView.mm:4217`: drag-scroll timer at 25 Hz.
- `Frameworks/OakTextView/src/OTVStatusBar.mm:307`: macro-recording animation at 50 Hz.
- `Frameworks/SoftwareUpdate/src/SoftwareUpdate.mm:685`: progress polling at 25 Hz.
- Impact: Small individually. Each runs for the life of its operation, so check that all are invalidated when the view is hidden.

### Blocking waits in the `mate` CLI

- Problem: `mate` retries connecting up to 10 times with a 500 ms sleep after launching the app, which can block a script for about 5 seconds.
- Files: `Applications/mate/src/mate.mm:391-400`
- Improvement path: Accept a `--timeout` flag and report the wait on stderr. Keep the default.

---

## Fragile Areas

### `NULL_STR` sentinel for "no value"

- Files: `default.rave` (`add FLAGS "-D'NULL_STR=\"\\uFFFF\"'"`), used about 568 times across `Frameworks/`, `Applications/`, and `PlugIns/`.
- Why fragile: `std::string` has no null, so the code encodes "absent" as the code point U+FFFF. A real string containing U+FFFF is indistinguishable from "absent", and every comparison must remember to use `NULL_STR` rather than `""`.
- Safe modification: Do not introduce new uses. For new code, use `std::optional<std::string>` or a `nullptr` pointer. Migrate one module at a time.
- Test coverage: No dedicated test for U+FFFF handling was found.

### Duplicated open/save state machines

- Files: `Frameworks/file/src/open.cc`, `Frameworks/file/src/save.cc`
- Why fragile: Both implement near-identical state machines (`kStateExecuteBinaryImportFilter` and `kStateExecuteTextExportFilter`) with their own `break` and `FIXME` logic. A fix to one path is easily missed in the other.
- Safe modification: Change both together, and add tests for each path before touching either.

### Snippet mirror and caret mapping

- Files: `Frameworks/editor/src/editor.cc:242`
- Why fragile: Multiple ranges are returned when a snippet has mirrors, and the code guesses which one is the caret.
- Safe modification: Do not change the heuristic without a regression test covering nested mirrors.

### Submodules and vendored code

- Files: `.gitmodules` (7 entries). In this checkout every submodule is uninitialized (`git submodule status` shows a leading `-`): `bin/CxxTest`, `Applications/TextMate/icons`, `PlugIns/dialog`, `PlugIns/dialog-1.x`, `vendor/Onigmo/vendor`, `vendor/kvdb/vendor`, `Applications/SyntaxMate/resources/SyntaxMate.tmBundle`.
- Why fragile: The build fails or silently lacks icons, tests, and the Onigmo/kvdb sources until `git submodule update --init --recursive` runs. The submodules point at `github.com/textmate/*` and a third-party bundle repository.
- Safe modification: Run `git submodule update --init --recursive` before any build. Pinned SHAs in the index are good; keep updating them deliberately.

### Build configuration assumes `/usr/local`

- Files: `configure:14-24`, which writes `local.rave` with `-I/usr/local/include` and `-L/usr/local/lib` and checks `/usr/local/include/boost/crc.hpp` and similar headers.
- Why fragile: Homebrew on Apple Silicon installs to `/opt/homebrew`, so `configure` fails with "dependency missing" even when the dependencies are installed. `configure` also uses `[[` under `#!/bin/sh`, which fails on non-bash `sh`.
- Safe modification: Detect the prefix with `brew --prefix` when available, and fall back to `/usr/local`. Declare the shell as `bash` if bash syntax is kept.

### Deployment target and toolchain

- Files: `default.rave` (`APP_MIN_OS "10.12"`, `-std=c++2a`, `-fobjc-arc`), `.travis.yml` (`xcode7.2`).
- Why fragile: macOS 10.12 is long out of support. `-std=c++2a` depends on compiler support that changed across Xcode releases. Raising `APP_MIN_OS` would drop systems that the code still targets.
- Safe modification: Build with the current Xcode before changing any of these values, and record the result in `README.md`.

### Multi-threaded `vfork` followed by non-trivial calls

- Files: `Frameworks/command/src/runner.mm:45-78`, `Frameworks/OakCommand/src/OakCommand.mm:71-101`, `Frameworks/OakDebug/src/OakAssert.mm:31-48`, `Applications/mate/src/mate.mm:91-94`, `Applications/PrivilegedTool/src/install.cc:46`
- Why fragile: Between `vfork` and `execve`, the child calls `close`, `dup`, `setpgid`, `chdir`, and `perror`. POSIX only guarantees async-signal-safe functions in that window, and `perror` may take locks.
- Safe modification: Keep the post-`vfork` code to async-signal-safe calls. Prefer `posix_spawn` (already used in `Frameworks/io/src/exec.cc:22-55`) for new code.

### Crash-reporting and logging inconsistency

- Files: `NSLog` in `Frameworks/document/src/OakDocument.mm:1783`, `os_log_error` in `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm`, `perror` in child processes (`Frameworks/command/src/runner.mm`).
- Why fragile: Three logging styles are mixed. Errors in the document path do not appear in the same log stream as update errors.
- Safe modification: Standardize on `os_log` (the `io` and update code already do so) when touching these files.

---

## Test Coverage Gaps

- Layout: 92 tracked files live under `tests/` directories across 24 test directories. These include `vendor/Onigmo/tests`. Fewer than half of the framework directories have tests.
- Frameworks with no `tests/` directory (24 of 48): `Frameworks/BundleEditor`, `BundleMenu`, `BundlesManager`, `CommitWindow`, `crash`, `CrashReporter`, `DocumentWindow`, `encoding`, `FileBrowser`, `Find`, `HTMLOutputWindow`, `license`, `MenuBuilder`, `OakCommand`, `OakDebug`, `OakFilterList`, `OakFoundation`, `OakSystem`, `OakTabBarView`, `OakTextView`, `Preferences`, `TMFileReference`, `undo`, `updater`.
- Not executed in CI: `.github/workflows/build.yml` runs `ninja TextMate` only. Test targets are declared with `tests` directives in `default.rave` files (for example, `Frameworks/HTMLOutput/default.rave:8`, `Frameworks/buffer/default.rave:5`) and are not invoked.
- Untested security paths:
  - rmate request handling (`Applications/TextMate/src/RMateServer.mm`)
  - privileged helper protocol (`Applications/PrivilegedTool/src/main.cc`)
  - update signature checks (`Frameworks/SoftwareUpdate/src/OakDownloadManager.mm`, `Frameworks/network/src/download_tbz.cc`)
  - load-error HTML construction (`Frameworks/HTMLOutput/src/browser/HOBrowserView.mm`)
- Untested parsing: `OakCompareVersionStrings.mm` (has a test file, `Frameworks/SoftwareUpdate/tests/t_OakCompareVersionStrings.mm`, but no overflow case).
- Priority: High for the security paths; Medium for `OakTextView` and `DocumentWindow` (large and frequently changed).
- Fix approach: Add the CI test step first. Then add tests for the rmate and update-signature paths, which can run without the UI.

---

## Dependencies at Risk

- Boost: Required by `configure` (`boost/crc.hpp`, `boost/variant.hpp`). Not pinned. Version drift is likely between the README and CI.
- Cap'n Proto: Used by `Frameworks/encoding` and `Frameworks/plist` (`capnp` sources). `configure` requires the library and `kj`. Pin a version.
- google-sparsehash: `sparsehash/dense_hash_map` is required. The project is effectively unmaintained.
- MultiMarkdown and Ragel: Build tools. Ragel generates `.rl` sources in `Frameworks/plist/src/`. A Ragel upgrade can change generated output.
- Onigmo and kvdb: vendored as submodules, with `vendor/Onigmo/src/setup.c` and `vendor/Onigmo/config.h` tracked. Check for upstream security fixes before any regex-related change.
- CxxTest: test harness submodule at `bin/CxxTest`. Uninitialized in this checkout.
- Ruby scripts: `bin/gen_test` has a hardcoded `#!/System/Library/Frameworks/Ruby.framework/Versions/Current/usr/bin/ruby` shebang. It breaks on systems with a different Ruby layout.
- Apple frameworks: see the deprecated-API table in Tech Debt. Each one is a compatibility risk on new macOS releases.

---

## Missing Critical Features

- Automated test execution in CI (see Test Coverage Gaps).
- A supported way to build on Apple Silicon without editing `local.rave` by hand (see Fragile Areas).
- Revocation list for licenses in public builds (the file is gitignored and weak-linked).
- Telemetry or crash upload: none found. Crash information stays local (`Frameworks/CrashReporter`). Confirm this is intended.

---

## Scaling Limits

- Document size: Paths that call `substr(0, to)` or copy the whole document (see Performance) scale linearly with file size. The cost is visible at large sizes, but no hard limit was identified in the code read for this audit.
- Not assessed: concurrency limits on the rmate server, and any file-size guard on open. Check these before relying on them.

---

*Concerns audit: 2026-10-19*
