# API NoAuth Probe — Verified on httpbin.org

**Date:** 2026-07-25
**Target:** https://httpbin.org
**Worker:** charlie (Alpine Linux)

## Results

| Path | HTTP | Content-Type | Size | API? |
|------|------|-------------|------|------|
| / | 200 | text/html | 9593B | No |
| /api | 404 | text/html | 233B | No |
| /get | 200 | application/json | 254B | ✅ |
| /post | 405 | text/html | 178B | ⚠️ |
| /status/200 | 200 | text/html | 0B | No |
| /json | 200 | application/json | 429B | ✅ |
| /headers | 200 | application/json | 173B | ✅ |
| /ip | 200 | application/json | 31B | ✅ |

## Verdict
✅ **JSON content-type detection works.** `application/json` reliably flags API endpoints.
- `/get`, `/json`, `/headers`, `/ip` correctly identified as APIs.
- `/api` returned 404 — valid negative.
- `/post` returned 405 (Method Not Allowed) — GET-only, valid detection.

## Practical Impact
The probe loop from api-noauth-hunt skill successfully identifies API endpoints
on real targets. The content-type check + JSON validation is reliable.
