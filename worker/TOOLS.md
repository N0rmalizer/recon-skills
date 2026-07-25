# Worker Tool Inventory

Verified tool availability across the 3-worker Barbarossa cluster.

## Charlie (Collect — Recon)
```
Host: charlie (172.18.0.4)
OS: Alpine Linux
```

| Tool | Version | Status | Notes |
|------|---------|--------|-------|
| nmap | 7.95 | ✅ | |
| masscan | | ✅ | ⚠️ Needs `apk add libpcap` first run |
| naabu | | ✅ | |
| amass | | ✅ | |
| subfinder | | ✅ | |
| dnsx | | ✅ | |
| httpx | | ✅ | |
| nuclei | | ✅ | |
| ffuf | 2.1.0 | ✅ | |
| katana | | ✅ | |
| curl | 8.14.1 | ✅ | Alpine build (BusyBox) |
| python3 | 3.12.13 | ✅ | |
| gcc | 14.2.0 | ✅ | |
| jq | 1.7.1 | ✅ | |

**grep notes:** Alpine uses BusyBox grep — NO `-P` (Perl regex). Use `-E` (extended).
**curl notes:** Supports `--max-time`, `--connect-timeout`. Does NOT support `--parallel`.

## Oscar (Operate — RE/Binary)
```
Host: oscar
OS: Debian 12
```

| Tool | Version | Status | Notes |
|------|---------|--------|-------|
| nmap | 7.93 | ✅ | |
| nuclei | | ✅ | |
| python3 | 3.11.2 | ✅ | |
| gdb | 13.1 | ✅ | |
| gcc | 12.2.0 | ✅ | |
| strace | 6.1 | ✅ | |
| ltrace | 0.7.3 | ✅ | |
| xxd | | ✅ | |
| file | 5.44 | ✅ | |

**grep notes:** Debian — supports `-P` (Perl regex), `-E`, `-o`.

## Papa (Persist — Anonymous)
```
Host: papa (172.18.0.3)
OS: Alpine Linux
SSH: ssh -i /hermes/ssh/key root@papa
Tor: torsocks (SOCKS5 :9050)
```

| Tool | Version | Status | Notes |
|------|---------|--------|-------|
| nmap | 7.95 | ✅ | |
| python3 | 3.12.13 | ✅ | |
| tor | 0.4.9.11 | ✅ | |
| torsocks | 2.5.0 | ✅ | |
| curl | 8.14.1 | ✅ | |
| subfinder | v2.14.0 | ✅ | Copied from charlie |
| masscan | | ✅ | Copied from charlie, libpcap installed |

**Usage:** Prefix commands with `torsocks` for anonymous operation.
**grep notes:** Same as Charlie — BusyBox, no `-P`.

---
Last updated: 2026-07-25
