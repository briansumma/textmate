---
last_mapped_commit: 346b52b108b387462d4b3def481fb74983ae89f3
last_mapped_at: 2026-10-19
---
<!-- refreshed: 2026-10-19 -->

# Architecture

**Analysis Date:** 2026-10-19

## System Overview

```text
┌────────────────────────────────────────────────────────────────────────┐
│  Applications/TextMate  (AppKit app: main.mm, AppController, plug-ins)  │
├───────────────────────────┬───────────────────────────┬────────────────┤
│  GUI frameworks (ObjC++)  │  Auxiliary apps (CLI/XPC) │  PlugIns/*     │
│  OakTextView, DocumentWin-│  mate, tm_query, bl, gtm, │  dialog,       │
│  dow, FileBrowser, Find,  │  indent, SyntaxMate (XPC),│  dialog-1.x    │
│  BundleEditor, Preferences│  PrivilegedTool (helper)  │  (submodules)  │
│  `Frameworks/*/src/*.mm`  │  `Applications/*/src/`    │  `PlugIns/`    │
└─────────────┬─────────────┴─────────────┬─────────────┴────────────────┘
              │                           │
              ▼                           ▼
┌────────────────────────────────────────────────────────────────────────┐
│  Editing core (C++): document model, editing commands, layout          │
│  buffer, editor, selection, undo, layout, command, document (ObjC)     │
│  `Frameworks/buffer/src`, `Frameworks/editor/src`, `Frameworks/layout` │
└─────────────┬──────────────────────────────────────────────────────────┘
              ▼
┌────────────────────────────────────────────────────────────────────────┐
│  Language and data services (C++)                                      │
│  parse, scope, regexp (Onigmo), bundles, settings, theme, plist        │
│  `Frameworks/parse`, `Frameworks/scope`, `Frameworks/bundles`          │
└─────────────┬──────────────────────────────────────────────────────────┘
              ▼
┌────────────────────────────────────────────────────────────────────────┐
│  Platform primitives (C++/ObjC)                                        │
│  cf, text, io, file, scm, network, authorization, crash, OakSystem,    │
│  OakFoundation, shared headers in `Shared/include/oak`                 │
└────────────────────────────────────────────────────────────────────────┘
```

## Component Responsibilities

| Component | Responsibility | File |
|-----------|----------------|------|
| App entry | Process setup, signal handling, session save on SIGTERM, `NSApplicationMain` | `Applications/TextMate/src/main.mm` |
| App controller | Global menus, app commands, document open/close routing, bundles menu | `Applications/TextMate/src/AppController.mm` (+ `AppController Commands.mm`, `AppController Documents.mm`, `AppController Menus.mm`) |
| Plug-in loader | Loads `PlugIns/*.plugin` bundles, exposes `TMPlugInController.sharedInstance` | `Applications/TextMate/src/TMPlugInController.mm` |
| `rmate` socket server | Listens on `/tmp/textmate-<uid>.sock` (AF_UNIX) for remote-edit clients | `Applications/TextMate/src/RMateServer.mm` |
| Window/session layer | One window per project; tabs, project layout, session restore | `Frameworks/DocumentWindow/src/DocumentWindowController.mm` |
| Document model (ObjC) | `OakDocument` owns a `ng::buffer_t`, path, file type, encoding, SCM status; `OakDocumentController` is the registry | `Frameworks/document/src/OakDocument.mm`, `Frameworks/document/src/OakDocumentController.mm` |
| Editor wrapper | `OakDocumentEditor` holds a `std::unique_ptr<ng::editor_t>` over the document's buffer | `Frameworks/document/src/OakDocumentEditor.mm` |
| Text storage | `ng::buffer_t` on `ng::detail::storage_t` on `oak::basic_tree_t` (balanced, indexed tree) | `Frameworks/buffer/src/buffer.h`, `Frameworks/buffer/src/storage.h`, `Shared/include/oak/basic_tree.h` |
| Editing operations | Movement, insert/delete, snippets, clipboard, indent, transforms | `Frameworks/editor/src/editor.cc`, `Frameworks/editor/src/snippets.cc` |
| Selections | Multi-caret/range selection model and transforms | `Frameworks/selection/src/selection.cc` |
| Undo | Undo/redo stack over buffer edits | `Frameworks/undo/src/undo.cc` |
| Layout | Soft-wrapping, folds, paragraph metrics, CoreText rendering into a `CGContext` | `Frameworks/layout/src/layout.cc`, `Frameworks/layout/src/render.cc` |
| Text view (ObjC) | Input handling, drawing, Cocoa key bindings | `Frameworks/OakTextView/src/OakTextView.mm` |
| Document view (ObjC) | Composes `OakTextView` + `GutterView` + `OTVStatusBar` | `Frameworks/OakTextView/src/OakDocumentView.mm` |
| Syntax parsing | Grammar compilation and incremental scope assignment | `Frameworks/parse/src/parse.cc`, `Frameworks/parse/src/grammar.cc`, `Frameworks/buffer/src/parsing.cc` |
| Scopes | Scope stack types and selector matching | `Frameworks/scope/src/scope.cc`, `Frameworks/scope/src/match.cc` |
| Bundles | Loads `.tmBundle` items (grammars, themes, commands, snippets), query API | `Frameworks/bundles/src/load.cc`, `Frameworks/bundles/src/query.cc` |
| Bundle manager (ObjC) | Process-wide bundle registry; singleton | `Frameworks/BundlesManager/src/BundlesManager.mm` |
| Bundle command runner | Parses bundle command definitions and runs them as child processes with pipes | `Frameworks/command/src/parser.cc`, `Frameworks/command/src/runner.mm` |
| Settings | Cascading `.tm_properties` and bundle-provided settings, expanded variables | `Frameworks/settings/src/settings.cc` |
| Theme | Theme parsing and mapping scopes to styles | `Frameworks/theme/src/theme.cc`, `Frameworks/theme/src/OakTheme.mm` |
| File I/O | Read/write/encoding/filter of documents; routes privileged access to the auth server | `Frameworks/file/src/open.cc`, `Frameworks/file/src/save.cc`, `Frameworks/file/src/encoding.cc` |
| Privilege escalation | Client (`authorization`) talks over AF_UNIX to a launchd helper that performs root ops | `Frameworks/authorization/src/connection.cc`, `Applications/PrivilegedTool/src/main.cc` |
| SCM | Detects repositories, watches FS events, computes per-file status | `Frameworks/scm/src/scm.cc`, `Frameworks/scm/src/fs_events.cc`, `Frameworks/scm/src/drivers/` |
| Networking | libcurl-based downloads, proxy resolution, TBZ bundle install, code-signature check | `Frameworks/network/src/download.cc`, `Frameworks/network/src/proxy.cc`, `Frameworks/network/src/filter_check_signature.cc` |
| Command-line client | `mate` opens files by talking to the app's socket | `Applications/mate/src/mate.mm` |
| Syntax service | XPC service exposing grammar/syntax operations to other apps | `Applications/SyntaxMate/src/main.mm`, `Applications/SyntaxMate/src/SyntaxMateImpl.mm` |

## Pattern Overview

**Overall:** Layered monorepo of ~60 native frameworks. C++ core libraries (no GUI dependency) under an Objective-C++ AppKit UI. Each framework is an independently built target described by its own `default.rave`, linked into `TextMate.app`.

**Key Characteristics:**
- Core data structures are pure C++ and are unit-tested with CxxTest (`Frameworks/*/tests/t_*.cc`). AppKit-facing code is in `.mm` files and is named with the `Oak` prefix (`OakTextView`, `OakAppKit`, `OakFoundation`).
- Objective-C is compiled with ARC (`-fobjc-arc`); C++ is `-std=c++2a`. Objective-C++ bridges the two layers, e.g. `OakDocumentEditor` holds a `std::unique_ptr<ng::editor_t>`.
- Process-wide services are Objective-C classes exposed through `@property (class, readonly) X* sharedInstance` (e.g. `BundlesManager`, `Preferences`, `OakDocumentController`, `SCMManager`, `FSEventsManager`, `KEventManager`, `Find`, `SoftwareUpdate`, `TMPlugInController`). At least 20 such singletons exist.
- Component communication is mostly direct calls, `NSNotificationCenter` notifications (`OakDocumentContentDidChangeNotification`, `OakDocumentDidSaveNotification`, etc. in `Frameworks/document/src/OakDocument.h`), and delegate interfaces (e.g. `command::delegate_t` in `Frameworks/command/src/runner.h`).
- Background work uses GCD queues (`org.textmate.scm.status`, `org.textmate.scm.info-cache` in `Frameworks/scm`; `org.textmate.write-html` in `Frameworks/OakCommand`), four `NSOperationQueue` instances, and one raw `std::thread`.

## Layers

**Platform primitives:**
- Purpose: Foundation types, CoreFoundation wrappers, string/encoding helpers, filesystem and process primitives.
- Location: `Frameworks/cf/src`, `Frameworks/text/src`, `Frameworks/io/src`, `Frameworks/OakSystem/src`, `Frameworks/OakFoundation/src`, `Frameworks/ns/src`, `Shared/include/oak/`
- Contains: `text::format`, `path::join`, `io::exec`, `io::create_pipe`, `oak::basic_tree_t`, `oak::application_t`
- Depends on: system libraries, CoreFoundation, Foundation
- Used by: every other framework

**Language and data services:**
- Purpose: Turn text and plists into grammars, scopes, settings, and themes.
- Location: `Frameworks/regexp` (wraps `vendor/Onigmo`), `Frameworks/scope`, `Frameworks/parse`, `Frameworks/bundles`, `Frameworks/settings`, `Frameworks/theme`, `Frameworks/plist`
- Contains: `regexp::pattern_t`, `scope::scope_t`, `parse::grammar_t`, `bundles::item_ptr`
- Depends on: `text`, `cf`, `io`, `plist`
- Used by: buffer, editor, layout, command, UI

**Editing core:**
- Purpose: Own text, selections, undo, edit commands, and geometry of the rendered text.
- Location: `Frameworks/buffer`, `Frameworks/selection`, `Frameworks/undo`, `Frameworks/editor`, `Frameworks/layout`, `Frameworks/command`, `Frameworks/document`
- Contains: `ng::buffer_t`, `ng::editor_t`, `ng::layout_t`, `ng::ranges_t`, `command::runner_t`, `OakDocument`
- Depends on: language services and platform primitives
- Used by: `OakTextView`, `DocumentWindow`, `BundleEditor`, `OakCommand`

**AppKit UI:**
- Purpose: Windows, views, menus, choosers, panels, and preferences.
- Location: `Frameworks/OakAppKit`, `Frameworks/OakTextView`, `Frameworks/OakTabBarView`, `Frameworks/DocumentWindow`, `Frameworks/FileBrowser`, `Frameworks/Find`, `Frameworks/OakFilterList`, `Frameworks/Preferences`, `Frameworks/BundleEditor`, `Frameworks/BundleMenu`, `Frameworks/MenuBuilder`, `Frameworks/CommitWindow`, `Frameworks/HTMLOutput`, `Frameworks/HTMLOutputWindow`, `Frameworks/SoftwareUpdate`
- Contains: `OakTextView`, `OakDocumentView`, `DocumentWindowController`, `FileBrowserViewController`, `OakTabBarView`, `OakFilterList`
- Depends on: editing core, language services
- Used by: `Applications/TextMate`

**Applications:**
- Purpose: Executables and bundles that link the frameworks.
- Location: `Applications/TextMate` (GUI), `Applications/mate`, `Applications/tm_query`, `Applications/bl`, `Applications/gtm`, `Applications/indent`, `Applications/pretty_plist`, `Applications/PrivilegedTool`, `Applications/SyntaxMate`, `Applications/QuickLookGenerator`, `Applications/NewApplication` (template)
- Depends on: frameworks as listed in each `default.rave`
- Used by: end users and the OS (QuickLook, XPC, launchd)

## Data Flow

### Primary Request Path (user types a character)

1. Key event arrives at `OakTextView` (`Frameworks/OakTextView/src/OakTextView.mm`), which maps it through Cocoa key bindings to an `ng::action_t` value (`Frameworks/editor/src/editor.h`).
2. `OakDocumentView` / `OakDocumentEditor` (`Frameworks/document/src/OakDocumentEditor.mm`) dispatches the action to `ng::editor_t` (`Frameworks/editor/src/editor.cc`).
3. `ng::editor_t` mutates `ng::buffer_t` (`Frameworks/buffer/src/buffer.cc`), which updates `ng::detail::storage_t` and the line index, then re-parses affected lines for scopes (`Frameworks/buffer/src/parsing.cc`, using `Frameworks/parse/src/parse.cc`).
4. The change is recorded by `Frameworks/undo/src/undo.cc` and the selection is updated in `Frameworks/selection/src/selection.cc`.
5. `OakDocument` posts `OakDocumentContentDidChangeNotification`; `OakTextView` invalidates and redraws using `ng::layout_t` (`Frameworks/layout/src/layout.cc`).

### Bundle Command Path (user runs ⌘R or a bundle command)

1. Command definition is looked up through `bundles::query` (`Frameworks/bundles/src/query.cc`) and parsed by `command::parser` (`Frameworks/command/src/parser.cc`).
2. `command::runner(...)` (`Frameworks/command/src/runner.mm`) builds the environment from `settings` (`Frameworks/settings/src/settings.cc`) and `TM_*` variables, then forks the child with `vfork()` and `io::create_pipe()` for stdin/stdout/stderr.
3. Output is delivered to a `command::delegate_t` (implemented by the document or the HTML output window) through `accept_result`, `show_document`, or `show_tool_tip`. HTML output goes through `Frameworks/HTMLOutput` and `Frameworks/HTMLOutputWindow`.

### External Open Path (`mate file.txt` or `rmate`)

1. `Applications/mate/src/mate.mm` computes `/tmp/textmate-<uid>.sock` (the `SUDO_UID` is used when run under sudo) and sends the open request.
2. The running app's `RMateServer` (`Applications/TextMate/src/RMateServer.mm`) binds the same AF_UNIX socket and dispatches each client to `OakDocumentController`.
3. The document is shown through `DocumentWindowController` (`Frameworks/DocumentWindow/src/DocumentWindowController.mm`). Remote `open` requests resolve `path`, `data`, or `uuid` arguments through `OakDocumentController.sharedInstance` (`Applications/TextMate/src/RMateServer.mm`).

### Privileged File Access

1. `Frameworks/file/src/open.cc` issues a `read_t` request through `oak::server_t` (`Frameworks/authorization/src/server.h`) when the user lacks permission.
2. `connect_to_auth_server()` opens `/var/run/com.macromates.auth_server.sock` (`Frameworks/authorization/src/constants.h`).
3. `Applications/PrivilegedTool/src/main.cc` (installed by `install.cc`, run via `launchd.cc` socket activation) performs the operation and returns the result over the socket.

### Startup

1. `main()` in `Applications/TextMate/src/main.mm` sets the support directory (`~/Library/Application Support/TextMate`), raises `RLIMIT_NOFILE` to 2048, installs `dispatch_source` handlers for SIGINT/SIGTERM, and clears any inherited `TM_*` environment variables.
2. `NSApplicationMain` runs `AppController` (`Applications/TextMate/src/AppController.mm`). It calls `[TMPlugInController.sharedInstance loadAllPlugIns:nil]` (line ~508) and, unless the session-restore user default is disabled, `[DocumentWindowController restoreSession]` (line ~575).

**State Management:**
- Per-document state lives on `OakDocument` (path, file type, encoding, SCM status) and `ng::buffer_t` (text, scopes, marks, spelling).
- Per-window state lives in `DocumentWindowController` (tabs, project layout, file browser).
- Global state is held in the `sharedInstance` singletons listed above and in user defaults. Session state is saved on quit and on SIGTERM (`DocumentWindowController saveSessionIncludingUntitledDocuments:`).

## Key Abstractions

**`oak::basic_tree_t`:**
- Purpose: Balanced (AA-tree) binary indexed tree with O(log N) prefix sums; backs text storage and layout.
- Examples: `Shared/include/oak/basic_tree.h`, documented in `INTERNALS.md`
- Pattern: Parameterized on `key_type` (must support `+`, `-`, default identity) and `value`; lookups take a 3-argument comparator (`search`, `offset`, `key`).

**`ng::buffer_t` and `ng::detail::storage_t`:**
- Purpose: Text storage with line index, scopes, spelling, marks, symbols.
- Examples: `Frameworks/buffer/src/buffer.h`, `Frameworks/buffer/src/storage.h`, `Frameworks/buffer/src/indexed_map.h`
- Pattern: Chunked storage with semantic services layered on top. `ng::indexed_map_t` is a segment map built on `basic_tree_t` (see `INTERNALS.md`).

**`ng::editor_t`:**
- Purpose: Interprets editing actions (`ng::action_t`) against a buffer and selection.
- Examples: `Frameworks/editor/src/editor.h`, `Frameworks/editor/src/editor.cc`
- Pattern: Single large action enum plus `switch` dispatch. Snippets and transforms are separate translation units (`snippets.cc`, `transform.cc`).

**`command::delegate_t` / `command::runner_t`:**
- Purpose: Run bundle commands and route their output to whatever view requested them.
- Examples: `Frameworks/command/src/runner.h`
- Pattern: Delegate interface with virtual methods (`write_unit_to_fd`, `accept_result`, `show_error`). The runner is held as `std::shared_ptr` and calls back through `enable_shared_from_this`.

**`OakDocument` / `OakDocumentController`:**
- Purpose: Model for files and untitled documents, and the registry of open documents.
- Examples: `Frameworks/document/src/OakDocument.h`, `Frameworks/document/src/OakDocumentController.h`
- Pattern: Objective-C facade over the C++ buffer. Uses `documentWithPath:`, `documentWithString:`, and `documentWithIdentifier:` factories and posts `NSNotification`s for changes.

**`oak::server_t<T>`:**
- Purpose: Asynchronous request/response dispatcher keyed by client ID.
- Examples: `Frameworks/authorization/src/server.h`, used in `Frameworks/file/src/open.cc`
- Pattern: Client registers a `T*` callback, receives `handle_request` results. Template parameters derive `request_t` from the callback type.

**Bundles (`bundles::item_t`):**
- Purpose: One item in a `.tmBundle` (grammar, theme, command, snippet, macro, etc.).
- Examples: `Frameworks/bundles/src/item.h`, `Frameworks/bundles/src/load.cc`, `Frameworks/bundles/src/locations.cc`
- Pattern: Items are searched in these locations (`Frameworks/bundles/src/locations.cc`): `~/Library/Application Support/TextMate`, its `Pristine Copy` and `Managed` subfolders, then `/Library/Application Support/TextMate` and its `Pristine Copy` subfolder.

## Entry Points

**GUI application:**
- Location: `Applications/TextMate/src/main.mm`
- Triggers: Launch of `TextMate.app`
- Responsibilities: Process setup, signal handling, `NSApplicationMain`

**Command-line client:**
- Location: `Applications/mate/src/mate.mm`
- Triggers: `mate` executable in `TextMate.app/Contents/MacOS/`
- Responsibilities: Opens files via the app socket; `-w` waits for close; `-` reads stdin

**Bundle-command tool client:**
- Location: `Applications/tm_query/src/tm_query.cc`, `Applications/bl/src/bl.cc`, `Applications/gtm/src/gtm.cc`, `Applications/indent/src/indent.cc`, `Applications/pretty_plist/src/`
- Triggers: Invoked by bundle commands or the user from the shell
- Responsibilities: Each performs one narrow task using `settings`, `bundles`, `plist`, or `text`

**Privileged helper:**
- Location: `Applications/PrivilegedTool/src/main.cc`
- Triggers: launchd socket activation (`MasterSocket`, `Applications/PrivilegedTool/src/launchd.cc`) or `install.cc` for installation
- Responsibilities: Root-level file operations on behalf of the app

**XPC syntax service:**
- Location: `Applications/SyntaxMate/src/main.mm`
- Triggers: `NSXPCListener serviceListener` connections
- Responsibilities: Exposes the `SyntaxMate` protocol

**QuickLook plug-in:**
- Location: `Applications/QuickLookGenerator/src/generate.mm`, `Applications/QuickLookGenerator/src/factory.c`
- Triggers: Finder QuickLook requests for TextMate-supported file types
- Responsibilities: Renders a document preview using `buffer`, `bundles`, `theme`, `scope`

**Test entry points:**
- Location: `bin/gen_test` (generates runners from `tests/t_*.cc` and `tests/gui_*.mm`), `Shared/include/test/`
- Triggers: `ninja <framework>/test` and `ninja <framework>/cxx_test` (targets created by `bin/rave`)
- Responsibilities: CxxTest-based unit and GUI tests

## Architectural Constraints

- **Threading:** Main thread owns all AppKit and document state. Background work uses GCD queues in `Frameworks/scm` (`org.textmate.scm.status`, `org.textmate.scm.info-cache`), `Frameworks/OakCommand` (`org.textmate.write-html`), and `NSOperationQueue` instances in the AppKit layer. Do not touch `ng::buffer_t` or `OakDocument` from background queues.
- **Global state:** Process-wide singletons via `sharedInstance` (see Key Characteristics). Module-level state includes the `oak::server_t` instance returned by `read_server()` in `Frameworks/file/src/open.cc` and the static socket path strings in `Applications/mate/src/mate.mm`. At startup the app unsets any inherited `TM_*` environment variables (`Applications/TextMate/src/main.mm`), so only variables TextMate sets itself reach bundle commands.
- **Circular framework dependencies:** The declared `require` edges in the `default.rave` files form five cycles, which the build system permits because it links them as one application. Known cycles:
  - `OakAppKit` → `file` → `command` → `OakAppKit`
  - `io` → `ns` → `plist` → `io`
  - `FileBrowser` → `OakCommand` → `FileBrowser`
  - `FileBrowser` → `OakCommand` → `HTMLOutput` → `FileBrowser`
  - `BundleEditor` → `OakTextView` → `Find` → `document` → `FileBrowser` → `OakCommand` → `BundleEditor`
  Any new `require` edge should be checked against these cycles before adding it. Breaking one cycle usually means moving a shared type into a lower layer such as `text` or `OakFoundation`.
- **Shared prelude:** `Shared/PCH/prelude.*` is force-included into every target (`prelude Shared/PCH/prelude.*` in root `default.rave`), so symbols defined there are global across the codebase.
- **Compile-time macros:** `NULL_STR` (`"￿"`) and `REST_API` (`https://api.textmate.org`) are defined on the command line for all targets (root `default.rave`).
- **Platform floor:** `APP_MIN_OS = 10.12`, set in root `default.rave`.

## Anti-Patterns

### Oversized translation units

**What happens:** Several UI files are very large: `Frameworks/OakTextView/src/OakTextView.mm` (4,625 lines), `Frameworks/DocumentWindow/src/DocumentWindowController.mm` (2,885), `Frameworks/FileBrowser/src/FileBrowserViewController.mm` (2,328), `Frameworks/document/src/OakDocument.mm` (1,866), `Frameworks/editor/src/editor.cc` (1,548).
**Why it's wrong:** Mixed responsibilities in one file make the cyclic dependencies above harder to untangle and increase the chance of unrelated regressions.
**Do this instead:** Put new behavior in a focused `.mm`/`.cc` pair in the existing framework (for example, a new editor action goes in its own file under `Frameworks/editor/src/` and is registered in the `ng::action_t` enum in `editor.h`).

### Reaching across layers through singletons

**What happens:** Views call `X.sharedInstance` directly (e.g. `BundlesManager`, `Preferences`, `OakDocumentController`) instead of receiving the dependency.
**Why it's wrong:** This creates hidden coupling and is the main source of the cycles listed above.
**Do this instead:** For new code, pass the dependency through the constructor or a delegate protocol (as `command::delegate_t` does in `Frameworks/command/src/runner.h`).

## Error Handling

**Strategy:** Fail fast for programmer errors using `oak/debug.h` assertions; report user-facing errors through alert panels (`OakRunIOAlertPanel`, `Frameworks/OakAppKit/src/IOAlertPanel.h`); crash reporting via `crash_reporter_info_t` (`Frameworks/crash/src/info.h`).

**Patterns:**
- Top-level `main()` catches `std::exception` and reports via `crash_reporter_info_t` (`Applications/TextMate/src/main.mm`).
- Bundle command failures go to `command::delegate_t::show_error` with the exit code and stderr.

## Cross-Cutting Concerns

**Logging:** `os_log(OS_LOG_DEFAULT, ...)` for lifecycle events (`Applications/TextMate/src/main.mm`); `oak/debug.h` macros for debug builds (debug builds link `OakDebug`, `Frameworks/OakDebug/src`).

**Validation:** No central validation layer was found. Checks live in the parsers that consume the data, e.g. `Frameworks/settings/src/parser.cc` for `.tm_properties` and `Frameworks/file/src/encoding.cc` for file encodings.

**Authentication:** Code signing and notarization are handled at build time by `bin/rave` (`signature`, `notarize` targets). At run time, privileged file operations authorize through `Frameworks/authorization` and the launchd helper. Downloads pass through the signature-check filter in `Frameworks/network/src/filter_check_signature.cc`.

**Configuration:** `.tm_properties` files are read from the document's directory up to `$HOME` (`Frameworks/settings/src/settings.cc`, `paths()`). Build configuration comes from `local.rave` (generated by `configure`) and the root `default.rave`.

---

*Architecture analysis: 2026-10-19*
