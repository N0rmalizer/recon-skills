# grep -Eo Kills masscan Output Parsing on Alpine

**Date:** 2026-07-25
**Test:** masscan output parsing on charlie (Alpine/BusyBox)
**Result:** FAILED

## What happened
```bash
masscan scanme.nmap.org -p1-1000 --rate=5000 | grep -Eo "port \d+"
# grep: unrecognized option: P
```

## Why
Alpine Linux uses BusyBox grep which does NOT support `-P` (Perl regex).
This is the same issue fixed across 42 skills in the `grep -Eo` → `grep -Eo` batch fix.

## Fix
```bash
masscan scanme.nmap.org -p1-1000 --rate=5000 | grep -Eo "port [0-9]+"
```

This is already addressed in the batch fix: 42 skills had `grep -Eo` replaced with `grep -Eo`.
