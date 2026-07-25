# Recon & Pentest Skill Pack

<p align="center">
  <img src="banner.png" alt="Recon & Pentest Skill Pack" width="800">
</p>

Field-validated offensive security skills for authorized reconnaissance, vulnerability hunting, and exploit chaining. Terminal-native. Research-backed. MIT licensed.

> **Disclaimer**: These skills are for authorized security testing only. Only test targets you own or have explicit written permission to test. Unauthorized scanning may violate computer fraud laws. The author assumes no liability for misuse.

> **Blog & research**: [hiago.sh](https://hiago.sh) — Pentest Playbook, field notes, and tooling.

---

## License

MIT — see [LICENSE](./LICENSE)

---

## What's Inside

```
recon-skills/
├── SOUL.md              — Philosophy & agent operating instructions
├── AGENTS.md            — Complete catalog + skill standards
├── STYLE.md             — Skill quality baseline (pitfalls, verification, rate limits)
├── LICENSE              — MIT license
├── recon/               — WordPress/CORS/XMLRPC recon, source leaks, JS secrets, web enum,
│                          email sec, staging hunt, port scans, hardcoded creds, S3/MinIO XSS,
│                          API exploitation, MCP security, LLM attacks, browser evasion, origin IP
│                          discovery, subdomain takeover, vhost enum, GitHub secrets, ASN mapping,
│                          visual recon, CMS detection
├── redteam/             — hunt-* skills (XSS, SQLi, SSRF, RCE, ATO, IDOR, CORS, Firebase,
│                          Supabase, MCP security, LLM attacks, schema-enum, write-gap, metrics, K8s, mass-assignment,
│                          prototype-pollution, BFLA, info-disclosure, Django, FastAPI, NestJS),
│                          plus recon-sector (parametrized, sectors.yaml database),
│                          plus methodology/ops tools
├── meta/                — Recon playbook, sector methodology, attack patterns, wave delta,
│                          Google dorks, pentest playbook
├── chains/              — Cross-attack chaining, WordPress full compromise
├── auth/                — SAML SSO attacks
├── infra/               — Docker privilege escalation
├── controller           — Agent orchestration
└── worker               — Multi-worker cluster (recon, heavy/RE, Tor)
```

## Key Skills

| Category | Skill | What It Does |
|----------|-------|-------------|
| **meta** | `recon-playbook` | 4-phase pipeline: target gen -> quick filter -> WP deep check -> deep invade |
| **recon** | `cors-credential-wordpress` | 8 CORS variants (V1-V8) |
| **recon** | `xmlrpc-exploitation` | System.multicall, pingback SSRF, IMDS role guessing, wp.uploadFile |
| **recon** | `web-enumeration` | 200+ sensitive file paths, .env extraction, path traversal, vhost enum |
| **recon** | `js-secrets-extraction` | 12 regex patterns for API keys, JWTs, Firebase, Supabase in JS bundles |
| **recon** | `email-security` | DMARC/SPF/DKIM checks, SMTP spoofing, header analysis |
| **chains** | `cross-attack-chains` | Attack chain methodology: CORS+XMLRPC->RCE, SSRF->IMDS, etc |
| **chains** | `wordpress-full-compromise` | Kill chains for full WordPress takeover |
| **meta** | `attack-patterns-reference` | 25 patterns (P-01 to P-25), 18 WP abuse patterns, 8 CORS variants |
| **meta** | `cross-wave-delta-analysis` | Compare waves: NEW / REGRESSION / PERSISTENT / CHANGE |
| **meta** | `sector-recon-methodology` | Tier-based sector selection + per-sector vulnerability baselines |
| **meta** | `google-dorks-catalog` | 100+ dork patterns by service type + GitHub code search |
| **redteam** | `recon-sector` | Parameterized sector recon: sectors.yaml database |


## Research

Incorporates external security research:

- **AI Agent Framework Audit** (2026): 56+ vulns across 13 frameworks, 7 CVEs (CVE-2026-2287)
- **HuntBook Methodology** (su6osec): XSS, SQLi, SSRF modern techniques
- **PortSwigger Research** (2025): SAML bypass, WebSocket attacks, HTTP anomaly detection
- **Tool Benchmarks**: dnsx 2x, httpx 5.6x, naabu 8x, ffuf vs curl rate-limit reality

See  and  for details.

## Quick Start

Clone into your agent's skills directory and reference via skill name:

```bash
git clone https://github.com/uphiago/recon-skills.git
```

Skills are self-contained markdown files with YAML frontmatter. Each skill documents trigger conditions, commands, verification steps, and pitfalls. See [STYLE.md](./STYLE.md) for the quality baseline and writing guidelines.

## Skill Standards

Every skill follows the [STYLE.md](./STYLE.md) baseline:

- **Conditions**: Trigger rules for when to use the skill
- **Commands**: Terminal-native commands with `--max-time` and rate limiting
- **Verification**: How to confirm findings
- **Pitfalls**: Known gotchas and edge cases
