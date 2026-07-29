# Recon & Pentest Skills

<p align="center">
  <img src="banner.png" alt="Recon and pentest skills" width="800">
</p>

A runtime-agnostic library of executable playbooks for authorized
reconnaissance, vulnerability validation, attack-path analysis, and security
reporting.

> These skills are for authorized security testing only. Only test targets you
> own or have explicit written permission to test.

> **Blog & research**: [hiago.sh](https://hiago.sh) - Pentest Playbook, field
> notes, and tooling.

## What This Repository Contains

Each skill is a self-contained `SKILL.md` with:

- clear activation conditions;
- explicit prerequisites;
- reproducible commands and procedures;
- false-positive and rate-limit guidance;
- verification criteria;
- references, scripts, or templates where they improve repeatability.

The library is organized by security domain:

```text
recon-skills/
|-- auth/       Authentication and SSO testing
|-- chains/     Multi-step attack-path analysis
|-- infra/      Infrastructure-focused techniques
|-- meta/       Engagement planning and cross-skill workflows
|-- recon/      Discovery, enumeration, and focused validation
`-- redteam/    Vulnerability-class and platform playbooks
```

The repository deliberately does not define an agent runtime, scheduler,
container topology, chat interface, model provider, or deployment platform. A
human operator, an agent, or an automation system can consume the same skills.

## Operating Model

Use the library as an evidence-driven pipeline:

```text
scope
  -> passive discovery
  -> active enumeration
  -> service and technology classification
  -> hypothesis-driven validation
  -> attack-path analysis
  -> evidence review
  -> reporting
```

Start with `meta/recon-playbook`, then load only the skills relevant to the
observed surface. Prefer focused validation over indiscriminate scanning.

Set a writable output location before running examples:

```bash
export OUTPUT_DIR="${OUTPUT_DIR:-./output}"
mkdir -p "$OUTPUT_DIR"
```

Commands assume standard Linux tooling unless a skill states otherwise. Tool
availability, network policy, concurrency, credentials, and isolation remain
the responsibility of the execution environment.

## High-Signal Entry Points

| Skill | Purpose |
|---|---|
| `meta/recon-playbook` | End-to-end recon workflow and escalation gates |
| `redteam/bb-methodology` | Bug bounty methodology and prioritization |
| `redteam/web2-recon` | Broad web attack-surface discovery |
| `redteam/offensive-osint` | External intelligence and asset pivots |
| `recon/subdomain-enumeration` | Passive and active subdomain discovery |
| `recon/port-service-discovery` | Port and service classification |
| `recon/web-enumeration` | Web paths, files, and technology enumeration |
| `recon/js-secrets-extraction` | Client-side bundle and secret analysis |
| `chains/cross-attack-chains` | Evidence-based attack-path construction |
| `redteam/triage-validation` | Finding validation before reporting |
| `redteam/evidence-hygiene` | Reproducible and redacted evidence capture |
| `redteam/report-writing` | Client and bug bounty reporting |

The `hunt-*` skills cover individual vulnerability classes and platform
surfaces. Use `find` or `rg` to locate a topic:

```bash
find . -name SKILL.md -print | sort
rg -n "SSRF|OAuth|GraphQL|Kubernetes" --glob 'SKILL.md'
```

## Portability Contract

Skills must not assume:

- a specific orchestrator, model, or agent framework;
- a fixed host, container name, user, private IP, or SSH topology;
- a privileged shell;
- a fixed output path;
- a particular tool wrapper when a standard CLI is sufficient.

When a command needs persistent output, use `OUTPUT_DIR`, defaulting to
`./output`. When a technique requires root, raw sockets, a browser, a proxy, or
specialized hardware, the skill must say so in `Prerequisites`.

## Quality and Safety

The quality baseline lives in [STYLE.md](STYLE.md). Contributor guidance lives
in [AGENTS.md](AGENTS.md). The operating principles live in [SOUL.md](SOUL.md).

Run the catalog validator before reviewing a change:

```bash
python3 scripts/validate_skills.py
```

Structural errors fail the command. Existing style debt is reported separately
as warnings so it can be improved incrementally.

## License

MIT. See [LICENSE](LICENSE).
