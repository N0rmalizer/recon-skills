# Rate Limiting Reality — Why curl Loops Fail

**Date:** 2026-07-25
**Finding:** httpbin.org rate-limits rapid curl requests to 503

## The Problem

When fuzzing endpoints rapidly with a curl loop:
```bash
while read word; do curl -sk "http://target/$word"; done
```
Targets respond with 503/429 after 2-3 requests. The entire loop produces garbage.

## The Solution
**ffuf** handles this natively:
- Automatic rate limit detection
- Built-in retry with backoff  
- Concurrent requests with `-t` flag
- Match/filter by status code, size, words, regex

## Impact on Skills
Any skill using `curl` in a loop for fuzzing MUST be upgraded to `ffuf`.
This is not an optimization — curl loops are broken against modern infrastructure.
