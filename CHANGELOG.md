# Changelog

## 1.0.0 (2026-11-10)

First public release. No changes to the audit rules since 0.5.2.

- Validation completed: regression on both seeded repositories in Full mode, Quick mode (42 of 42), a near-clean project, and a re-audit of a production project.
- README: Quick mode results; remaining limitation is the report path for a project with no findings.

## 0.5.2 (2026-11-10)

Changes from the 0.5.1 validation round (two seeded repositories and one near-clean project):

- Severity: consumption defects inside working caps that do not affect other tenants are Medium at most.
- Severity: weak guardrails are Low when structural controls in code already block the impact.
- README: validation results of 0.5.1.

## 0.5.1 (2026-11-10)

Changes from the 0.5.0 re-audit of a production project:

- Step 0: asks for a previous audit report; the report gets a "Status of the previous audit" section.
- Coverage matrix: columns may be merged but never dropped; "n/a" needs a reason.
- Step 5: every empty, "no", "—" or "?" cell of the matrix and inventory is resolved before writing the report.
- Rule 7 applies to every library a finding depends on, including provider SDKs.
- Severity: correctness-only defects are Low at most.
- README: validation results of the re-audit.

## 0.5.0 (2026-11-06)

- Packaged as a multi-file skill: detection patterns, report template, OWASP map and probes moved to `references/`; changelog and maintenance notes moved out of `SKILL.md`.
- Apache-2.0 license, NOTICE and bilingual README.
- Step 0: one round of six questions (adds project maturity and mode); answered questions are confirmed, not asked again; maturity never lowers technical severity; one report per project.
- Quick mode: reduced coverage matrix, no data store inventory.
- Coverage matrix: caches column.
- Rule 7: reading library bytecode or sources needs no permission; running anything needs explicit permission and happens outside the project. Rule 12: never invent findings.
- OWASP 2026 IDs first, with 2025 and Agentic (ASI01–ASI10) IDs.
- New checks: staff-only data in prompts, user text as template parameter, case-insensitive delimiter stripping (B04); filter expressions via builder APIs, context propagation into async, reactive and streaming code, silent default tenant (B05); persistence entities as tool arguments, server-side amounts, state checks and idempotency, agent-to-agent messages as untrusted (B06); reasoning blocks, strict schema mode, HTML email (B07); default shared conversation IDs, custom stores that really delete, caches keyed by user or tenant (B08); retries never repeat side effects, virtual-thread pinning on Java 21–23 (B09); framework prompt-logging switches, counting all tokens including failures (B10); minimal tool fields (B11); rendered HTML not persisted, non-HTML content type (B12); no milestone or snapshot versions (B14); bounded re-prompt loops, streaming output checks (B15); temperature for tool use (B17); batch jobs never mix users (B18); resource URIs checked against the caller, server prompts, client sampling and elicitation approval (B19).
- Detection: Google ADK, Koog, legacy Vertex AI SDK.
- Report: no-findings section.

## 0.4.0 (2026-11-06)

Threat-context questions mandatory unless unattended or declined; framework behavior verified against the local library (constants, ordering, parsers) before falling back to "Likely"; findings rated by their own impact with combinations in a separate "Attack chains" section and no mixed labels; protective transformations (redaction) must run before memory and cover re-sent history and tool results.

## 0.3.0 (2026-11-06)

B19 made revision-aware for MCP 2026-07-28 (stateless protocol, deprecated features, `cacheScope`, client registration); OWASP 2025/2026 mapping; date-dependent rules must get the date from code; no transactions held during model calls; LLM jobs must not block the shared scheduler; matrix and inventory gaps must end as findings or explained decisions; shorter reports (full entries only for Critical and High, merged findings, top 5 fixes); report labels in the user's language.

## 0.2.0 (2026-11-05)

Richer coverage matrix (provider filter, data sensitivity, quotas, guard position) with mandatory explanation of gaps; data store inventory including framework-created tables, diffed against deletion and export; tracing of memory across retries and fallbacks; secret scanner coverage against the project's own credentials; model-triggered operations consume quotas; sensitivity labels checked against actual content; model output does not penalize the user; configuration keys verified to bind.

## 0.1.0 (2026-11-05)

First version.

## Maintenance

- Keep principles in the checklist blocks and anything dated in "Volatile facts" in `SKILL.md`.
- Review at least quarterly and on major releases of Spring AI, LangChain4j, the MCP specification and the OWASP lists.
- Bump the version in `SKILL.md` and README files, and record here what changed and why.
- Validate changes against test repositories with seeded issues and decoys, and against real projects; track detection rate, false positives and severity accuracy.
