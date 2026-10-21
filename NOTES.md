# Research Notes: CVE-2023-32681 — Proxy-Authorization header leak in Python `requests`

All facts below were gathered from live GitHub data on the `psf/requests` repository on 2026-10-02 (issues opened individually, PR records and files pulled directly, releases checked via the release history).

## CVE
**CVE-2023-32681** — GitHub Security Advisory **GHSA-j8r2-6x86-q33q**, "Unintended leak of Proxy-Authorization header in requests", severity **medium** (CVSS 3.1 score 6.1, vector CVSS:3.1/AV:N/AC:H/PR:N/UI:R/S:C/C:H/I:N/A:N), CWE-200 (Exposure of Sensitive Information to an Unauthorized Actor). Advisory: https://github.com/psf/requests/security/advisories/GHSA-j8r2-6x86-q33q

## Summary (plain language)
Since Requests v2.3.0, if you configured a proxy with credentials embedded in the proxy URL (e.g. `https://username:password@proxy:8080`), Requests built a `Proxy-Authorization` header to authenticate with the proxy. When a request got redirected, the `rebuild_proxies` code recomputed and re-attached that header to the new request. For plain-HTTP requests through a proxy, the proxy strips the header before forwarding, so nothing leaked; but when the redirect target was HTTPS, the proxy has no visibility into the tunneled traffic, so Requests ended up sending the `Proxy-Authorization` value — your proxy credentials — to the destination server. A malicious redirect target could therefore capture those credentials. Only proxy credentials passed in the URL's user-information component were affected; requests without proxies (or without URL-embedded proxy credentials) were not exposed. Fixed in v2.31.0 by no longer attaching the header to redirects with an HTTPS destination (urllib3's ProxyManager already handles proxy auth in the default adapter), with `allow_redirects=False` as a temporary workaround.

## Issues returned by the live searches (psf/requests)
Search 1 — issue search on `CVE-2023-32681`: **0 results**. (Also 0 for `32681` and for `GHSA-j8r2-6x86-q33q` — the CVE was handled via the security advisory process rather than a public issue.)
Search 2 — issue search on the leaked header name `Proxy-Authorization`: 47 total matches; searching titles (`in:title Proxy-Authorization`) narrows this to 6:
- **#5888 — "Manually set Proxy-Authorization header is always removed in 2.26"** — state: **closed** (closed 2021-11-24, state_reason: completed, locked, label: Bug, milestone 2.27.0)
- **#7615 — "Proposed fix for #5888: preserve manually-set Proxy-Authorization on initial request"** — state: **closed** (closed 2026-08-31, state_reason: not_planned, locked as off-topic, labels: Feature Request, actions/autoclose-feat)
- **#2426 — "Bug: field "uri" in header["Proxy-Authorization"] should not be empty"** — state: **closed** (closed 2015-07-17, state_reason: completed, locked)
- **#2677 — "can't use proxy-authorization in header"** — state: **closed** (closed 2015-07-17, state_reason: completed, locked)
- **#3252 — "Setting Proxy-Authorization Header Manually"** — state: **closed** (closed 2016-06-02, state_reason: completed, locked)
- **#5900 — "Basic authentication with a proxy not using Proxy-Authorization header"** — state: **closed** (closed 2021-08-17, state_reason: completed, locked)
(Also surfaced by the broad header search but off-topic for this CVE: #6025 "Support for proxy that requires DigestAuth" — open; #5975 "Request doesn't provide a way to refresh headers for each retry, making it impossible to regenerate expired Authorization headers" — open.)

## Fix pull request
- **psf/requests#6457 — "v2.31.0"** — state: **closed / MERGED** (merged at 2023-05-22T15:10:33Z by nateprewitt; head `nateprewitt:v2.31.0`; 1 commit, +29/−2, 2 files changed).
  - Files touched: **`HISTORY.md`** (modified, +27) and **`requests/__version__.py`** (modified, +2/−2, version bump 2.30.0 → 2.31.0).
  - Context: the actual code fix landed via the private security-advisory merge commit `74ea7cf7a6a27a4eeb2ae24e162bcc942a6706d5` ("Merge pull request from GHSA-j8r2-6x86-q33q", 2023-05-22), which modified **`requests/sessions.py`** (+3/−1) and **`tests/test_requests.py`** (+20); release PR #6457 is the public PR that shipped it. Follow-up formatting PR #6458 ("I knew we missed a step...", merged 2023-05-22) also touched `requests/sessions.py` and `tests/test_requests.py`.

## Release that first shipped the fix
- **Requests v2.31.0** — release `v2.31.0`, published 2023-05-22T15:26:23Z (tag `v2.31.0`). The release notes carry the full security notice for CVE-2023-32681 and point to GHSA-j8r2-6x86-q33q; the advisory lists vulnerable range `>= 2.3.0, < 2.31.0` with first patched version `2.31.0`. https://github.com/psf/requests/releases/tag/v2.31.0

## Credits (from the advisory)
Dennis Brinkrolf (Haxolot) and Tobias Funke discovered and disclosed; sethmlarson reviewed; nateprewitt developed the fix.
