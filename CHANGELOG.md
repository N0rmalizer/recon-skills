# Changelog

## 2026-07-25 — v1 Release

### Initial release with 147 skills
- **recon/** (41): Subdomain enumeration, CORS credential reflection, XMLRPC exploitation, WordPress plugin hunting, JS secrets extraction, API no-auth hunting, email security, source leak hunting, port scanning, ASN infrastructure mapping, web enumeration, GitHub secret hunting, CMS detection, browser evasion, origin IP discovery, subdomain takeover, vhost enumeration, visual recon, and more.
- **redteam/** (94): Per-class vulnerability hunting (XSS, SQLi, SSRF, RCE, IDOR, ATO, CSRF, LFI, SSTI, XXE, JWT, SAML, OAuth, GraphQL, WebSocket, cache poison, deserialization, race condition, host header, HTTP smuggling, mass assignment, prototype pollution, broken function-level auth, information disclosure, CORS, Firebase, Supabase, MCP security, LLM attacks, cloud IAM, Docker, K8s, and more). Plus methodology, reporting, and ops skills.
- **meta/** (6): Recon playbook, sector methodology, attack patterns reference, cross-wave delta analysis, Google dorks catalog, pentest playbook.
- **chains/** (2): Cross-attack chaining, WordPress full compromise.
- **auth/** (1): SAML SSO attacks.
- **infra/** (1): Docker privilege escalation.

### Quality baseline
- All skills follow STYLE.md: pitfalls, verification sections, --max-time on curl, grep -E for Alpine compat.
- YAML frontmatter standardized: version, license, category on all skills.
- No author attribution, no unverifiable target counts, no real organization data or IPs.

### Research integration
- AI Agent Framework Audit (correctover, 2026): 56+ vulns, CVE-2026-2287 (CrewAI RCE CVSS 9.8)
- HuntBook Methodology (su6osec, 2026): XSS, SQLi, SSRF techniques
- PortSwigger Research (2025): SAML bypass, WebSocket, HTTP anomaly detection
- Tool benchmarks: dnsx (2x), httpx (5.6x), naabu (8x), ffuf vs curl rate-limit testing

### License
MIT — see LICENSE file.
