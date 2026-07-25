# httpx vs curl Loop — HTTP Probing Speed Test

**Date:** 2026-07-25
**Target:** https://httpbin.org (10 endpoints)
**Worker:** charlie (Alpine Linux)

## Results

| Method | Time | Results | Notes |
|--------|------|---------|-------|
| httpx -status-code -title | 2.74s | 10 URLs + codes + titles | Concurrent, structured |
| curl loop (1s delay) | 15.36s | 10 codes only | Sequential, raw output |

## Analysis

- **httpx is ~5.6x faster** (2.74s vs 15.36s with 1s delay)
- httpx automatically adds status codes, titles, content-length, and tech stack
- httpx handles redirects, timeouts, and retries natively
- curl loop requires manual --max-time, --connect-timeout, redirect handling

## Verdict
✅ **httpx replaces bare curl probing for batch HTTP checks.**
Use httpx for batch alive checks, tech detection, and status probing.
Keep curl for one-off requests, API calls with custom headers, and PoC reproduction.

## Command

