# 401/403 Skill Technical Restoration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restore the broad technical 401/403 bypass playbook from PR #9 while correcting factual errors, broken references, and invalid tooling without adding disclaimers or policy guardrails.

**Architecture:** Use commit `eaf5053` as the coverage baseline and the approved design as the content contract. Keep all major technique families in one `SKILL.md`, apply concise applicability labels to legacy and stack-specific behavior, and validate coverage with shell assertions plus the repository validator.

**Tech Stack:** Markdown, YAML frontmatter, Bash syntax checks, repository Python validator, Git.

---

### Task 1: Capture the failing coverage baseline

**Files:**
- Inspect: `recon/401-403-bypass/SKILL.md`
- Reference: `docs/superpowers/specs/2026-08-31-401-403-skill-restoration-design.md`

- [ ] **Step 1: Run the technique-coverage assertions against the reduced file**

```bash
skill_file="recon/401-403-bypass/SKILL.md"
test "$(wc -l < "$skill_file")" -ge 250 &&
rg -q 'X-HTTP-Method-Override' "$skill_file" &&
rg -q 'PROPFIND|WebDAV' "$skill_file" &&
rg -q 'X-Forwarded-For' "$skill_file" &&
rg -q 'HTTP/1\.0' "$skill_file" &&
rg -q 'IIS/ASP\.NET' "$skill_file" &&
rg -q 'Tomcat/Java' "$skill_file" &&
rg -q '## Quick Reference' "$skill_file"
```

Expected: FAIL because the current 144-line rewrite omits several original
technique families.

- [ ] **Step 2: Record the original and current section inventories**

```bash
git show eaf50534771e54e6c0c666fefe0c67ad572e892f:recon/401-403-bypass/SKILL.md \
  | rg '^#{1,3} '
rg '^#{1,3} ' recon/401-403-bypass/SKILL.md
```

Expected: the original inventory includes path, method, header, protocol,
combination, technology, automation, decision-tree, and quick-reference
sections; the reduced file does not.

### Task 2: Restore the technical structure and path techniques

**Files:**
- Modify: `recon/401-403-bypass/SKILL.md`

- [ ] **Step 1: Replace the reduced introduction with technical frontmatter and overview**

Use this frontmatter exactly:

```yaml
---
name: 401-403-bypass-techniques
description: Use when a protected HTTP route returns 401 or 403 and proxy, server, framework, or parser differentials may expose an alternate access path.
version: 1.0.0
revision_date: 2026-08-31
license: MIT
platforms: [linux]
compatibility: Requires curl; byp4xx is optional.
metadata:
  tags: [recon, authorization, http, access-control, path-normalization]
  category: recon
  related_skills:
    - hunt-auth-bypass
    - hunt-http-smuggling
    - hunt-ssrf
    - hunt-idor
---
```

The overview must define the core condition: an intermediary and backend make
different decisions after path, method, header, or protocol normalization. It
must not contain a disclaimer, authorization gate, or policy language.

- [ ] **Step 2: Keep the validator-facing sections concise and technical**

Add these exact section headings:

```markdown
## When to Use
## Prerequisites
## How to Run
## Procedure
```

`When to Use` lists 401/403 routes, reverse-proxy/backend differentials, WAF
normalization, and server/framework routing. `Prerequisites` lists an HTTP
client capable of preserving raw paths and optional `byp4xx`. `How to Run`
contains one baseline `curl --path-as-is` command that records headers and body.

- [ ] **Step 3: Restore the path-manipulation matrix**

Restore distinct examples for:

```text
trailing slash and trailing dot
case sensitivity
single percent encoding
double decoding
dot-segment and duplicate-slash normalization
semicolon/path parameters
suffix and extension handling
backslash normalization
combined path transformations
```

Keep null-byte and overlong-UTF-8 behavior only as `legacy parser behavior`.
Remove the invalid claim that `%C0%6E` is an overlong encoding and do not assert
that `%C0%AE` encodes `n`. Each example must state the parser condition it tests
rather than promising a `200` response.

### Task 3: Restore method, header, protocol, and stack-specific coverage

**Files:**
- Modify: `recon/401-403-bypass/SKILL.md`

- [ ] **Step 1: Restore HTTP method techniques**

Include direct method changes, `HEAD`, `OPTIONS`, `TRACE`, `CONNECT`, WebDAV
methods, custom verbs, and these override mechanisms:

```http
X-HTTP-Method-Override: PUT
X-Method-Override: POST
X-HTTP-Method: DELETE
_method=PUT
```

Describe `HEAD` as a bodyless response and `OPTIONS` as method discovery. Do not
state that either confirms access. Mark methods with write semantics using the
technical label `state-changing method`; do not add a policy gate.

- [ ] **Step 2: Restore header-based differentials**

Include rewrite headers, forwarding/client-IP headers, encoded loopback forms,
referer/origin behavior, host routing, content-type switches, and AJAX routing.
Keep the original matrices but label them according to the consuming component:
`reverse-proxy rewrite`, `trusted-proxy configuration`, `host routing`, or
`application routing`.

- [ ] **Step 3: Restore protocol and combination sections**

Include HTTP/1.0, legacy HTTP/0.9 behavior, HTTP/2 pseudo-header routing, and
method/path/header combinations. Do not reference nonexistent relative skill
paths. Reference related skills by skill name only where useful.

- [ ] **Step 4: Restore the server/framework matrix**

Include Apache, Nginx, IIS/ASP.NET, Tomcat/Java, and Spring. Preserve old but
real behaviors with labels such as `legacy IIS`, `older Spring suffix matching`,
or `Tomcat path-parameter handling`. Remove examples that cannot be tied to the
named parser or server behavior.

### Task 4: Restore tooling, decision flow, and verification

**Files:**
- Modify: `recon/401-403-bypass/SKILL.md`

- [ ] **Step 1: Correct the automation section**

Keep `byp4xx`, `dirsearch`, `feroxbuster`, and Burp Intruder. Remove
`sting8k/403bypasser`. Use the valid command:

```bash
byp4xx -m 10 --rate 5 -xD "https://target.example/admin"
```

Explain `--rate` only as the request-rate setting and `-xD` as exclusion of
default-credential checks. Do not add a general rate-limit policy.

- [ ] **Step 2: Restore the technical decision tree and quick reference**

The decision order is:

```text
baseline -> path variants -> method variants -> headers -> protocol -> combinations -> related techniques -> automation
```

Restore a compact payload table with the transformation, target component,
stack condition, and expected comparison signal. Replace every `200 = bypass`
claim with `compare protected content and routing behavior with the denied
baseline`.

- [ ] **Step 3: Add concise technical pitfalls and verification**

Add these exact headings:

```markdown
## Pitfalls
## Verification
```

Pitfalls cover login/error pages returning `200`, redirects, cached responses,
unsupported methods, normalization at multiple layers, and legacy parser
assumptions. Verification compares status, `Location`, headers, title, body
length, stable protected-content markers, and backend behavior.

### Task 5: Verify the restored skill and commit it locally

**Files:**
- Verify: `recon/401-403-bypass/SKILL.md`

- [ ] **Step 1: Run the coverage assertions again**

```bash
skill_file="recon/401-403-bypass/SKILL.md"
test "$(wc -l < "$skill_file")" -ge 250 &&
rg -q 'X-HTTP-Method-Override' "$skill_file" &&
rg -q 'PROPFIND|WebDAV' "$skill_file" &&
rg -q 'X-Forwarded-For' "$skill_file" &&
rg -q 'HTTP/1\.0' "$skill_file" &&
rg -q 'IIS/ASP\.NET' "$skill_file" &&
rg -q 'Tomcat/Java' "$skill_file" &&
rg -q '## Quick Reference' "$skill_file"
```

Expected: PASS.

- [ ] **Step 2: Assert removed defects and unwanted policy text are absent**

```bash
! rg -n 'AI LOAD INSTRUCTION|200 = bypass|replaceall|byp4xx\.sh|sting8k/403bypasser|author:' \
  recon/401-403-bypass/SKILL.md
! rg -ni 'written authorization|explicit authorization|disclaimer|permission to test' \
  recon/401-403-bypass/SKILL.md
```

Expected: PASS with no matches.

- [ ] **Step 3: Run syntax and repository validation**

```bash
awk 'BEGIN { in_code=0 } /^```bash$/ { in_code=1; next } /^```$/ && in_code { in_code=0; next } in_code { print }' \
  recon/401-403-bypass/SKILL.md | bash -n
git diff --check
python3 scripts/validate_skills.py
```

Expected: Bash syntax succeeds, `git diff --check` is empty, and the validator
reports no structural errors. Existing repository-wide style warnings may
remain unchanged.

- [ ] **Step 4: Review the final diff against the original PR**

```bash
git diff --stat eaf50534771e54e6c0c666fefe0c67ad572e892f..HEAD -- \
  recon/401-403-bypass/SKILL.md
git diff eaf50534771e54e6c0c666fefe0c67ad572e892f..HEAD -- \
  recon/401-403-bypass/SKILL.md
```

Expected: all major original technique families remain, while the listed
factual errors, broken tools, broken references, and policy text are absent.

- [ ] **Step 5: Commit locally**

```bash
git add recon/401-403-bypass/SKILL.md
git commit -m "fix: restore technical 401-403 bypass coverage"
```

### Task 6: Publish through a new PR in `uphiago/recon-skills`

**Files:**
- No file modifications.

- [ ] **Step 1: Verify the local branch and destination**

```bash
git status --short --branch
git remote get-url origin
git ls-remote https://github.com/uphiago/recon-skills.git refs/heads/main
```

Expected: the worktree is clean, `origin` is `uphiago/recon-skills`, and the
base branch SHA is recorded before publication.

- [ ] **Step 2: Push a new branch to the repository owner remote**

```bash
git push origin HEAD:refs/heads/review/pr-9-technical-restoration
```

Expected: a new `review/pr-9-technical-restoration` branch is created in
`uphiago/recon-skills`. Do not push to `N0rmalizer/recon-skills`.

- [ ] **Step 3: Open the new PR**

```bash
gh pr create \
  --repo uphiago/recon-skills \
  --base main \
  --head review/pr-9-technical-restoration \
  --title "Restore and harden PR #9 technical skill coverage" \
  --body "Carries PR #9 into an owner-controlled branch, restores technical 401/403 bypass coverage, corrects factual/tooling errors, removes unwanted external-source and author metadata, and preserves the reviewed XSS and subdomain fixes."
```

Expected: GitHub returns the URL of a new open PR in `uphiago/recon-skills`.

- [ ] **Step 4: Verify the published PR head**

```bash
gh pr view --repo uphiago/recon-skills --json number,url,headRefName,baseRefName,state,commits
```

Expected: the PR is open, base is `main`, head is
`review/pr-9-technical-restoration`, and the local restoration commit is listed.
