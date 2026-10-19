# Phase 2: Launch and Run Verification - Pattern Map

**Mapped:** 2026-10-19
**Files analyzed:** 7 BUILD-05 call sites (conditional on a RUN failure), plus 3 compile-only sites
**Analogs found:** 2 / 7 with an in-repo modern-API precedent

Phase 2 changes code only when a RUN test fails. Test artifacts go in `~/runtests/` (outside the repo). The only in-repo file this phase writes is `02-RUN-LOG.md`.

## File Classification

| Call site (file:line) | Role | Data Flow | Test | Closest Analog | Match Quality |
|---|---|---|---|---|---|
| `Frameworks/HTMLOutput/src/OakHTMLOutputView.mm:199` (NSURLConnection) | WebView policy delegate | request-response | RUN-04 | `Frameworks/SoftwareUpdate/src/SoftwareUpdate.mm`, `Frameworks/CrashReporter/src/CrashReporter.mm` (NSURLSession) | role-match (partial) |
| `Frameworks/SoftwareUpdate/src/SoftwareUpdate.mm:557` (NSTask.launchPath) | task launcher, UI action | event-driven | RUN-01 (see gap) | `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm:109` (same deprecated `launchPath`) | none for `executableURL` |
| `Shared/include/oak/compat.h:8-16` (Gestalt) | utility | request-response | RUN-01, RUN-05 | `Applications/TextMate/src/AppController.mm:490`, `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm:303` (NSProcessInfo) | exact |
| `Frameworks/io/src/resource.cc:34` (Get1Resource) | utility | file-I/O | RUN-01 | none in repo | no analog |
| `Frameworks/TMFileReference/src/TMFileReference.mm:257-260` (GetIconRef) | utility | file-I/O | RUN-02 (file icon) | none in repo | no analog |
| `Frameworks/OakCommand/src/OakCommand.mm:679` (WebView URL scheme registration) | provider (`+load`) | event-driven | RUN-01 | none in repo | no analog |
| `Applications/TextMate/src/ODBEditorSuite.mm` (Apple Events, AE* calls) | handler | event-driven | not run by RUN-01 to RUN-05 | none (same file) | no analog |

Compile-only sites (no change planned; check the build output and RUN-01 for the mate install prompt): `Shared/include/oak/compat.h:18-21`, `Frameworks/Preferences/src/TerminalPreferences.mm:50`, `Frameworks/authorization/src/server.cc:26`.

## Pattern Assignments

### `Frameworks/HTMLOutput/src/OakHTMLOutputView.mm:199` (request-response, RUN-04)

**Existing call** (lines 197-199), inside the WebView policy delegate:
```objc
- (void)webView:(WebView*)sender decidePolicyForNavigationAction:(NSDictionary*)actionInformation request:(NSURLRequest*)request frame:(WebFrame*)frame decisionListener:(id <WebPolicyDecisionListener>)listener
{
	if([NSURLConnection canHandleRequest:request])
```
**Migration analog:** `NSURLSession` precedent in `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm` and `Frameworks/CrashReporter/src/CrashReporter.mm`. No repo code maps `canHandleRequest` directly to a `NSURLSession` call. Keep the branch behavior the same.
**Test:** RUN-04 fires this on every navigation in the HTML output window, so a link click in the output exercises it.

### `Frameworks/SoftwareUpdate/src/SoftwareUpdate.mm:557` (event-driven, RUN-01)

**Existing call** (lines 556-557), inside `- (void)takeURLToInstallFrom:(NSButton*)sender` (line 535):
```objc
NSTask* task = [[NSTask alloc] init];
task.launchPath     = @"/bin/sh";
```
**Migration analog:** none. `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm:109` also uses `launchPath`, so no repo code uses `executableURL`.
**Gap:** this site runs only when the user clicks to install an update. The RUN-01 launch path does not reach it. The planner must either add a step that triggers it, or record it as untested.

### `Shared/include/oak/compat.h:8-16` (utility, request-response, RUN-01 and RUN-05)

**Existing call** (lines 6-16), with a suppression block:
```cpp
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
	inline size_t get_gestalt (OSType selector)
	{
		SInt32 res;
		return Gestalt(selector, &res) == noErr ? res : 0;
	}
```
**Migration analog:** `Applications/TextMate/src/AppController.mm:490`, and `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm:303`:
```objc
NSOperatingSystemVersion osVersion = NSProcessInfo.processInfo.operatingSystemVersion;
```
Note: `compat.h:18-21` (`AuthorizationExecuteWithPrivileges`) sits inside the same suppression block (lines 6-22). Changing `Gestalt` must not touch it.

### `Frameworks/io/src/resource.cc:34` (file-I/O, RUN-01)

**Existing call** (line 34): `if(Handle handle = Get1Resource(theType, theID))`, inside a `FSOpenResFile` block. No deprecation pragma at this site. Keep the resource-fork open/close pairing unchanged.

### `Frameworks/TMFileReference/src/TMFileReference.mm:257-260` (file-I/O, RUN-02)

**Existing call** (lines 257-260), with a suppression block (`push` at 257, `pop` at 266):
```objc
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
				IconRef iconRef;
				if(GetIconRef(kOnSystemDisk, kSystemIconsCreator, kAliasBadgeIcon, &iconRef) == noErr)
```
**Migration analog:** none in repo. Keep the suppression block as the default if the site is left alone.

### `Frameworks/OakCommand/src/OakCommand.mm:679` (event-driven, RUN-01)

**Existing call** (lines 676-679), in `+load`:
```objc
@implementation OakFileHandleURLProtocol
+ (void)load
{
	[self registerClass:self];
	[WebView registerURLSchemeAsLocal:kOakFileHandleURLScheme];
}
```
**Migration analog:** none. This runs at process load, so RUN-01 covers it. Migrate only if RUN-01 or RUN-04 fails.

### `Applications/TextMate/src/ODBEditorSuite.mm` (event-driven, Apple Events)

**Existing calls:** `AEDuplicateDesc`, `AEGetDescData`, `AECoerceDesc`, `AEGetNthDesc`, `AEGetParamDesc`, `AEDisposeDesc` (lines 16-62). The file has no `-Wdeprecated-declarations` pragma. It is referenced from `Applications/TextMate/src/AppController Documents.mm`. The file is tracked and in the build by that reference. No RUN-01 to RUN-05 test sends an ODB Apple Event, so the planner should record this as untested unless a test covers it.

## Shared Patterns

### Deprecation suppression block
**Source:** `Shared/include/oak/compat.h:6-22`, `Frameworks/TMFileReference/src/TMFileReference.mm:257-266`
**Apply to:** Any deprecated call left in place (Gestalt, Get1Resource, GetIconRef, AuthorizationExecuteWithPrivileges)
```cpp
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
/* deprecated call */
#pragma clang diagnostic pop
```

### Commits and logs (process, not code)
**Source:** `.planning/phases/02-launch-and-run-verification/02-CONTEXT.md` (Failures and Commits)
**Apply to:** Every BUILD-05 fix. One commit per requirement ID. The commit message names the ID. Log exit status and the first and last 20 lines of output in `02-RUN-LOG.md`. Full logs go to `~/runtests/logs/`.

## No Analog Found

| File or call site | Role | Data Flow | Reason |
|---|---|---|---|
| `~/runtests/` test scripts and logs | test artifact | file-I/O | Outside the repo. No test scripts exist in the repo for GUI or bundle checks. |
| `.planning/phases/02-launch-and-run-verification/02-RUN-LOG.md` | log | file-I/O | Format from CONTEXT. Phase 1 kept summaries in `01-0N-SUMMARY.md`. |

## Metadata

**Analog search scope:** `Frameworks/`, `Applications/TextMate/src/`, `Shared/include/oak/`
**Files scanned:** 9 call-site files and 3 analog files
**Pattern extraction date:** 2026-10-19
