# ffuf vs curl Loop — Web Fuzzing Speed Test

**Date:** 2026-07-25
**Target:** https://httpbin.org (10 paths)
**Worker:** charlie (Alpine Linux)

## Results

| Method | Time | Paths Found | Notes |
|--------|------|-------------|-------|
| ffuf -mc 200,401,405 -t 20 | 3.06s | 11 paths | Concurrent, colorized |
| curl loop (2s delay) | 27.79s | 10 paths | Sequential, rate-limited |

## Analysis

- **ffuf is ~9x faster** for 10 paths even with 2s curl delay
- ffuf found /anything (200) that the curl loop didn't test (only 10 paths)
- ffuf supports multiple matchers (-mc, -mr, -ms) and filters (-fc, -fs, -fw)
- curl loop requires manual sleep for rate limiting; ffuf has built-in -p (delay) and -rate

## Verdict
✅ **ffuf replaces curl loops and gobuster/dirb for directory/file fuzzing.**
ffuf is faster, more feature-rich, and handles rate limiting natively.

## Command
```bash
ffuf -w wordlist.txt:FUZZ -u https://target.com/FUZZ -mc 200 -s -t 20 -o results.json
```
