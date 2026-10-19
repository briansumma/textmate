---
schema_version: 1
open_count: 1
waived_count: 0
fixed_count: 0
total_count: 1
last_updated: 2026-10-19T23:17:50.735Z
---

# Broken Windows Ledger

> Cross-phase defect register. With `workflow.windows_enforce` enabled, `/gsd-ship` blocks while `open_count > 0`.
> Waive with `gsd-tools windows waive <id> "<reason>"` (reason required).
> Mark fixed with `gsd-tools windows fixed <id>`.

| id | phase | kind | file | line | description | status | reason | recorded_at | resolved_at |
|----|-------|------|------|------|-------------|--------|--------|-------------|-------------|
| 1 | 2 | unrun-verify | Frameworks/HTMLOutput/src/OakHTMLOutputView.mm | 199 | BUILD-05 site a (NSURLConnection canHandleRequest in HTML output policy delegate) not exercised by RUN-04: rendered output has no link seen in captured area; page not fully scrolled. Open item for Brian. | open |  | 2026-10-19T23:17:50.735Z |  |

````json
[
  {
    "id": 1,
    "kind": "unrun-verify",
    "phase": "2",
    "file": "Frameworks/HTMLOutput/src/OakHTMLOutputView.mm",
    "line": 199,
    "description": "BUILD-05 site a (NSURLConnection canHandleRequest in HTML output policy delegate) not exercised by RUN-04: rendered output has no link seen in captured area; page not fully scrolled. Open item for Brian.",
    "status": "open",
    "reason": "",
    "recorded_at": "2026-10-19T23:17:50.735Z",
    "resolved_at": null,
    "milestone": null
  }
]
````
