# llm-integration-audit-java

[Español](README.es.md)

A Claude skill that audits how a Java application integrates with large language models, end to end: how input enters, how prompts are built, how models are called, how tools, memory and retrieval behave, how output is handled and delivered, and how the whole flow fails, costs money and handles personal data. It also covers apps that expose or consume MCP, and multi-agent setups.

Version 1.0.0 · Apache-2.0 · Copyright (c) 2026 Igor Denyshchyk · Denis Art (DsA)

## What it covers

- Clients: Spring AI, LangChain4j (plain, Quarkus, Micronaut, Helidon), Semantic Kernel, Embabel, LangGraph4j, Google ADK, Koog, MCP Java SDK, Anthropic, OpenAI, Google Gen AI, AWS Bedrock and Azure SDKs, local models, and raw HTTP calls to providers.
- 20 checklist blocks (B00–B19): secrets, access and consumption, prompt construction, RAG, tools and agency, output handling, memory and caches, resilience, cost and observability, personal data, client channel, tests, dependencies, guardrails, multimodal, model lifecycle, background jobs, MCP.
- Mapped to the OWASP Top 10 for LLM Applications (2025 and 2026) and the OWASP Top 10 for Agentic Applications.
- Two modes: Quick (first pass or pull request) and Full.

## How it works

1. Asks one round of context questions: who reaches the LLM features, what data flows, what the LLM can cause, multi-tenancy, project maturity, mode.
2. Detects the stack and versions.
3. Maps every model call site, draws the flow with trust boundaries, and builds a coverage matrix and (in Full mode) an inventory of every store holding user data, including tables created by frameworks at runtime.
4. Traces each flow through the relevant blocks.
5. Verifies each finding, looks for mitigations elsewhere, and checks framework behavior against the library version in use.
6. Writes a report with evidence (`file:line` and trace), severity, confidence (Confirmed / Likely / Needs runtime check), attack chains and runtime checks pending. It asks before saving it as `LLM_AUDIT.md`.

The audit is read-only: it never modifies the project and only runs code with your explicit permission.

## Install

- Claude apps (claude.ai, desktop): zip the `llm-integration-audit-java` folder and upload it in Customize > Skills. If an older single-file version is installed, replace it.
- Claude Code: copy the folder to `~/.claude/skills/` (personal) or `.claude/skills/` inside a repository (project).

## Use

Ask Claude something like "audit the LLM integration of this project" or "review how this app calls the model", with the project available to Claude. Answer the context questions; the answers set severities.

## Contents

```
SKILL.md                     workflow, rules, severity, quick checks, checklist blocks
references/detection.md      detection patterns per client, where controls live
references/report-template.md report structure, including the no-findings case
references/owasp-map.md      OWASP LLM 2025/2026 and Agentic mapping
references/probes.md         adversarial test ideas
CHANGELOG.md                 versions and maintenance notes
```

## Validation so far

Measured during development, with honest limits:

- Two seeded test repositories (Spring AI, LangChain4j, MCP server; 63 seeded issues and 27 decoys in total): 41.5 of 42 and 21 of 21 issues found, with no false positives on decoys (v0.2, v0.3 and again with v0.5.1).
- One near-clean multi-tenant project (Spring AI): both seeded issues found, no decoy reported, no invented findings; the extra findings were real defects, verified against the framework sources. Severity was one level high on consumption findings, which led to the calibration rules in 0.5.2.
- Quick mode on the first seeded repository: 42 of 42 issues found, no false positives on decoys.
- Three public open-source projects: about 25 findings checked by hand as real, none false.
- One private production project: 11 of 13 known issues found by the first version; after those were fixed, a re-audit with 0.5.0 found 3 further medium issues (configured tool-call limits not reaching the client in use, memory duplicated across provider fallback, personal text labeled as impersonal) that the first version could not detect.

Limits: the test repositories were written by the same author as the skill; the report path for a project with no findings has not been exercised; only Spring AI and LangChain4j have been exercised in depth. The best test is your own project: results from other clients, missed issues, false positives and severity disagreements are welcome as issues.

## Limitations

- Review guidance, not a security guarantee. Findings come from static reading. A clean report does not mean the integration is safe.
- Model behavior, provider settings and infrastructure in front of the app cannot be verified from code; the report lists them as runtime checks.
- Legal and regulatory points (GDPR, EU AI Act, provider terms) are reported as items to verify, never as legal advice.
- Framework defaults and standards change. The skill keeps dated facts in one section; check the "Facts last verified" date in `SKILL.md`.

## License

Apache License 2.0. See `LICENSE` and `NOTICE`.

## Author

Igor Denyshchyk · Denis Art (DsA)
