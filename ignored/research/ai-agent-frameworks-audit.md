# We Found 56+ Security Vulnerabilities (6 Critical RCEs) Across 13 AI Agent Frameworks - Here Is What Systematic Auditing Taught Us

Source: https://dev.to/correctover/we-found-56-security-vulnerabilities-6-critical-rces-across-13-ai-agent-frameworks-here-is-2ji6

# We Found 56+ Security Vulnerabilities (6 Critical RCEs) Across 13 AI Agent Frameworks — Here Is What Systematic Auditing Taught Us

The AI agent ecosystem is growing faster than its security practices can keep up. Over the past six weeks, we ran a systematic security audit across 13+ mainstream AI agent frameworks — CrewAI, AutoGen, AG2, LlamaIndex, Haystack, LiteLLM, Semantic Kernel, LangChain, Dify, FastMCP, MCP Python SDK, Griptape, and Docker MCP. The results were sobering: 56+ verified security vulnerabilities, including 7 that we responsibly disclosed to MSRC, ZDI, HackerOne, and GitHub Security. Six of those are Remote Code Execution (RCE) vulnerabilities with CVSS scores of 9.8 or 9.3. One is a critical SSRF that bypasses existing guardrail protections.

This is not a theoretical exercise. Every vulnerability listed here was verified with a working proof of concept and, where possible, submitted through a formal bounty channel. Some produced same-day responses from Microsoft's Security Response Center.

---

## The 7 Responsibly Disclosed Vulnerabilities

### 1. CrewAI MCP StdioTransport RCE (CVSS 9.8)

The `StdioTransport.__init__()` method in CrewAI passes user-controlled command strings directly to `stdio_client()` with zero validation. Any MCP server configuration pointing to a malicious command triggers arbitrary OS process execution. CrewAI was the most vulnerable mainstream framework we tested — no allowlist, no input sanitization, nothing. We filed this with MSRC and received a case number (126356) the same day. It is now tracked as CVE-2026-2287.

### 2. AutoGen Studio RCE (CVSS 9.8)

Microsoft AutoGen's `magentic-one-cli` accepts a `--config` parameter that is passed directly to Python's `open()` without any path validation. Combined with AutoGen's `exec()` usage in `captainagent` (another MSRC-reported RCE), an attacker who controls the configuration file path achieves full remote code execution on the host. We submitted both vectors to MSRC alongside the path traversal disclosure.

### 3. AG2 eval() \`__str__\` Bypass RCE (CVSS 9.8)

AG2 (the community fork of AutoGen) uses `eval()` in `context_expression.py` for string interpolation during agent group chat. The code does escape string values, but objects that implement a custom `__str__()` method bypass that escape mechanism entirely. The source code even contains a comment explicitly acknowledging that "custom `__str__` injection is out of scope." An LLM-controlled `context_variables` value with a crafted `__str__` method produces full RCE with no additional exploit steps. Filed as GitHub Issue #3073.

### 4. LlamaIndex Pickle Deserialization RCE (CVSS 9.8)

The `Workflows` serializer in LlamaIndex calls `pickle.loads(base64.b64decode(value))` — textbook pickle RCE against one of the most widely used LLM orchestration frameworks. The `PickleSerializer` is registered as one of the default serializers for workflow state persistence, meaning any workflow that serializes intermediate state is exploitable. A malicious workflow definition triggers arbitrary code execution at deserialization time. Filed as GitHub Issue #22296.

### 5-6. Haystack Pipeline Deserialization RCEs (CVSS 9.8 and 9.3)

Haystack 2.31.0 has two independent deserialization RCE pathways, both critical.

The first (CVSS 9.8): `Pipeline.loads()` calls `import_class_by_name()` to dynamically import any Python class from YAML or JSON input. The chain `thread_safe_import()` followed by `getattr(module, class_name)` gives an attacker full control over class instantiation — no module allowlist is enforced.

The second (CVSS 9.3): `deserialize_callable("os.system")` returns a callable `os.system` function object directly. This deserialized callable is used by `Tool.from_dict()` and `OutputAdapter.from_dict()`, meaning any pipeline loading serialized tool definitions can be hijacked. No type restrictions or module allowlists are applied.

### 7. LiteLLM Guardrail SSRF (CVSS 8.6)

LiteLLM's guardrail hook system uses `is_valid_url()` to validate URLs — but this function performs only a syntax check. The platform separately provides `validate_url()`, which performs DNS resolution, IP blacklist checking, and URL rewriting, but it is never called from the guardrail `http_request()` path. This gap allows an attacker who can inject a guardrail configuration to access cloud metadata endpoints (169.254.169.254), Azure Wire Server, and internal network services, all through a validated-by-syntax URL. Filed as GitHub Issue #32862.

---

## The Scale of the Problem

These seven disclosures are the visible part of a much larger finding. Our automated scanning using the Correctover CCS (Call Correctness Standard) v1.0 detection engine identified **56+ verified vulnerabilities** across 13+ frameworks — after false positive filtering and manual PoC verification.

The largest single source of vulnerabilities is what we call the **MCP readOnlyHint gap**. The `modelcontextprotocol/python-sdk` defines a `readOnlyHint` boolean on tools to indicate read-only intent, but the runtime never enforces it. Every downstream framework inherits this broken contract. We found 87 production code instances of unguarded tool invocation across six frameworks — AutoGen, Semantic Kernel, FastMCP, Dify, Griptape, and MCP Python SDK itself. The readOnlyHint acts as documentation only, with zero runtime consequences.

Beyond the readOnlyHint gap, three vulnerability patterns appeared consistently across frameworks:

- **Tool argument injection**: Frameworks pass LLM-generated tool call arguments directly to OS-level functions like `subprocess.Popen()` with no validation or escaping. This was most severe in CrewAI (zero mitigations) but also present in LiteLLM (allowlist with known bypasses).
- **Path traversal in config loaders**: User-controlled file paths in CLI arguments and configuration files feed directly into `open()` calls. AutoGen and Dify both had this pattern at CVSS 9.8.
- **Environment variable leakage**: CLI tools pass `os.environ` to child processes, leaking API keys and secrets. Found in MCP Python SDK and FastMCP.

---

## How We Found Them: Methodology

The audit process combines automated scanning with manual validation:

1. **24 CCS detection rules** — compiled from real vulnerability research across six months of agent framework analysis. Each rule targets a specific exploit class: eval/exec misuse, unsafe deserialization (pickle, yaml.load, import_by_name), subprocess injection, path traversal, SSRF, and environment leakage.
2. **Dual-path detection** — AST scanning identifies structural patterns (e.g., function calls to `eval()`, `pickle.loads()`, `subprocess.Popen()`) while regex verification cross-checks against known exploit signatures. This dual-path approach significantly reduces false positives.
3. **PoC-gated classification** — every candidate finding receives a working proof of concept before being classified as a confirmed vulnerability. Only PoC-verified findings enter the final count.

The 24 detection rules are published as part of the **CCS v1.0 standard** at github.com/Correctover/standards. The accompanying open dataset includes 20,000 production API traces from 13 LLM providers and 33 models — part of a total 80,000-trace corpus collected through real API measurements.

Runtime verification latency is P50=22 microseconds (measured across 1 million samples), making it feasible to insert into production agent call paths with no perceptible overhead.

---

## Why This Matters for AI Agents in Production

The vulnerabilities we found share a disturbing pattern: they are not obscure edge cases. They are fundamental architectural gaps in how agent frameworks handle untrusted input from LLMs.

Every framework we tested assumes LLM outputs are authoritative. The architecture treats "what the LLM says" as trusted and routes it directly to tool execution — with no verification layer in between. This is the equivalent of running `eval()` on user input in a web application, a practice the security industry stopped tolerating two decades ago.

The frameworks that have implemented security measures — LiteLLM's allowlist, AutoGen's partial mitigations — apply them inconsistently with known bypasses. The MCP readOnlyHint is the clearest example: a security contract defined in the protocol specification but never enforced in the runtime, giving developers a false sense of safety.

When an agent framework runs in production with access to internal APIs, databases, filesystem, and code execution, every one of these vulnerabilities becomes a potential data breach or full host compromise.

---

## Runtime Call Verification: The Missing Security Layer

What the AI agent ecosystem needs is not more per-framework security patches applied after vulnerabilities are found. It needs a **verification layer** that sits on the execution path between the LLM and tool execution — enforcing policy regardless of which framework is in use.

The Correctover CCS SDK implements this as a synchronous interceptor: a `@govern()` decorator that validates every tool call against a policy before the tool executes. Unlike observer-pattern hooks (which produce a callback but do not block execution — a CWE-636 fail-open pattern present in every major framework we audited), the CCS interceptor operates on the execution path itself. If governance crashes, the tool is blocked. Fail-closed, not fail-open.

The SDK integrates with CrewAI, AutoGen, LangChain, and LangGraph in 2-4 lines of code. The CCS v1.0 standard, detection rules, and integration kit are all open source at **github.com/Correctover**.

---

The 56+ vulnerabilities we found are not someone else's problem. If you are deploying AI agents in production today — with access to internal APIs, databases, or code execution — you are exposed to these same attack surfaces. The question is not whether your framework has these vulnerabilities, but whether you have a runtime verification layer catching them before they reach production.

---

*This research was conducted by the Correctover security research team. Responsible disclosure timeline and individual advisory details are available at [github.com/Correctover/standards](https://github.com/Correctover/standards) and [github.com/Correctover/mcp-security-audit](https://github.com/Correctover/mcp-security-audit).*
