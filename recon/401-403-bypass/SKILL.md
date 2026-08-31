---
name: 401-403-bypass-techniques
description: Validate 401/403 access control on authorized targets.
version: 1.0.0
revision_date: 2026-08-31
license: MIT
platforms: [linux]
compatibility: Requires curl; byp4xx is optional.
metadata:
  tags: [recon, authorization, http, access-control]
  category: recon
  related_skills:
    - hunt-auth-bypass
    - hunt-http-smuggling
    - hunt-ssrf
    - hunt-idor
---

# 401/403 Access-Control Validation

Use this playbook only for an asset and path covered by written authorization. A
changed status code is a lead, not proof of an access-control bypass. The result
must show that protected content or an authorized action became available to an
unauthenticated or otherwise unauthorized principal.

## When to Use

- A permitted endpoint returns `401` or `403` and the owner has approved access-control testing.
- You need to compare proxy and application path normalization.
- You need repeatable, read-only evidence for a potential authorization issue.

Do not use this skill for credential attacks, default-credential testing, broad
internet scanning, or testing endpoints outside the written scope.

## Prerequisites

- Written authorization naming the host, paths, test window, and request limits.
- A test account or other approved principal for comparison with the unauthenticated baseline.
- `curl` and a dedicated output directory. Use HTTPS and preserve certificate validation.
- Optional: [`byp4xx`](https://github.com/lobuhi/byp4xx), installed from its official repository and reviewed locally before use.

## How to Run

Start with a single read-only baseline and a small, rate-limited candidate set:

```bash
TARGET_BASE="${TARGET_BASE:-https://target.example}"
TARGET_PATH="${TARGET_PATH:-/admin}"
OUTPUT_ROOT="${OUTPUT_DIR:-./output}"
OUTDIR="$OUTPUT_ROOT/401-403"

case "$TARGET_BASE" in
  https://*) ;;
  *) echo "TARGET_BASE must use https://" >&2; exit 2 ;;
esac

mkdir -p "$OUTDIR"
printf 'case\tstatus\tbytes\teffective_url\n' > "$OUTDIR/results.tsv"

# Read-only baseline. Do not follow redirects; inspect Location and the body.
curl -sS --path-as-is --max-time 10 --connect-timeout 5 \
  -D "$OUTDIR/baseline.headers" \
  -o "$OUTDIR/baseline.body" \
  -w "baseline\t%{http_code}\t%{size_download}\t%{url_effective}\n" \
  "${TARGET_BASE}${TARGET_PATH}" >> "$OUTDIR/results.tsv"

CANDIDATES=(
  "${TARGET_PATH%/}/"
  "/./${TARGET_PATH#/}"
  "//${TARGET_PATH#/}"
)

index=0
for candidate in "${CANDIDATES[@]}"; do
  index=$((index + 1))
  curl -sS --path-as-is --max-time 10 --connect-timeout 5 \
    -D "$OUTDIR/candidate_${index}.headers" \
    -o "$OUTDIR/candidate_${index}.body" \
    -w "candidate_${index}\t%{http_code}\t%{size_download}\t%{url_effective}\n" \
    "${TARGET_BASE}${candidate}" >> "$OUTDIR/results.tsv"
  sleep 1
done
```

Run the optional tool only after reconfirming the written authorization, target
path, test account, and rate limit immediately before execution. Disable any
default-credential behavior:

```bash
byp4xx -m 10 --rate 1 -xD "${TARGET_BASE}${TARGET_PATH}"
```

## Procedure

1. Confirm the exact host, path, HTTP method, test window, and request budget. Record the approved test account and the expected protected marker without placing credentials in a repository.
2. Capture a baseline with `GET`. Record status, redirect location, headers, body length, title, and a stable protected-content marker. A login page, generic error page, or CDN block is not protected content.
3. Test one read-only path variant at a time: trailing slash, dot segment, duplicate slash, or a case/encoding variant only when the deployed stack is known to normalize it differently.
4. Test proxy-specific headers only when the owner confirms that a proxy or rewrite layer uses them. Send one header at a time, compare against the baseline, and never assume a client-supplied identity header is trusted.
5. Compare the candidate response semantically. A `200` is meaningful only when the unauthorized request returns the protected resource or an approved harmless canary, with no compensating authorization check.
6. Before any request that can change state (`POST`, `PUT`, `PATCH`, or `DELETE`), stop and reconfirm written authorization, the test account, the exact endpoint, and a harmless payload. Keep such requests manual; this skill intentionally provides no state-changing command.
7. Preserve request/response pairs, redact cookies and tokens, and stop if the target returns real sensitive data or causes an unintended state change.

### Read-only candidate matrix

| Candidate | Use when | Qualification |
|---|---|---|
| Trailing slash or dot segment | The router and proxy normalize paths differently | Compare the final route and protected marker, not only the status |
| Duplicate slash | The deployed router documents slash normalization | Do not assume every framework collapses `//` |
| Case variant | The backend is case-insensitive and the proxy is case-sensitive | Avoid guessing on case-sensitive filesystems |
| Percent-encoded unreserved character | The stack has a documented decoding difference | Test one segment and record every decode boundary |
| Semicolon path parameter | A Java/Tomcat-style path-parameter parser is confirmed | Treat as stack-specific; do not use as a generic payload |
| Backslash or double encoding | The owner confirms an IIS, proxy, or legacy parser is in scope | Keep disabled by default and use only in a controlled test window |

Do not treat overlong UTF-8, null-byte suffixes, or obsolete parser tricks as
generic bypasses. Modern parsers commonly reject them, and malformed encodings
can create noisy or unsafe behavior.

### Proxy and rewrite headers

Potentially relevant headers include `X-Original-URL`, `X-Rewrite-URL`, and
forwarding headers. They are meaningful only when a specific proxy or rewrite
configuration consumes them. Test them in isolation, capture the complete
request, and verify the application-level authorization decision. Do not send
spoofed client-IP or identity headers as a default bypass recipe.

## Pitfalls

- `401` normally indicates that authentication is required or invalid; `403` indicates that the server understood the request but refuses to fulfill it. Neither status alone identifies a vulnerability.
- A `200` can be a login page, WAF challenge, cached error page, or public shell. A `3xx` can only be a redirect to login. Inspect the body, title, `Location`, cache headers, and protected marker.
- `HEAD` has no response body, and `OPTIONS` or `TRACE` exposure does not prove access to a protected resource.
- Path and header behavior differs across CDN, reverse proxy, web server, framework, and application layers. Record which layer changed the response.
- Keep requests slow and bounded. Do not use `-k`, unbounded retries, credential lists, or concurrency against a live target.
- Do not store cookies, authorization headers, reset tokens, or real sensitive responses in the repository. Redact evidence before sharing it.

## Verification

Treat a candidate as validated only when all of the following are true:

- The request was within written scope and used the approved principal.
- The candidate response differs from the baseline in a reproducible way.
- The response contains a stable, harmless marker from the protected resource, not just a new status code or page length.
- The application did not subsequently enforce authorization, redirect to login, or return a cached/public response.
- The test was read-only, or any state-changing action had separate explicit authorization and was executed manually with a harmless payload.
- Evidence contains the timestamp, method, normalized path, relevant headers, redacted body excerpt, and exact tool versions.
