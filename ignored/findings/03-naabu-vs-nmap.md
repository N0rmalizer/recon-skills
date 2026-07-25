# naabu vs nmap — Port Scan Speed Test

**Date:** 2026-07-25
**Target:** scanme.nmap.org (ports 1-1000)
**Worker:** charlie (Alpine Linux)

## Results

| Method | Time | Ports Found | Notes |
|--------|------|-------------|-------|
| naabu -p 1-1000 | ~1.0s | 22, 80 | Fast, Go-based, concurrent |
| nmap -p 1-1000 | ~7.7s | 22, 80 | Slower, but includes service detection |

## Analysis

- **naabu is ~8x faster** for the same port range (1-1000)
- Both found identical results (22/tcp ssh, 80/tcp http)
- naabu uses SYN scan with adjustable rate; nmap does full TCP connect by default
- naabu output is cleaner (just IP:port pairs), nmap adds state and service columns

## Verdict
✅ **naabu replaces manual port loops and is faster than nmap for initial discovery.**
Use naabu for initial port discovery, nmap for service/version detection.

## Command
target.com:443
target.com:80
