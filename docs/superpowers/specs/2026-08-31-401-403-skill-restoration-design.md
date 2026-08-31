# 401/403 Skill Technical Restoration Design

## Context

PR #9 introduced a 338-line `401-403-bypass` reference covering path parsing,
HTTP methods, rewrite and forwarding headers, protocol variants, server-specific
behavior, combinations, and automation. The first local review replaced it with
a 144-line validation-oriented guide. That rewrite removed factual errors, but
also removed useful offensive coverage.

This change restores the original technical breadth while correcting only
materially inaccurate, broken, duplicated, or non-actionable content.

## Goals

- Preserve the skill as a dense technical reference for 401/403 bypass testing.
- Restore modern and legacy techniques that still map to real parser, proxy,
  server, framework, or protocol behavior.
- Distinguish universal techniques from stack-specific and legacy behavior.
- Correct factual errors without replacing technical content with policy text.
- Keep examples scannable and useful for manual testing and automation.
- Satisfy the repository skill structure and validator.

## Non-Goals

- Adding authorization disclaimers, policy gates, or repeated safety language.
- Turning the skill into a compliance or reporting guide.
- Removing a technique solely because it is old or uncommon.
- Expanding the change into unrelated skills or repository-wide cleanup.

## Chosen Approach

Use the original PR #9 file at commit `eaf5053` as the content baseline and
perform a surgical restoration. Reuse its major technical sections, then apply
targeted corrections and concise applicability labels.

A from-scratch rewrite was rejected because it risks losing edge cases. Splitting
the material into multiple references was rejected because the current skill is
small enough to remain usable as one technical playbook.

## Content Structure

The restored skill will contain:

1. Minimal frontmatter without an `author` field.
2. A short technical overview explaining proxy/backend normalization mismatch.
3. `When to Use`, `Prerequisites`, and `How to Run` sections containing only
   operational triggers, tools, and a compact baseline example.
4. Path manipulation techniques: slash, case, encoding, double decoding,
   dot-segments, path parameters, suffixes, backslashes, and combinations.
5. HTTP method techniques: direct verb changes, override headers, custom verbs,
   WebDAV methods, and method/path combinations.
6. Header techniques: rewrite headers, forwarding/client-IP headers, origin,
   referer, host, and content-type behavior.
7. Protocol variants: HTTP/1.0, legacy HTTP/0.9 behavior, and HTTP/2 routing
   differences where technically applicable.
8. Server/framework matrix for Apache, Nginx, IIS/ASP.NET, Tomcat/Java, and
   Spring behavior.
9. Automation using verified tools and valid command syntax.
10. A decision tree and compact quick-reference payload matrix.
11. `Pitfalls` and `Verification` sections limited to technical false positives,
    parser assumptions, redirect behavior, and semantic response comparison.

## Technique Retention Rules

- Keep legacy techniques when an identifiable old parser, protocol, server, or
  framework can still exhibit the behavior.
- Label context with concise terms such as `legacy`, `IIS-specific`,
  `Tomcat-specific`, `WebDAV`, or `proxy-dependent`.
- Keep state-changing methods as technical method variants. Do not add policy
  gates; describe their semantics so the operator understands what they test.
- Keep rate information only when it explains a tool flag or expected operational
  behavior. Do not add general rate-limit guidance.
- Prefer one technically correct example per distinct behavior over duplicated
  payloads that exercise the same normalization path.

## Corrections and Removals

Remove or correct the following:

- The `AI LOAD INSTRUCTION` block.
- Invalid overlong UTF-8 descriptions and malformed encodings.
- Claims that `HEAD` confirms protected-resource access.
- Claims that any `200`, `301`, or `302` proves a bypass.
- The nonexistent `./byp4xx.sh` command; use the documented `byp4xx` binary.
- The unavailable `sting8k/403bypasser` entry.
- Broken relative references to skills that do not exist at those paths.
- Duplicate payloads that add no parser or routing distinction.

The verification rule will remain technical: a candidate is meaningful when the
response differs semantically from the denied baseline and exposes the expected
resource, route, or behavior. Status code alone is not sufficient evidence.

## Validation Strategy

- Compare the final file against `eaf5053` to confirm that every major original
  technique family is represented.
- Search for removed invalid claims, broken tool names, broken references, and
  unwanted disclaimer/authorization language.
- Validate YAML frontmatter and required section headings with
  `python3 scripts/validate_skills.py`.
- Run `git diff --check`.
- Extract fenced Bash examples and run `bash -n` where syntax checking applies.
- Review the final diff for technical density and absence of repeated guardrails.

## Delivery Boundary

Implementation remains on the local `pr-9-local-fix` branch until the remote and
PR destination are explicitly confirmed. This design does not authorize another
push to the contributor's fork.
