---
last_mapped_commit: 346b52b108b387462d4b3def481fb74983ae89f3
last_mapped_at: 2026-10-19
---
# Coding Conventions

**Analysis Date:** 2026-10-19

## Scope and Language Mix

The codebase is macOS-only C++17/C++2a, Objective-C (ARC), Objective-C++ (`.mm`), and a little C. Conventions below are derived from the sources under `Frameworks/`, `Applications/`, `Shared/`, and `PlugIns/`. Build flags live in `default.rave` (`CXX_FLAGS "-std=c++2a"`, `OBJC_FLAGS "-fobjc-abi-version=3 -fobjc-arc"`).

Rough file counts (tracked, `git ls-files`): ~210 `.cc`, ~290 `.h`, ~160 `.mm`, a handful of `.m`/`.c`. The `.rave` build descriptors (one per target) are Ruby-like DSL files, not code.

**No formatter or linter is configured.** There is no `.clang-format`, `.editorconfig`, `.swiftlint`, `.pylintrc`, or lint step in CI. Style is enforced by review and by copying the surrounding file. Match the file you are editing.

## Indentation and Whitespace

**Tabs, never spaces, for indentation.**
- 519 of 659 C/C++/ObjC source files under `Frameworks/`, `Applications/`, `Shared/` are tab-indented. Zero files use 4-space or 2-space block indentation.
- `.tm_properties` sets `tabSize = 3` and `softTabs = false` for `source.c++, source.objc++`. Keep that in mind when aligning.

**Alignment uses tabs or spaces inside a line:**

```cpp
// Tab before a short statement that follows an `else`, as in Frameworks/text/src/trim.cc
if(last == std::string::npos)
		last = str.size();
else	++last;

// Column alignment of consecutive assignments uses spaces
std::string::size_type first = str.find_first_not_of(trimChars);
std::string::size_type last  = str.find_last_not_of(trimChars);
```

**No trailing whitespace is the goal** (not enforced by tooling). The user's global rule for Bash/Python trailing-space stripping does not apply to this repo's sources.

**Line length:** no enforced limit. About 1,194 C++ lines and 1,969 Objective-C++ lines exceed 120 columns. Do not reflow existing long lines in unrelated edits. For new code, keep lines readable and, when wrapping a long argument list, put one argument per line with a tab-indented continuation.

## Braces and Control Flow

**Allman style: opening brace on its own line for functions, classes, namespaces, blocks, and control structures.**
- 3,581 own-line `{` vs 18 same-line `) {` in `.cc` files.

```cpp
namespace text
{
	std::string trim (std::string const& str, std::string const& trimChars)
	{
		std::string::size_type first = str.find_first_not_of(trimChars);
		if(first == std::string::npos)
			first = str.size();
		return first;
	}

} /* text */
```

**No space between a control keyword and its parenthesis.** Function declarations and definitions DO get a space before the parameter list.
- `if(`, `for(`, `while(`, `switch(`, `catch(` appear ~8,100 times without space and 8 times with space.
- Function definitions: `bool save (bool useDeltaIfNonLocal = true);` in `Frameworks/bundles/src/item.h`.
- Function calls: `text::trim(str, chars)` with no space.

**Braces may be omitted for single-statement bodies.** This is common (`if(first == npos)` followed by an unbraced statement on the next line in `Frameworks/text/src/trim.cc`). Add braces when the body spans more than one statement or a comment would be ambiguous.

**`else` goes on its own line, aligned with its `if`.** `else` on the same line as `}` is rare (10 occurrences). The exception is a one-liner `else` followed by a tab, as shown above.

**Closing braces for namespaces carry a comment with the namespace name:**

```cpp
} /* bundles */
```

Found in ~219 places. Anonymous namespaces are written as `namespace` on one line and `{` on the next (`Frameworks/file/src/save.cc`).

## Naming Patterns

**Files:**
- C++ `.cc`/`.h`: lowercase `snake_case`, one unit per file: `Frameworks/text/src/trim.cc`, `Frameworks/io/src/path.h`, `Frameworks/bundles/src/item.h`.
- Objective-C `.mm`/`.h`: `PascalCase` matching the class name: `Frameworks/OakTabBarView/src/OakTabBarView.mm`, `Frameworks/document/src/OakDocument.mm`.
- Category files use a space in the name (existing only): `Frameworks/OakFoundation/src/NSString Additions.mm`, `Frameworks/OakAppKit/src/NSImage Additions.h`. Do not copy this for new files.
- Umbrella headers named after the framework: `Frameworks/bundles/src/bundles.h` includes the module's public headers.

**Types:**
- C++ `struct`/`enum`/`typedef` names use `snake_case` with a `_t` suffix: `item_t` (`Frameworks/bundles/src/item.h`), `info_t` (`Frameworks/scm/src/scm.h`), `kind_t`, `scope::selector_t`. About 376 `struct *_t` declarations.
- Smart-pointer aliases are `typedef`s with a `_ptr` suffix next to the type: `typedef std::shared_ptr<item_t> item_ptr;`. Weak variants use `_weak_ptr` (`weak_info_ptr`). See `Frameworks/scm/src/scm.h`.
- Objective-C classes and protocols use a project prefix: `OakTabBarView`, `OakTabItem`, `TMFileReference`, `OakDocument`. Keep the prefix; do not invent new unprefixed classes.

**Functions and methods (C++):**
- `snake_case`, no `get_` prefix. Getters are named after the value: `item->name()`, `item->paths()`, `item->kind()`. Setters are `set_*`: `set_name`, `set_parent_menu`, `set_plist`.
- Predicates read as English: `disabled()`, `hidden_from_user()`, `tracks_directories()`, `scm_enabled_for_path()`.
- Free functions in a namespace follow the same rule: `text::trim`, `path::normalize`, `scm::wait_for_status`.

**Functions and methods (Objective-C):**
- Methods are `lowerCamelCase`: `reloadData`, `performClose:`, `tabBarView:shouldSelectIndex:`. Delegate/data-source methods follow Cocoa naming: `- (BOOL)tabBarView:(OakTabBarView*)aTabBarView shouldSelectIndex:(NSUInteger)anIndex;` from `Frameworks/OakTabBarView/src/OakTabBarView.h`.
- Factory methods use `+ (instancetype)tabItemWithTitle:...`.

**Variables and parameters:**
- Parameters in C++ are mixed. Newer code uses `lowerCamelCase`: `trimChars`, `includeDisabledItems`, `shouldInitialize`, `useDeltaIfNonLocal`, `newName` (about 270 camelCase parameter declarations vs 31 snake_case). Use `lowerCamelCase` for new parameters and locals unless the surrounding function is clearly snake_case.
- Objective-C parameters are prefixed with `a`/`an` as in `aTabBarView`, `anIndex`, `aTitle` (about 120 occurrences). Keep this convention for ObjC methods.
- Member variables in C++ are prefixed with an underscore: `_paths`, `_fields`, `_plist_mutex`, `_scope_selector` (about 378 declarations). The underscore is required for class members; do not use trailing underscores.
- Objective-C ivars also use a leading underscore (`_title`, `_path`, `_identifier` in `Frameworks/OakTabBarView/src/OakTabBarView.mm`).

**Constants and enums:**
- Named constants use a `k` prefix followed by `CamelCase`: `kItemTypeCommand`, `kFieldName`, `kStateIdle`, `kSeparatorUUID` (about 800 uses).
- Global string-key constants are declared `extern` in the header and defined in the `.cc`: `extern std::string const kFieldName;` in `Frameworks/bundles/src/item.h`.
- Objective-C string constants: `static NSString* const OakTabItemPasteboardType = @"...";` and `static NSString* kUserDefaultsTabItemMinWidthKey = @"tabItemMinWidth";` in `Frameworks/OakTabBarView/src/OakTabBarView.mm`.
- Enums are unscoped and carry the `kind_t` suffix. `enum class` is used only in 3 places; prefer the unscoped form for consistency unless a scoped enum is needed to avoid name collisions.

**Macros:**
- `UPPER_SNAKE_CASE`, with the `OAK_` prefix for project macros: `OAK_ASSERT`, `OAK_DEBUG_VAR`, `OAK_ITERATOR_MACROS_H_*`.
- Two assertion families exist. Use the right one (see Error Handling).
- Include guards are not macros in the public-API sense but follow the same uppercase form (see Module Design).

**Namespaces:** lowercase, no suffix: `text`, `bundles`, `scm`, `io`, `path`, `plist`, `scope`, `ng` (editor/buffer), `oak`, `test`.

**Test identifiers:** `test_<behavior>`, `setup_<name>`, `benchmark_<name>`. See TESTING.md.

## Header Files

**Include guards, not `#pragma once`.** 0 `#pragma once` uses. Guard format:

```cpp
#ifndef TEXT_TRIM_H_5NIIFUM8
#define TEXT_TRIM_H_5NIIFUM8

namespace text
{
	std::string trim (std::string const& str, std::string const& trimChars = " \t\r\n");

} /* text */

#endif /* end of include guard: TEXT_TRIM_H_5NIIFUM8 */
```

- Pattern: `<MODULE>_H_<8 random uppercase alphanumerics>`. Generate a fresh 8-character suffix for each new header; do not reuse one from another file.
- Closing comment is always `/* end of include guard: <NAME> */`.
- Roughly 290 `.h` files exist in `Frameworks/` and `Shared/`. The unguarded headers are Objective-C headers (use `#import` and `@interface`) plus a few C++ headers that should be fixed when next touched, e.g. `Frameworks/text/src/transcode.h`, `Frameworks/MenuBuilder/src/MenuBuilder.h`, `Frameworks/Find/src/CommonAncestor.h`.

**Declarations in headers, definitions in `.cc`/`.mm`.** Trivial accessors may be defined inline in the class body with aligned braces:

```cpp
bool disabled () const         { return _disabled; }
bool hidden_from_user () const { return _hidden_from_user; }
```

(`Frameworks/bundles/src/item.h`.) Align the `{ ... }` bodies across consecutive one-liners.

**Visibility:** `private:` section lists `_member` fields and private helper structs. Public API first.

**Deleted copy operations** are explicit: `info_t (info_t const& rhs) = delete;` (`Frameworks/scm/src/scm.h`).

## Include Order

**C++ (`.cc`, `.h`):**
1. The file's own header first, in quotes: `#include "trim.h"` (`Frameworks/text/src/trim.cc`).
2. Project headers. Same-module headers use quotes; cross-framework headers use angle brackets with the framework name as the root: `#include <text/tokenize.h>`, `#include <oak/debug.h>`, `#include <authorization/server.h>`.
3. Standard and system headers last, or interleaved with project headers. No blank lines between groups in the observed files (`Frameworks/file/src/save.cc`).

Example from `Frameworks/file/src/save.cc`:

```cpp
#include "save.h"
#include "status.h"
#include "encoding.h"
#include "filter.h"
#include <authorization/server.h>
#include <io/intermediate.h>
#include <text/trim.h>
#include <oak/debug.h>
```

**Objective-C / Objective-C++ (`.mm`, `.h`):** use `#import` only. About 1,016 `#import` vs 18 `#include` in `.mm` files. The own header is first:

```objc
#import "OakTabBarView.h"
#import <OakAppKit/NSImage Additions.h>
#import <OakFoundation/OakFoundation.h>
#import <oak/misc.h>
```

**Shared precompiled prelude.** `default.rave` adds `prelude Shared/PCH/prelude.*`, which pulls in `<algorithm>`, `<map>`, `<memory>`, `<string>`, `<vector>`, `<thread>`, `<mutex>`, `<functional>`, `<boost/variant.hpp>`, `<boost/crc.hpp>`, `<sparsehash/dense_hash_map>`, and `Shared/PCH/prelude-mac.h` (Carbon, CoreFoundation, `os/log.h`, and others). Do not re-include these in every file. Include only what is not in the prelude when the file is small and self-contained, or when the file is reused outside the app build (tests and tools).

**Include paths.** `default.rave` adds `-IShared/include`. Framework public headers are copied to `_Include/<framework>/include` by the build (see `.tm_properties`, `TM_FRAMEWORK_INCLUDE`). A header is public only if it is listed in that framework's `default.rave` `headers` line.

## Formatting Idioms (Observed)

**Pointers and references: east-const.**
- `std::string const& name`, `char const* value`, `oak::uuid_t const& uuid () const` (about 422 east-const uses, 0 west-const).
- Pointer declarations hug the type in both C++ and ObjC: `NSString* title`, `std::string* error`.

**Null values:**
- C++ pointers: prefer `nullptr` for new code (279 uses), though `NULL` (184 uses) is still present in older files.
- Empty-sentinel strings: `NULL_STR`. It is a preprocessor define set in `default.rave` as `-D'NULL_STR="￿"'`. Use it as the default for "not provided" string parameters: `std::string const& name = NULL_STR`. Test with `== NULL_STR` / `!= NULL_STR`, never with `.empty()`.

**Type deduction and modern C++:**
- `auto` is used widely (about 1,300 occurrences), including range-for (`for(auto const& test : tests)`).
- `std::optional` is used for "maybe a value" returns (`std::optional<double> does_match (...)` in `Frameworks/bundles/src/item.h`).
- `std::shared_ptr` is the default owning pointer (about 90 uses). `unique_ptr` is rare (9); raw `new` (64 uses) is legacy and should not be added to.
- Containers default to `std::map`/`std::vector`/`std::set` (`std::map` appears ~550 times); `emplace`/`emplace_back` is preferred over `push_back` of temporaries (302 `emplace` uses).

**Ternaries and multi-line expressions:** single-line where short. Break before `:` and `?` when wrapping. Example: `return first <= last ? str.substr(first, last - first) : "";`.

**Inline one-line functions:** allowed in headers when aligned; see Header Files.

**Long argument lists:** many signatures stay on one line (e.g. the `file_context_t` constructor in `Frameworks/file/src/save.cc`). When wrapping, place one argument per line with a tab-indented continuation.

**`#pragma mark`:** rare (4 uses). Objective-C section banners are preferred (see Comments).

## Objective-C and Objective-C++ Conventions

- **ARC is mandatory.** `-fobjc-arc` in `default.rave`. Zero `retain`/`release` calls. Use `__weak` for delegates and back-references: `@property (nonatomic, weak) id <OakTabBarViewDelegate> delegate;` (37 `__weak` uses).
- **Properties** are declared with explicit attributes: `@property (nonatomic, readonly)`, `@property (nonatomic, getter = isModified) BOOL modified;`. Declare public properties in the `.h`, private ones in a class extension `@interface Foo ()` inside the `.mm` (`Frameworks/OakTabBarView/src/OakTabBarViewController.mm`).
- **Initializers** follow the `if(self = [super init])` shape:

```objc
- (instancetype)initWithTitle:(NSString*)title path:(NSString*)path identifier:(NSString*)identifier modified:(BOOL)modified

{
	if(self = [super init])
	{
		_title       = title;
		_path        = path;
		_identifier  = identifier;
		_modified    = modified;
	}
	return self;
}
```

- **Autorelease pools:** `@autoreleasepool` is used in background loops (9 occurrences). Add one inside loops that allocate many objects.
- **Blocks** are used as callbacks (298 `^` occurrences; `__block` 47). Prefer blocks for one-shot callbacks; use `std::function` (34 uses) for C++-only APIs.
- **Use `NSLog` in UI/Objective-C code only when the surrounding file already does** (see Logging).
- **Static helper functions** in `.mm` files are `static` at file scope, not private methods (e.g. `static void DisableImplicitAnimationForBlock (void(^handler)())` in `Frameworks/OakTabBarView/src/OakTabBarView.mm`).
- **Protocols** are declared in the public header next to the class they serve (`@protocol OakTabBarViewDelegate <NSObject>` with `@optional` section).
- **Exceptions:** `@throw [NSException ...]` is used only for unsupported accessibility attributes (`Frameworks/OakTextView/src/OakTextView.mm`). Do not introduce new ObjC exception use for control flow.

## Code Style Specifics

- **Strings:** user-visible strings and comments use typographic quotes and dashes: `‘%@’` in `NSLog`/UI strings, `“surrounding words”`, em dash `—`. Keep them when editing neighboring lines; do not normalize to ASCII quotes.
- **Printf-style formatting:** prefer `text::format("...%s...", ...)` (`Frameworks/text/src/format.h`) for `std::string` results. Use `fprintf`/`os_log` only for logging.
- **Static analysis flags** (`default.rave`): `-Wall -Wwrite-strings -Wformat -Winit-self -Wmissing-include-dirs`, with `-Wno-parentheses -Wno-sign-compare -Wno-switch -Wno-c99-designator`. Keep new code warning-free under `-Wall`.
- **Unsigned char:** the build uses `-funsigned-char`. Do not write code that assumes signed `char`.
- **Debug vs release:** `config debug` adds `-fsanitize=address` and `-Os`; `config release` adds `-DNDEBUG -flto=thin`. Code must compile with `NDEBUG` defined (see Error Handling).

## Import and Module Organization

- Each framework is a directory `Frameworks/<name>/` with `src/`, optional `tests/`, optional `resources/`, and a `default.rave` target.
- Public headers are declared in `default.rave`:

```
target "${dirname}" {
	headers src/*.h
	sources src/*.cc
	tests tests/*.cc
	frameworks CoreFoundation
	libraries iconv
}
```

(`Frameworks/text/default.rave`.) Only headers listed here are exported. Private headers may sit next to the sources without being listed.
- Dependencies are declared by name with `require`: `require bundles io ns parse regexp scope text` (`Frameworks/buffer/default.rave`).
- Umbrella headers (`Frameworks/bundles/src/bundles.h`) include a module's public headers and are guarded like any other header.

## Error Handling

**C++ core: return values and out-parameters, not exceptions.**
- Functions that can fail return `bool` (or a value) and write details to `std::string& error`:

```cpp
// Frameworks/network/src/download.h
virtual bool receive_end (std::string& error) { return true; }

// Frameworks/file/src/save.cc
int fd = dest.open(&error);
if(fd == -1)
	;
else if(write(fd, bytes->get(), bytes->size()) != bytes->size())
	error = text::format("write: %s", strerror(errno));
```

(28 `std::string& error` out-parameters.)
- Check `errno` immediately after the failing system call and format it with `strerror(errno)` through `text::format`.
- Use `std::optional` for "absent" results when there is no error text to report (19 uses).
- Use `NULL_STR` to mean "no value" in string arguments and returns; check with `== NULL_STR`.

**Exceptions** exist but are rare (about 6 `throw` statements in the tree). Where they are used, they are caught at a boundary with `catch(std::exception const& e)`:
- `Applications/TextMate/src/main.mm:79`, `Frameworks/encoding/src/encoding.mm:159`, `Frameworks/OakTextView/src/OakTextView.mm:2204`, `Frameworks/plist/src/fs_cache.cc:92`.
- Test failures use `oak_assertion_error`, which throws `oak_exception` (see TESTING.md).

**Assertions: two families. Use the right one.**
- **`ASSERT`, `ASSERT_EQ`, `ASSERT_LT`, ... `ASSERTF`** (`Frameworks/OakDebug/src/OakAssert.h`): runtime invariant checks in product code. Compiled out when `NDEBUG` is defined (release builds). About 173 uses in `src/`. Use them to document invariants that the caller must satisfy, not to validate user input.
- **`OAK_ASSERT`, `OAK_ASSERT_EQ`, ... `OAK_FAIL`, `OAK_WARN`** are test-only macros defined by the test generator `bin/gen_test`. Use them in `tests/` only.
- Do not use `assert()` from `<cassert>` (not used in the tree).

**Fail fast.** The project favors stopping on a broken invariant over continuing silently. Examples: `abort()` after setup failure in `Frameworks/network/tests/t_download.cc`; `OAK_FAIL("no SCM info for path: " + jail.path())` in `Frameworks/scm/tests/t_git.cc`.

**Objective-C errors:** pass `NSError*` where Cocoa APIs expect it (93 mentions); check the return value first, then the error.

## Logging

**Primary logger: unified logging via `os_log`.** Define the logger once per call site with `OS_LOG_DEFAULT`:

```cpp
os_log_error(OS_LOG_DEFAULT, "Malformed symbol transformation at offset %td (expected ‘/’): %{public}s", it - src.data(), src.c_str());
```

- `os_log_error` (170 uses), `os_log_info` (21), `os_log_debug` (5), `os_log` (1). Examples: `Frameworks/buffer/src/symbols.cc:39`, `Frameworks/authorization/src/authorization.h:60`.
- Mark user-data and paths with `%{public}s`; without it, unified logging redacts the value.
- `os_log` and `os/log.h` are included through `Shared/PCH/prelude-mac.h`; do not add a new include.

**Objective-C UI code:** `NSLog(@"...")` appears about 42 times, e.g. `Frameworks/BundlesManager/src/BundlesManager.mm:609`. Keep `NSLog` in files that already use it; for new Objective-C code prefer `os_log_error` for consistency with the C++ layer.

**Stderr for tools and tests:** `fprintf(stderr, "*** ...")` (88 uses), mostly in test setup and CLI helpers (`Frameworks/network/tests/t_download.cc`, `bin/gen_test`). Prefix fatal test-setup messages with `***`.

**Debug tracing (dormant):** `OAK_DEBUG_VAR(name)` and `D(DBF_name, code)` in `Frameworks/OakDebug/src/OakDebugLog.h`, enabled at run time through the `DebugFlags` environment variable (`Frameworks/OakDebug/src/OakDebugLog.cc`). The `bug(format, ...)` macro prints file, function, and line. Both are compiled out when `NDEBUG` is set. Use these for developer-only tracing; use `os_log_debug` for anything that should survive into a log.

**Do not log secrets or credentials.** Redact tokens and keys in any new log line.

## Comments

**Comment the why, not the what.** Inline comments explain non-obvious constraints, platform quirks, or ordering requirements. Do not restate the code on the line above it.

**Section banners** separate logical groups inside large files (1,486 occurrences). Use this exact form, matching the width of the title:

```objc
// ==============
// = OakTabItem =
// ==============
```

(`Frameworks/OakTabBarView/src/OakTabBarView.mm`.)

**TODO / FIXME:** write `// TODO` followed by the text, with no colon (about 43 `TODO` and 21 `FIXME` in the tree). Example: `// TODO We should delay checking to the misspellings function.` in `Frameworks/buffer/src/spelling.cc:68`. Add an owner or issue reference only when one exists.

**Doc comments:** not used. `///` appears once and `/** */` never. Document public APIs in the header with short `//` comments next to the declaration when the name is not self-explanatory, as in `Frameworks/bundles/src/item.h`:

```cpp
std::string _executable; // e.g. ‘git’
std::string _variable;   // e.g. ‘TM_GIT’
```

**Commented-out code:** `#if 0` blocks exist (2 uses: `Frameworks/OakDebug/src/OakAssert.h:28`, `Frameworks/OakAppKit/tests/gui_dictionary.mm:45`). Do not add new ones; delete unused code.

## Function Design

- **Size:** no enforced limit. Large files exist: `Frameworks/OakTextView/src/OakTextView.mm` (~4,600 lines), `Frameworks/DocumentWindow/src/DocumentWindowController.mm` (~2,900), `Frameworks/FileBrowser/src/FileBrowserViewController.mm` (~2,300). Prefer splitting new behavior into a helper in a new file over extending these.
- **Parameters:** pass `std::string const&` for strings, `std::vector<...> const&` for lists, and `oak::uuid_t const&` for IDs. Pass by value only for cheap types or shared pointers (`item_ptr`). Use default arguments for optional flags (`bool includeDisabledItems = false`).
- **Output parameters:** `std::string& error` or `double* progress` (see `Frameworks/network/src/download_tbz.h`). Mark const-ness on read-only pointers: `bool const* stopFlag = NULL`.
- **Return values:** `bool` for success/failure with an error out-parameter; `std::optional<T>` for optional values; `std::pair<size_t, std::string>` for "index plus payload" results (`Frameworks/buffer/src/buffer.h`).
- **Callbacks:** ObjC blocks `void (^block)(info_t const&)` in C++ headers (`Frameworks/scm/src/scm.h`), or `std::function` for pure C++ code.
- **Lambdas:** `[&]` and `[=]` are both used (84 occurrences). Prefer `[&]` for synchronous helpers; capture explicitly when a lambda outlives the current scope.
- **Concurrency:** `std::mutex` with `std::lock_guard`/`std::unique_lock` (38 uses); GCD via `dispatch_async` and friends (186 uses) in ObjC code. Protect shared state with a named mutex member (`mutable std::mutex _plist_mutex;` in `Frameworks/bundles/src/item.h`).

## Module Design

**Exports:** a header is the module's public surface. Declare free functions in a namespace, classes as `struct` with `public` members by default, and keep helpers `static` in the `.cc` or in an anonymous namespace.

**Barrel / umbrella headers:** one per framework, named after the framework, that includes the public headers in dependency order. Example: `Frameworks/bundles/src/bundles.h`.

**Anonymous namespaces** wrap file-local types and helpers in `.cc` files:

```cpp
namespace
{
	struct file_context_t : file::save_context_t
	{
		...
	};
}
```

(`Frameworks/file/src/save.cc`.) Use `static` for file-local functions that do not need a type.

**Cross-framework includes** use the framework name as the root: `#include <scope/scope.h>`, `#include <plist/plist.h>`, `#include <text/format.h>`. Never use `../` to reach another framework.

## Commit Messages (for completeness)

From `CONTRIBUTING.md` and recent history on `develop`:
- Summary line under 70 characters, imperative mood, no trailing period: `Remove image, modified, and SCMStatus properties from FileItem`.
- Blank line, then reasoning if the change is not obvious.
- One commit does one thing. Fix-ups use `fixup!` plus the target summary line.
- Include Xcode-saved `.xib` changes in a separate `Save xib file with Xcode «version»` commit.

---

*Convention analysis: 2026-10-19*
