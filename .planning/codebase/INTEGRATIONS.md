---
last_mapped_commit: 346b52b108b387462d4b3def481fb74983ae89f3
last_mapped_at: 2026-10-19
---
# External Integrations

**Analysis Date:** 2026-10-19

## APIs & External Services

**TextMate backend (`REST_API`):**
- Base URL: `https://api.textmate.org` - compiled into the app as `-DREST_API="https://api.textmate.org"` (`default.rave`). Source of the `REST_API` macro: `default.rave`, override via `FLAGS` in `local.rave`.
- Endpoints referenced in source:
  - `${REST_API}/releases/release`, `/releases/beta`, `/releases/nightly` - update channels, with query params `v=<version>&os=<major.minor.patch>` (`Applications/TextMate/src/AppController.mm` lines ~494-496).
  - `${REST_API}/bundles` - remote bundle index (`Frameworks/BundlesManager/src/BundlesManager.mm` line ~68; `Frameworks/updater/src/updater.cc` line ~72 registers source `org.textmate.updates.default`).
  - `${REST_API}/crashes` - crash report upload (`Applications/TextMate/src/AppController.mm` line ~614; `Frameworks/CrashReporter/src/CrashReporter.mm`, multipart/form-data POST, 60 s timeout).
  - Reachability probe: `can_reach_host` on the `REST_API` host before grammar lookup (`Frameworks/DocumentWindow/src/DocumentWindowController.mm` line ~1016).
- Client: `NSURLSession` (`Frameworks/SoftwareUpdate/src/SoftwareUpdate.mm`, `Frameworks/CrashReporter/src/CrashReporter.mm`, `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm`) and libcurl (`Frameworks/network/src/download_tbz.cc`, `Frameworks/updater/src/download.cc`).
- Release upload (release tooling only): `https://api.textmate.org/releases` via `curl -F` in `local-orig.rave` (`define deploy`).

**Update signing and verification:**
- Downloads are verified with a public-key signature delivered in HTTP metadata headers `x-amz-meta-x-signature` and `x-amz-meta-x-signee` (`Frameworks/network/src/constants.cc`, `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm` line 5). The `x-amz-meta-` prefix implies an S3-compatible object store behind `api.textmate.org`; the storage host is not declared in this repo.
- Verification uses Security.framework `SecVerifyTransformCreate` against public keys passed in by the app (`Frameworks/SoftwareUpdate/src/OakDownloadManager.mm`, `Frameworks/network/src/filter_check_signature.cc`).
- Release signer (`local-orig.rave`): `openssl dgst -dss1` (DSA with SHA-1), signature base64-encoded and uploaded as the `signature` form field.
- Public keys are embedded in the app, not stored in this repo as secrets.

**Software Update client:**
- `Frameworks/SoftwareUpdate/src/SoftwareUpdate.mm` - checks version (`plist[@"url"]` from channel feed), supports "suspend until" (`Skip version check: Suspended until`).
- `Frameworks/SoftwareUpdate/src/OakCompareVersionStrings.mm` - version comparison.
- Entity tags (`ETag`, `If-None-Match`) used for conditional downloads (`Frameworks/network/src/download_tbz.cc` line ~146; `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm` line ~331).

**Licensing (`Frameworks/license/`):**
- Local RSA verification: the license key is base32, decoded and decrypted with a public key via `SecKeyCreateDecryptedData(kSecKeyAlgorithmRSAEncryptionRaw)` (`Frameworks/license/src/license.cc`).
- Revocation check via DNS: `gethostbyname("<serial>.bl.textmate.org")`; an answer of `0xDEADBEEF` marks the serial revoked (`Frameworks/license/src/license.cc` lines ~171-185). This is a network call on every license validation for the serial.
- Storage: license is persisted to the macOS keychain (`Frameworks/license/src/keychain.cc`, `Frameworks/license/src/keychain.h`) via `SecKeychainItemCopyAttributesAndData`.
- Purchase link: `https://shop.macromates.com` (`Frameworks/license/src/LicenseManager.mm` line ~145).
- Revocation blocklist and purchase endpoints are owned by MacroMates; no third-party license service is used.

**Documentation / external links opened in browser:**
- `https://macromates.com/textmate/manual/`, `https://macromates.com/support`, `https://macromates.com/download` (`Applications/TextMate/about/About.md`, `README.md`).
- `https://github.com/textmate/textmate/wiki/FAQ`, `.../wiki/Hidden-Settings` (`Applications/TextMate/about/About.md`).
- `https://github.com/textmate/rmate/` (`Frameworks/Preferences/src/TerminalPreferences.mm` line ~240).
- Issue tracker template: `https://github.com/textmate/textmate/issues/%s` (`.tm_properties`, `TM_ISSUE_URL`).
- Mailing list: `https://lists.macromates.com/listinfo/textmate` (`README.md`, `CONTRIBUTING.md`).

## Source Control Integrations

**Local VCS tools (invoked as subprocesses, not via a network API):**
- Git - `Frameworks/scm/src/drivers/git.cc` (`driver_t("git", "%s/.git", "git")`). Detects `.git` in the path.
- Mercurial - `Frameworks/scm/src/drivers/hg.cc` (`.hg`, executable `hg`).
- Subversion - `Frameworks/scm/src/drivers/svn.cc` (`.svn`, executable `svn`).
- Executable lookup: `Frameworks/scm/src/drivers/api.cc` line 42 special-cases `/usr/bin/git` and `/usr/bin/svn`. Test overrides via `TM_GIT`, `TM_HG`, `TM_SVN`.
- File-system events drive status refresh: `Frameworks/scm/src/fs_events.cc` (FSEvents).
- Commit window: `Frameworks/CommitWindow/` - connects to a separate `commit` tool process via `NSConnection` (`Frameworks/CommitWindow/tool/commit.mm`).

## Bundles & Language Sources

- Bundle install/update via tar archives: `Frameworks/network/src/tbz.cc` extracts `.tbz` archives with `/usr/bin/tar -jxmkC ... --disable-copyfile --exclude "._*"`.
- `Frameworks/SoftwareUpdate/src/OakDownloadManager.mm` also extracts with `/usr/bin/tar` after signature check.
- Remote bundle index: `${REST_API}/bundles` (see above).
- Bundled bundles: `Applications/TextMate/support/` (copied as `SharedSupport`, `Applications/TextMate/default.rave`), `Applications/SyntaxMate/resources/SyntaxMate.tmBundle` (submodule).
- Bundle editor templates: `Frameworks/BundleEditor/templates/*.plist` (no network).

## Data Storage

**Databases (local only):**
- SQLite 3 via `sqlite3_open` - clipboard history in `Frameworks/OakAppKit/src/OakPasteboard.mm` (`databaseURL`; `:memory:` supported for tests).
- kvdb (vendored, SQLite-backed key-value store) - `vendor/kvdb/vendor/kvdb/*.m`; used by `Applications/TextMate/src/Favorites.mm` (`sharedProjectStateDB`) and `Frameworks/DocumentWindow/src/DocumentWindowController.mm` (`sharedProjectStateDB`, session restore).
- Cap'n Proto binary cache for plist files: `Frameworks/plist/src/fs_cache.cc` (`capnp/serialize-packed.h`, `cache.capnp`).
- Encoding frequency tables: `Frameworks/encoding/src/encoding.mm` (`frequencies.capnp`).

**File Storage:**
- Local filesystem only. Settings: `~/Library/Application Support/TextMate/` (`Global.tmProperties`, project state).
- Temp/socket: `/tmp/textmate-<uid>.sock` (see IPC below).
- Extended attributes: ETag stored via `setxattr` on downloaded files (`Frameworks/SoftwareUpdate/src/OakDownloadManager.mm` line ~363).

**Caching:**
- No external cache service. Local `${builddir}/.cache` for ThinLTO (`default.rave` release config).

## Authentication & Identity

**Auth provider:**
- None for end users. TextMate has no account system.
- License key (see Licensing above) is the only identity; stored in the macOS keychain.

**Privilege escalation:**
- `Applications/PrivilegedTool/` - privileged helper installed via `Applications/PrivilegedTool/src/install.cc` (launchd plist written with `OnDemand`, `MasterSocket` launch socket in `Applications/PrivilegedTool/src/launchd.cc`).
- Authorization: `AuthorizationCreate` in `Frameworks/authorization/src/authorization.h` and `Frameworks/Preferences/src/TerminalPreferences.mm`; `AuthorizationExecuteWithPrivileges` (deprecated API) in `Shared/include/oak/compat.h`.
- IPC between the app and the privileged tool: AF_UNIX stream socket (`Frameworks/authorization/src/connection.cc`, `Frameworks/authorization/src/server.cc`).

**Keychain access:**
- Proxy credentials: `SecKeychainItemCopyAttributesAndData` in `Frameworks/network/src/proxy.cc`, `Frameworks/network/src/key_chain.cc`.
- License storage: `Frameworks/license/src/keychain.cc`.

## Network Configuration

**Proxies:**
- Read from System Configuration: `SCDynamicStoreCopyProxies` in `Frameworks/network/src/proxy.cc`. Applied to libcurl via `CURLOPT_PROXY`, `CURLOPT_PROXYTYPE` (HTTP or SOCKS4) and `CURLOPT_PROXYUSERPWD` (`Frameworks/network/src/download_tbz.cc` lines ~150-157).

**User agent / telemetry-like data:**
- Every libcurl download sends `create_agent_info_string()` as `User-Agent` (`Frameworks/network/src/user_agent.cc`, `Frameworks/network/src/download_tbz.cc` line ~131). The string contains: app name and version, the **hardware UUID** from `gethostuuid()`, OS version, `HW_MACHINE`, `HW_MODEL`, and `HW_NCPU`.
- Update checks append `v=<version>&os=<x.y.z>` to the channel URL (`Applications/TextMate/src/AppController.mm` line ~490).
- Pending crash reports are posted to `${REST_API}/crashes` automatically at launch (`Applications/TextMate/src/AppController.mm` line ~614, `postNewCrashReportsToURLString`). No user-consent gate is visible at that call site.

## Monitoring & Observability

**Error tracking:**
- None third-party. Crash reports go to MacroMates (`${REST_API}/crashes`). Crash handler: `Frameworks/crash/`, `Frameworks/CrashReporter/src/CrashReporter.mm` (notifications, `NSWorkspace openURL` for report location).
- `symbolicate` script for dSYM archives: `Frameworks/CrashReporter/bin/symbolicate` (reads `DSYM_ARCHIVE_DIR` `*.tbz`).

**Logs:**
- Unified logging via `os_log` / `os_log_error` (e.g. `Frameworks/SoftwareUpdate/src/SoftwareUpdate.mm`, `Frameworks/network/src/proxy.cc` subsystem `TextMate/proxy`).
- `bin/show_log` - script to display TextMate logs (`#!/usr/bin/env ruby -wKU`).
- No Sentry, Datadog, or similar SDK.

## CI/CD & Deployment

**CI:**
- GitHub Actions: `.github/workflows/build.yml` - `runs-on: macOS-latest`, on every push. Steps: checkout with `submodules: recursive`, `brew install boost capnp google-sparsehash multimarkdown ninja ragel`, `./configure`, `ninja TextMate`.
- Travis CI (legacy): `.travis.yml` - `language: objective-c`, `osx_image: xcode7.2`, installs `libressl` as well. Not reflected by `.github` workflow; likely stale.
- Note: repository is hosted on GitHub (`origin https://github.com/textmate/textmate.git`). Bitbucket is not used for this repo.

**Hosting / distribution:**
- Release artifacts: `.tbz` archive built by `bin/rave` (`define tbz`), tagged `v<version>` via `git tag`, uploaded to `https://api.textmate.org/releases` (`local-orig.rave`).
- GitHub release creation via `hub release create --prerelease` (`local-orig.rave` `define upload`).
- Code signing: `codesign --force --options runtime` with Developer ID (`local-orig.rave` `CS_IDENTITY`).
- Notarization: `xcrun altool --notarize-app` followed by polling in `bin/notarize_await` (`bin/rave` lines ~1238-1246). Credentials come from `APPLE_NOTARIZE_USER` and `APPLE_NOTARIZE_PASS` (names only).

## Environment Configuration

**Required env vars (build/release only, names only):**
- `APPLE_DEVELOPER_ID` - codesign identity (`local-orig.rave`).
- `APPLE_NOTARIZE_USER`, `APPLE_NOTARIZE_PASS` - notarization credentials (`local-orig.rave`).
- `TEXTMATE_CHANGES`, `TEXTMATE_VERSION` - release notes path and version (`local-orig.rave`; version derived from `Applications/TextMate/about/Changes.md`).
- `builddir` - optional build output directory (`configure`, `bin/rave`).
- `TM_GIT`, `TM_HG`, `TM_SVN` - optional SCM executable overrides (tests).

**Runtime env:**
- None required by the app. Shell `PATH` is used by `bin/rave` (`ENV['PATH']`) to find `pbzip2` and other tools.

**Secrets location (names and paths only; contents not read):**
- Release signing key file path referenced in `local-orig.rave` (`${HOME}/.local/share/private/sign-textmate.key`). The file is not in this repo.
- Release signing key also retrieved through `pass Keys/sign-textmate.key` (`local-orig.rave`). Entry lives in the user's password store, not in the repo.
- `.gitignore` excludes `local.rave` (per-machine build config, may hold credentials via `${APPLE_NOTARIZE_PASS}`).
- No `.env` file present in the repo root.

## Inter-Process and Local Integrations

**`txmt://` URL scheme:**
- Registered in `Applications/TextMate/Info.plist` (`CFBundleURLSchemes` = `txmt`). Handled by `Applications/TextMate/src/AppController.mm`.

**`mate` command-line client:**
- `Applications/mate/src/mate.mm` - connects to a Unix domain socket at `/tmp/textmate-<uid>.sock` (`socket_path()` line ~17). Server side not located in this analysis.
- `Applications/tm_query/` - query helper, installed into `MacOS/` (`Applications/TextMate/default.rave`).

**rmate (remote editing) server:**
- `Applications/TextMate/src/RMateServer.mm` - TCP listener (IPv6 socket) on port `52698` by default (`Frameworks/Preferences/src/Keys.mm`). Binds to `localhost` unless `rmateServerListen = remote`. Enable/disable via `rmateServerDisabled`.
- Protocol is the rmate/RMate line protocol; client project: `https://github.com/textmate/rmate/`.

**AppleEvents:**
- `NSAppleEventsUsageDescription` in `Applications/TextMate/Info.plist` - used to reply to "Edit in TextMate" requests from other apps.
- Entitlement `com.apple.security.automation.apple-events` in `Applications/TextMate/Entitlements.plist`.

**Distributed Objects / NSConnection:**
- `Frameworks/CommitWindow/src/CommitWindow.mm` and `Frameworks/CommitWindow/tool/commit.mm` - exchange messages with a separate commit-window process using `NSConnection rootProxyForConnectionWithRegisteredName`.

**QuickLook:**
- `Applications/QuickLookGenerator/` - QuickLook generator bundle; `TextMateQL` copied into `Library/QuickLook` (`Applications/TextMate/default.rave`).

**Authorization/privileged helper:**
- See Authentication & Identity above.

## Webhooks & Callbacks

**Incoming:**
- None over the network except the rmate TCP listener (`Applications/TextMate/src/RMateServer.mm`, port 52698) and the local `txmt://` URL handler.
- Local Unix socket `/tmp/textmate-<uid>.sock` is the endpoint `mate` connects to (`Applications/mate/src/mate.mm`). The server-side listener was not located during this analysis.

**Outgoing:**
- Update checks to `${REST_API}/releases/*`.
- Bundle index fetch from `${REST_API}/bundles`.
- Crash report POST to `${REST_API}/crashes`.
- License revocation DNS lookups to `*.bl.textmate.org`.
- Git, Mercurial, and Subversion commands run as local subprocesses (no outgoing HTTP from those).

---

*Integration audit: 2026-10-19*
