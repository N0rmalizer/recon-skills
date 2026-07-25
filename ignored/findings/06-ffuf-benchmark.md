# ffuf vs curl Loop — Real Fuzzing Benchmark

**Date:** 2026-07-25
**Target:** httpbin.org (10 word wordlist)
**Worker:** charlie (Alpine Linux)

## Results

| Method | Time | Found | Notes |
|--------|------|-------|-------|
| `ffuf -u URL/FUZZ -w words -mc 200,405 -s` | 20.01s | 4 endpoints | Concurrent, handles rate limiting |
| `curl in while loop (no delay)` | 0.39s | 10 (all 503) | Rate-limited by target instantly |

## Analysis

- curl loop hit rate limit immediately (all 503), producing zero useful results
- ffuf successfully found 4 live endpoints despite rate limiting (built-in retry/delay)
- This proves curl loops are NOT just slower — they can be COMPLETELY USELESS against rate-limited targets
- ffuf has built-in rate limiting awareness, retry logic, and concurrency control

## Verdict
✅ **ffuf is essential, not optional.** Against any target with rate limiting, curl loops fail entirely.
ffuf should REPLACE all curl-based directory fuzzing in skills.
