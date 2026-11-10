---
name: llm-integration-audit-java
description: Audit how a Java app integrates with LLMs end to end (Spring AI, LangChain4j, SDKs, MCP): injection, guardrails, tools, memory, RAG, resilience, cost, privacy. Use to review or harden LLM code.
---

# LLM Integration Audit for Java

Version 1.0.0 · Facts last verified: 2026-11-06 · Apache-2.0 · Copyright (c) 2026 Igor Denyshchyk · Denis Art (DsA)

> Review guidance, not a security guarantee. Findings come from static reading unless the user allows running something. A clean report does not mean the integration is safe.

## Purpose and scope

Review the full flow between a Java application and any LLM: how input enters, how prompts are built, how the model is called, how tools, memory and retrieval behave, how output is handled and delivered, and how the whole thing fails, costs money and handles personal data. It also covers apps that expose or consume MCP, and multi-agent setups.

In scope: LLM-specific security (OWASP Top 10 for LLM Applications and for Agentic Applications), correctness of the integration, resilience, cost control, observability, privacy of data sent to providers, and MCP.

Out of scope: generic web security unrelated to the LLM flow (CSRF, session handling, unrelated endpoints). Mention such issues only if found incidentally, in a separate short section, and suggest a general security review.

## Reference files

Read these only when the step that needs them is reached:

- `references/detection.md`: detection patterns per client and where controls usually live (Steps 2 and 5).
- `references/report-template.md`: report structure, including the no-findings case (Step 6).
- `references/owasp-map.md`: OWASP LLM 2025/2026 and Agentic mapping (when assigning IDs).
- `references/probes.md`: adversarial probes (when suggesting tests).

## Operating rules

1. Read-only. Do not modify the project. Propose fixes in the report only.
2. Trace flows, do not tick boxes. A finding exists when you can follow the data from where it enters to where the problem happens. The checklist tells you where to look; the trace is the finding.
3. Evidence is mandatory. Every finding cites `file:line` and the path that proves it (entry point → … → sink).
4. Every finding has a confidence level:
   - Confirmed: the code path is fully traced, or framework behavior was verified in the library (rule 7).
   - Likely: depends on framework behavior or configuration you could not verify; say what to check.
   - Needs runtime check: cannot be decided statically (env vars, provider console settings, actual model behavior).
5. Try to disprove each finding before reporting it: look for a mitigation elsewhere (advisor, filter, aspect, gateway, framework default). If you cannot rule one out, lower the confidence instead of dropping or inflating it.
6. Each checklist item is tagged [REQ] (requirement: deviation is a finding) or [REC] (recommendation: deviation is a suggestion, unless the threat context makes it a requirement). Do not present recommendations as violations.
7. Framework defaults change between versions. Never assert a default from memory. Read the project's dependency versions, then verify the behavior for that exact version, in this order:
   - read the library itself when it is available locally (Maven `~/.m2/repository`, Gradle cache, sources jars): constants, default values, ordering and parsers, with `javap -constants` or the sources jar. Reading is allowed without asking;
   - running anything (a scratch test, a build, the application) requires the user's explicit permission, and happens outside the project;
   - the framework documentation for that version, if web access is available;
   - otherwise mark the point as "Likely" and say exactly what to verify.

   This applies to every library a finding depends on (provider SDKs, HTTP clients, MCP SDK, vector store clients), not only the main framework.
8. Legal and regulatory points (GDPR, EU AI Act, provider terms) are reported as "verify X", never as legal conclusions.
9. Proposed fixes follow the conventions of the audited project (style, logging language, error message language, comment policy, libraries already in use). Do not introduce a new framework when the project already has one for the same job.
10. Never print secrets found in the code or config. Mask them (first 4 chars + `…`) and recommend rotation.
11. Write the report in the language the user is using in the conversation, including labels (severity, confidence, section titles). Keep technical identifiers (class names, OWASP IDs, property keys) as they are.
12. Never invent findings to fill a report. A short report with few findings is a valid result.

## Workflow

### Step 0 — Context and mode (one round of questions)

Before reviewing, ask everything below in a single round. Skip any question the user already answered in their request, and confirm those answers in one line instead of asking again. Only infer answers from the code if the session is unattended or the user declines; then say so at the top of the report ("context inferred, not confirmed by the user").

1. Who can reach the LLM features: only the owner, authenticated users of one organization, the public, other systems?
2. What data flows through: public content, internal business data, personal data, special-category data (health, finance, minors)?
3. What can the LLM cause: only text, writes to the app's data, external actions (emails, payments, deletions, calls to third parties)? Offer "whatever the code allows" as an option.
4. Is it multi-tenant (several customers or users whose data must never mix)?
5. Project maturity: demo or sample, internal tool, or production?
6. Mode: Quick or Full (see Step 1).
7. Is there a previous audit report of this project? If so, read it and report the status of each earlier finding.

Use the answers to set severities (see Severity) and to decide which [REC] items become [REQ]. Maturity never lowers the technical severity of a finding; for demos and samples, add the note "expected in a demo, must be fixed before real use" where it applies, so the report stays useful to whoever copies the code.

If the user asks to audit several projects, produce one report per project, each with its own context. For large projects, recommend one session per project.

### Step 1 — Modes

- Quick: stack detection, flow map, a reduced coverage matrix (call site × input guard × output guard × identity source × token cap and timeout), and only the Quick checks (Q1–Q15). No data store inventory. Good for a first pass or a pull request.
- Full: everything below.

### Step 2 — Stack detection (block B00)

Identify clients, providers, models, channels and deployment. Use `references/detection.md`. Record versions from `pom.xml` / `build.gradle(.kts)`.

### Step 3 — Flow map and inventory (block B01)

- List every place where the app calls a model, including the easy-to-miss ones: translation, classification, intent detection, summarization, embeddings, moderation, transcription, vision, image generation, scheduled jobs, startup or health checks that call a model, and calls from one agent to another.
- For each call site record: entry point, provider/model, prompt source, untrusted inputs, tools available, memory used, output destination, guardrails applied.
- Draw a Mermaid flow diagram with trust boundaries (user input, external content, model output, tool results, messages between agents).
- Build the coverage matrix. One row per call site, with these columns:
  - input guard, and whether it runs before any model call on that input;
  - output guard, and its position relative to memory writes, tool side effects and delivery;
  - provider and model, and the provider-side safety filter (on, off, configured how);
  - data sensitivity of what is sent, and whether that provider is allowed for it;
  - quota or budget consumed (and whether model-triggered paths consume it too);
  - output token cap, timeout, retries;
  - memory read and written (conversation ID);
  - caches of prompts, answers or embeddings, and their key;
  - logging and telemetry of content.

  Columns may be merged to keep the table readable, but never dropped; a column that does not apply to a row shows "n/a" with the reason. Every empty or "no" cell is either explained (a documented decision, or not applicable) or reported. Gaps in this matrix are usually the most valuable findings: compare rows with each other, an inconsistency between similar call sites is a strong signal.
- Full mode only: build the data store inventory, every place where user data or user-linked content ends up:
  - tables created by migrations;
  - tables created by frameworks at runtime (for example JDBC chat memory or vector store tables created by `initialize-schema` or auto-DDL options), which schema tests based on migrations do not see;
  - in-memory maps, caches and queues;
  - files and object storage;
  - logs, metrics labels, error trackers;
  - copies at third parties (LLM provider retention, hosted vector databases, search or extraction APIs).

  For each store record: the real key that links it to a user (it may be a different column, such as `conversation_id` with feature prefixes, or a metadata field), retention, and whether user deletion, export and backup cover it. Diff this inventory against the deletion and export code: every store not covered is a candidate finding.

### Step 4 — Review

For each flow, walk it end to end and apply the relevant blocks (B02–B19). In Quick mode apply only Q1–Q15. Skip blocks that do not apply and say so in one line in the report.

### Step 5 — Verify

Re-read the code for every candidate finding, apply rules 5 and 7, assign confidence, and merge duplicates:

- One root cause = one finding, listing all affected locations.
- Findings closed by the same fix are merged into one (mention the secondary effects inside it).
- Before writing the report, go through every cell of the coverage matrix and the data store inventory that is empty, "no", "—", "?" or "not covered", and resolve each one as a finding, an explained decision, "n/a" with the reason, or a runtime check. A gap that only appears in a table is easy to miss.

### Step 6 — Report

Produce the report with `references/report-template.md`. Keep it usable: full entries only for Critical and High; Medium, Low and hardening go in compact tables. If there are more than about 15 Critical and High findings, keep full entries for the most severe and group the rest by root cause. If there are no findings, follow the no-findings section of the template. Then ask whether to save it as `LLM_AUDIT.md` at the project root.

## Severity

- Critical: exploitable with no special access and leads to data exposure across users or tenants, unauthorized actions, credential leakage, or unbounded spend.
- High: exploitable by a normal user, or a realistic indirect injection path, leading to data exposure, harmful actions, or major cost or availability impact.
- Medium: requires specific conditions, partial impact, or weakens a defense layer that others still cover.
- Low: hardening, defense in depth, maintainability of a security control.
- Info: observation or good practice worth keeping.

A defect that only affects correctness (duplicated or corrupted history, wrong output, inconsistent state) with no security, privacy, cost or availability impact is Low at most; say so in the finding.

A consumption or cost defect that stays inside caps that already work (per-user rate limit, output token cap, tool-call limit, tenant or user budget) and does not affect other tenants is Medium at most. It is High or Critical only when it bypasses those caps, makes spend unbounded, or lets one user or tenant exhaust shared capacity for others.

A weak guardrail (keyword lists, missing normalization, narrow output checks) is Low when structural controls in code already block the impact (identity from server context, validated tool arguments, no destructive tools, scoped retrieval). It is Medium or higher only when the guardrail is the main or only control for that risk.

Adjust with the threat context: a single-owner personal tool lowers exposure-based severities; public or multi-tenant raises them; the ability to trigger external actions raises anything touching tools or output.

Rate each finding by its own impact and exploitability. When several findings combine into a worse attack (for example, an injection path plus an over-privileged tool plus an exfiltration channel), keep each finding at its own severity and describe the combination in a separate "Attack chains" section, with its own severity. Use only the five labels above; no mixed labels such as "High-Critical".

## Quick checks (Q1–Q15)

- Q1 Secrets: provider keys not in the repo, not in client code, not in URLs or query strings. (B02)
- Q2 Consumption: who can call is controlled; per-user rate limit; per-user or global budget, also consumed by model-triggered paths; input size cap; output token cap. (B03)
- Q3 Prompt roles: user, external or other-agent text never placed in the system role or concatenated into instructions. (B04)
- Q4 Untrusted content: retrieved, fetched, uploaded, tool-returned and agent-to-agent content is delimited and treated as data. (B05, B06)
- Q5 Tool authority: identity and tenant come from server context, never from model arguments; least privilege; confirmation for destructive actions outside the model. (B06)
- Q6 Output execution: model output is never executed or interpolated into SQL, shell, expression languages, templates or reflection; structured output is validated by meaning, not just shape. (B07)
- Q7 Output delivery: escaping matches the channel; links validated; nothing auto-loads model-controlled URLs (images, previews). (B07, B12)
- Q8 Isolation: conversation memory, vector search and response caches are scoped by server-assigned user or tenant. (B05, B08)
- Q9 Output guard placement: output checks run before persistence, memory and delivery, on every call site. (B08, B15)
- Q10 Guard ordering: no model call happens before the input verdict; text is re-checked after any transformation (translation, transcription, OCR). (B15)
- Q11 Timeouts: every call has connect and read timeouts; the total of a fallback chain fits within the caller's timeout; no stacked retry layers; retries never repeat side effects. (B09)
- Q12 Fallback parity: fallback providers apply the same safety, token limits and data-handling rules as the primary. (B09, B17)
- Q13 Personal data: minimization before sending; sensitivity labels match the content sent; provider data-use terms checked; erasure covers every store, including framework-created tables. (B11)
- Q14 Telemetry: logs, traces and error trackers contain no prompts, responses, reasoning blocks or personal data unless explicitly intended and protected. (B10)
- Q15 MCP: protocol revision identified; if the app is an MCP server, every tool authorizes per user and tokens are audience-bound and never passed through; if it is a client, servers are allowlisted and tool results are untrusted. (B19)

## Checklist blocks

Block headings give OWASP LLM 2026 IDs, with the 2025 IDs after "2025:", and OWASP Agentic (ASI) IDs where they apply. Full mapping in `references/owasp-map.md`. In the report, use the newest edition you verified and give the older ID in brackets when it differs.

### B00 — Stack detection

- Clients and frameworks in use, with versions (see `references/detection.md`).
- Providers and model IDs; where they are configured.
- Channels in and out: REST, WebSocket, SSE, messaging bots, email, queues, scheduled jobs.
- Deployment constraints that matter for the flow: inbound proxy or gateway timeouts, number of instances, restarts (affects in-memory state), Java version (affects virtual threads).

### B01 — Flow map and inventory

- [REQ] Complete inventory of model call sites (see Step 3).
- [REQ] Trust boundaries marked.
- [REQ] Coverage matrix built, every gap explained or reported.
- [REQ] Full mode: data store inventory built and diffed against deletion and export code, including framework-created tables.
- [REC] One central factory or gateway for creating model clients, so that policies apply everywhere.

### B02 — Secrets and configuration (LLM02, LLM04 · 2025: LLM02, LLM03)

- [REQ] Keys come from environment or a secret manager; none in the repo or its history, none in client bundles.
- [REQ] No keys in URLs or query strings (they leak into logs, proxies and error messages); use headers.
- [REQ] Management endpoints (actuator `env`, `configprops`, `heapdump`, `loggers`) are not exposed or are protected.
- [REQ] Configuration keys actually bind: a property under the wrong path or with a typo is usually ignored without warning. For security- or cost-relevant settings (safety settings, grounding, token caps, timeouts, retries), confirm the key exists for the framework version in use; recommend verifying at runtime (configuration properties endpoint or a test that reads the bound options).
- [REC] Separate keys per environment and per purpose, with provider-side spend limits.
- [REC] Model IDs configurable, not hard-coded.
- [REC] Required configuration validated at startup with a clear failure.
- [REC] Rotate any key that was exposed, including to audit tools.

### B03 — Access and consumption (LLM06 · 2025: LLM10)

- [REQ] LLM-backed entry points require authentication appropriate to the threat context, or an explicit allowlist.
- [REQ] Per-user rate limiting; global limit for shared quotas.
- [REQ] Input size cap before any model call.
- [REQ] Output token cap on every call (max tokens or the provider equivalent, including reasoning-token limits where they apply).
- [REQ] Budget per user or tenant (requests, tokens or cost) where the public or many users can reach the feature.
- [REQ] Paid operations triggered by the model (control tokens in its output, tool calls, scheduled jobs it created) consume the same quotas and budgets as the same operation triggered by the user. Check where each quota is decremented: if it is in a handler for the manual path, the model path probably bypasses it.
- [REC] Separate quotas per feature; reserve scarce quota for essential features.
- [REC] Limits and counters survive restarts and work across instances, or the gap is a documented decision.
- [REC] Concurrency limit for expensive operations (vision, documents, long generations).

### B04 — Prompt construction (LLM01, LLM08 · 2025: LLM01, LLM07 · ASI01)

- [REQ] System instructions and user content in separate roles; no concatenation of user, external or other-agent text into the system prompt.
- [REQ] No secrets, credentials, internal URLs, staff-only data or authorization rules that the model is trusted to enforce in prompts.
- [REQ] Template variables are escaped for the template engine in use (some engines interpret braces or expressions); user text is passed as a parameter, never concatenated into the template.
- [REQ] Delimiters around untrusted content, and those delimiter strings stripped from the content itself (case-insensitive, with spacing variants).
- [REQ] Rules that depend on the current date or time (deadlines, return windows, expiry) get it from code (a clock), never from the model's own knowledge; better still, the rule is evaluated in code.
- [REC] Other context injected into prompts (time zone, user settings, catalog data) generated by code and consistent with reality.
- [REC] Model outputs are not fed back into later prompts without filtering (self-reinforcing errors, injected content persisting).
- [REC] Prompts versioned and changed through review, with regression evals.
- [REC] Do not rely on prompt instructions alone for anything security-relevant; enforce in code.

### B05 — External content and RAG (LLM01, LLM09 · 2025: LLM01, LLM08 · ASI01, ASI06)

- [REQ] All retrieved, fetched, uploaded or provider-grounded content is treated as untrusted data and delimited.
- [REQ] Vector search and document retrieval are filtered by the server-assigned user or tenant; filter expressions are built with the framework's builder API, never by concatenating user input into a filter string.
- [REQ] URL fetching (directly or via tools) blocks private, loopback, link-local and metadata addresses, and re-checks after redirects and DNS resolution (SSRF, DNS rebinding).
- [REQ] Size and page limits on ingested content.
- [REQ] Context needed for isolation (tenant, user) reaches asynchronous and reactive code explicitly: `ThreadLocal` values are lost in `@Async` executors, Reactor pipelines and streaming tool calls unless propagated; a silent default tenant when the value is missing is a finding.
- [REC] Stored content that will be retrieved later is screened at ingestion, not only at query time (stored injection).
- [REC] Embedding model and dimension are checked against the store; re-index when the model changes.
- [REC] Sources shown to the user when answers rely on retrieved or web content.

### B06 — Tools and agency (LLM03, LLM01 · 2025: LLM06, LLM01 · ASI02, ASI03, ASI05, ASI07, ASI09)

- [REQ] User identity and tenant reach tools from server-side context (tool context, memory ID, security context), never from model-generated arguments or structured output fields.
- [REQ] Each tool re-checks authorization for the specific object it touches (no IDOR via model-chosen IDs).
- [REQ] Tool arguments validated (types, ranges, enums, lengths) in code; tools never accept persistence entities directly (IDs and collections in an entity let the model overwrite or attach records).
- [REQ] Amounts, prices and other values with business impact come from server-side sources, not from model arguments.
- [REQ] Destructive or external actions require explicit user confirmation, with expiry, outside the model's control; state checks (for example "only pay a booking that is still held") and idempotency keys prevent repeating them.
- [REQ] Limit on tool-call iterations per request.
- [REQ] Tool results that contain external or third-party content are treated as untrusted.
- [REQ] Messages from one agent to another (plans, summaries, delegated tasks) are untrusted input for the receiving agent: never placed in its system role, delimited, and validated when they drive tools.
- [REC] Critical computations (dates, amounts, permissions) done deterministically in code, not by the model.
- [REC] Expose to the model only the tools needed for that flow.
- [REC] Side effects that cannot be undone do not happen before the output guard decides; or the guard decision does not depend on them.
- [REC] Audit log of tool invocations (who, what, result), without personal data in the arguments.

### B07 — Output handling (LLM10, LLM02, LLM07 · 2025: LLM05, LLM02, LLM09 · ASI05)

- [REQ] Model output never executed or interpolated into SQL, shell, SpEL or other expression languages, templates, reflection or deserialization.
- [REQ] Structured output validated by meaning (ranges, allowed values, referential checks); invalid output fails closed or degrades explicitly.
- [REQ] Escaping matches the channel (HTML, Markdown, email HTML, messaging markup, CSV, PDF).
- [REQ] Links: scheme allowlist; domain allowlist or explicit marking where links can carry data out.
- [REQ] Anything the channel loads automatically from model-controlled URLs (images in web pages and HTML emails, link previews, unfurls) is disabled or restricted: it is a zero-click exfiltration channel.
- [REQ] Control tokens the model emits in free text (for example, a marker that triggers a search) are validated, bounded and treated as possibly injected.
- [REQ] Reasoning or thinking content returned by the provider does not reach users, logs or memory unless that is a deliberate, documented choice.
- [REC] Use the provider's strict structured-output mode (JSON Schema enforcement, strict tool schemas) where it exists, in addition to validation in code.
- [REC] Handle empty responses, truncation (length stop reason), refusals and unexpected language or format explicitly.
- [REC] Secret and personal-data scanning on output before delivery.

### B08 — Conversation memory and caches (LLM02 · ASI06)

- [REQ] Conversation IDs assigned by the server from the authenticated user or tenant, never taken from the client and never left at a framework default shared by everyone.
- [REQ] Output guard runs before the response is written to memory; blocked responses never enter memory.
- [REQ] Any transformation of user content meant to protect it (redaction, pseudonymization, filtering) runs before the memory component stores it, and also covers the history that is re-sent on later turns and the results of tools. Check the actual execution order of advisors, interceptors or listeners for the framework version (see rule 7), not the order in which they are declared.
- [REQ] Retries and provider fallbacks do not duplicate or corrupt memory. Trace each fallback or retry path that uses a memory component: write down what is stored before the call (often the user message), what is stored after it, and what remains when the first attempt fails and the second one runs with the same conversation ID.
- [REQ] Memory is deleted when the user deletes their data, using the real key and every prefix or variant in use; custom memory stores implement deletion for real.
- [REQ] Caches of answers, prompts or embeddings (application caches such as `@Cacheable`, semantic caches, provider-side prompt caching keyed by content) include the user or tenant in the key whenever the cached content depends on them; check the key actually used, not the intent.
- [REC] Memory window bounded; inactive conversations evicted; separate conversation IDs per feature that should not share context.
- [REC] Retention period defined and enforced; storage protected appropriately for the data it holds.

### B09 — Resilience (LLM06 · 2025: LLM10 · ASI08)

- [REQ] Connect and read timeouts on every model and tool call.
- [REQ] The sum of timeouts and retries across a fallback chain fits within the timeout of whoever waits for the answer (proxy, gateway, webhook, client); otherwise process asynchronously.
- [REQ] Retry layers are not stacked unknowingly (SDK retries plus framework retries plus custom retries).
- [REQ] Retries only on retryable errors, with backoff and jitter, honoring Retry-After; a retry never re-runs steps that already had side effects (tool calls, payments, emails).
- [REQ] Errors shown to users reveal no internals (stack traces, provider messages, prompts).
- [REQ] No database transaction or lock is held open while waiting for a model, tool or external HTTP call: read, call, then write in a short transaction.
- [REC] Circuit breakers per model or endpoint, not only per provider.
- [REC] Fallback providers apply the same policy as the primary (safety settings, token caps, data handling); differences are documented.
- [REC] Inbound event deduplication (webhooks, queues) so a redelivery does not trigger a second model call or action.
- [REC] Streaming: cancel the upstream call when the client disconnects; apply output checks to streamed content (buffer, chunk-level checks or post-check with retraction).
- [REC] Bulkhead or concurrency limits so slow providers do not exhaust threads. On Java 21–23 with virtual threads, blocking model calls inside `synchronized` blocks pin carrier threads; Java 24 and later removed that case (JEP 491).

### B10 — Cost and observability (LLM06 · 2025: LLM10)

- [REQ] Usage counted at one central point that every call passes through (for example, framework observations or a client wrapper), not per call site.
- [REQ] Logs, traces and third-party error trackers (including breadcrumbs, exception messages, framework request/response logging and prompt-logging properties) exclude prompts, responses, reasoning content and personal data, or protect them deliberately.
- [REC] Count input and output tokens of every call, including failed attempts, retries, guardrail re-prompts and auxiliary agents.
- [REC] Metric types fit the data: counters for events, gauges for current state.
- [REC] Verify that the metrics exporter actually exists at runtime, not only that it is configured.
- [REC] Latency measured until the call really finishes, not until the caller stops waiting.
- [REC] Alerts that do not flap (hysteresis or minimum duration).
- [REC] Correlation ID across the steps of one request.

### B11 — Personal data and regulation (LLM02)

- [REQ] Personal data sent to providers is minimized; pseudonymize where possible; tools return the minimum fields needed, never whole records.
- [REQ] Each call is classified by data sensitivity, and routing to providers respects that classification.
- [REQ] Sensitivity labels are checked against the content actually sent, not only the feature's intent: a query written by the user, or by the model from the conversation, is user content; a summary of user documents is user content.
- [REQ] Verify the provider's data-use terms for the plan in use (training on inputs, retention, human review, region). Free tiers often differ from paid ones.
- [REQ] User data deletion covers every store in the data store inventory: conversation memory, embeddings, caches, guardrail and audit logs, and any copy held for retries.
- [REC] Data export includes the same stores.
- [REC] Verify whether EU AI Act transparency duties apply (users told they interact with AI; AI-generated content marked where required), and whether a DPA with each provider is needed.

### B12 — Client and channel (LLM10 · 2025: LLM05)

- [REQ] No provider keys or direct provider calls from browser or mobile code.
- [REQ] Safe rendering for the client technology (for example, no raw HTML insertion of model output, no sanitizer bypasses, Markdown rendered with a sanitizer and URL scheme allowlist); rendered HTML is not persisted and re-injected (for example via browser storage).
- [REQ] Responses returned as plain strings declare a non-HTML content type, so a browser does not render model output as HTML.
- [REC] Messaging channels: parse mode and markup escaped; link previews controlled (see B07).
- [REC] Streaming consumed with backpressure and cancellation.

### B13 — Testing and evals

- [REQ] Unit or integration tests with the model mocked (or HTTP-level stubs) for every call site's error handling.
- [REQ] Adversarial tests for each guard (see `references/probes.md`).
- [REC] Tests that fallback and retry paths keep memory consistent and do not repeat side effects.
- [REC] Startup and dependency wiring tests (misconfigured beans often break only in production).
- [REC] Regression evals when prompts or models change.

### B14 — Dependencies (LLM04 · 2025: LLM03 · ASI04)

- [REQ] Known CVEs in LLM client libraries, MCP SDKs and document parsers.
- [REC] Versions reasonably current and pinned; milestone or snapshot versions avoided in anything handling real data; update policy in place.

### B15 — Guardrails (LLM01, LLM02, LLM10, LLM08, LLM07 · 2025: LLM01, LLM02, LLM05, LLM07, LLM09 · ASI01)

Input side:

- [REQ] Input checks run before any model call on that input, including auxiliary calls such as translation or intent detection.
- [REQ] Normalization before matching: Unicode NFKC, removal of format characters (`\p{Cf}`, zero-width, bidi), homoglyphs, decoding of common encodings.
- [REQ] Text is re-checked after any transformation (translation, transcription, OCR, document extraction).
- [REC] Topic or scope restriction for single-purpose assistants.

Classifier (if a safety model is used):

- [REQ] The whole text is classified: no truncation that leaves unchecked sections (use overlapping windows).
- [REQ] Text placed in the classifier prompt is escaped; the classifier itself can be prompt-injected (closing delimiters, fake verdicts).
- [REQ] The classifier supports the application's languages.
- [REQ] Explicit behavior on classifier failure (fail-open or fail-closed), chosen per flow and documented.
- [REC] Check how fail-closed interacts with circuit breakers: an open breaker under fail-closed blocks everything.
- [REC] Timeouts, fallback classifier, and metrics for verdicts, errors and latency.

Output side:

- [REQ] Output checks on every call site that reaches a user, or a common filter at the single exit point (ideally both), including streaming paths.
- [REQ] Output checks happen before persistence, memory and delivery (see B08).
- [REQ] Outputs transformed by another model after the check (translation, rewriting) are checked again.
- [REQ] Secret scanning on output covers every credential type the project itself uses: list the credentials in configuration (providers, search APIs, messaging tokens, cloud accounts, databases) and check that the scanner's patterns match each format.
- [REC] Hidden context leakage detection (canary token, fingerprints of the scaffolding).

General:

- [REQ] Guardrails are not the only defense (least privilege, validation and isolation still apply).
- [REQ] The main model does not police itself in the same prompt as the only control.
- [REQ] Unsafe content produced by the model is not counted as a violation by the user: only input verdicts feed per-user penalties; output verdicts feed metrics and alerts.
- [REQ] Re-prompt or retry loops driven by guardrails are bounded to a small number.
- [REC] Repeated violations tracked per user with temporary blocking and an admin alert.
- [REC] Guardrail blocks do not count as provider failures in circuit breakers.
- [REC] Thresholds configurable; false positives reviewable.

### B16 — Multimodal (LLM01 · ASI01)

- [REQ] Size, duration and page limits for audio, images and documents.
- [REQ] Text obtained from media (transcripts, OCR, vision descriptions) goes through input checks like typed text.
- [REQ] Instructions hidden in images, documents and metadata are considered (indirect injection); at minimum the prompt treats media text as data.
- [REC] Provider content-filter errors mapped to clear user messages.
- [REC] Generated media checked or labeled according to the threat context.

### B17 — Model and provider lifecycle

- [REQ] Model IDs pinned and configurable.
- [REC] Automatic detection of retired or failing models (catalog checks, health probes) with an alert.
- [REC] Embedding dimension checked on startup or periodically.
- [REC] Endpoints without an SLA (free or preview tiers) are treated as unreliable, with fallback and monitoring.
- [REC] Behavioral differences between providers documented (safety filters, tool-calling reliability, language quirks); sampling settings suited to tool use (low temperature) on models that can write data.

### B18 — Scheduled and background work (LLM03, LLM06 · 2025: LLM06, LLM10 · ASI08, ASI10)

- [REQ] Limit on how many scheduled or recurring jobs the model can create per user.
- [REQ] Scheduled executions have their own budget and do not bypass per-request limits.
- [REQ] Guards run again at execution time, not only when the job was created.
- [REQ] Failed runs are retried a bounded number of times and the user is told; failures are not silently dropped and not retried forever.
- [REQ] Long or retrying LLM work does not block a shared scheduler pool (the default pool can be a single thread): use a dedicated executor, timeouts and a per-run cap on items.
- [REQ] Batch jobs never mix data of different users or tenants in one prompt.
- [REC] Jobs process only new or changed items instead of re-sending the same data on every run.

### B19 — MCP (LLM03, LLM01, LLM04 · 2025: LLM06, LLM01, LLM03 · ASI02, ASI03, ASI04, ASI07)

First identify the MCP protocol revision(s) the SDK or server in use implements, and compare it with the current revision in Volatile facts. Several features changed or were deprecated between revisions; apply the points that match the implemented revision and report deprecated features in use.

The app as MCP server:

- [REQ] Protocol revision identified; deprecated features (for example HTTP+SSE transport, Sampling, Roots, Logging as of 2026-07-28) not adopted in new code.
- [REQ] HTTP transport authenticates according to the MCP authorization spec of that revision (OAuth-based, the server is a resource server): token audience validated, issuer validated, Protected Resource Metadata (RFC 9728) published, and tokens received from clients never passed through to other APIs (use the server's own credentials or token exchange).
- [REQ] Every tool and resource authorizes per user and object, not only "token is valid"; resource URIs with user identifiers (for example `profile://{username}`) are checked against the caller; scopes per tool for sensitive operations.
- [REQ] Input schemas strict and validated server-side; classic injections inside tools (SQL, command, path traversal in resources and file arguments, SSRF) checked.
- [REQ] Local HTTP servers bind to localhost and validate the Origin header (DNS rebinding).
- [REQ] State across calls cannot be hijacked: in revisions with protocol sessions (up to 2025-11-25) session IDs are random and never used as authentication; in 2026-07-28 and later, server-minted state handles passed as tool arguments are unguessable and bound to the caller.
- [REQ] Tool results contain no other users' data, secrets or internal stack traces; third-party content in results is marked as untrusted.
- [REQ] Tool annotations (read-only, destructive) are hints for clients; enforcement and confirmation of destructive actions happen on the server.
- [REQ] Per-user responses are not marked as publicly cacheable (from 2026-07-28 list and read results carry `cacheScope`; per-user content must be `private`).
- [REC] Prompts exposed by the server do not interpolate arguments into instructions without escaping, and do not steer the client toward destructive tools.
- [REC] Tool descriptions stable and versioned (silent changes after approval = rug pull); deterministic tool order.
- [REC] Requests for more input from the client (elicitation; from 2026-07-28 via multi round-trip `input_required` results) are treated as untrusted client input when they come back, and cost and data sent are controlled.
- [REC] Rate limits per client and user; response size limits and pagination.
- [REC] Audit log of tool calls.

The app as MCP client:

- [REQ] Server allowlist; credentials per server, sent in headers, never in URLs.
- [REQ] Tool results treated as untrusted and passed through guards.
- [REQ] Timeouts, iteration limits and circuit breaking for MCP calls.
- [REQ] Sampling and elicitation requests from servers are not auto-approved: a human or an explicit policy decides, and their content is not logged in full.
- [REC] Client registration and credentials follow the revision: credentials keyed by issuer and not reused with another authorization server; `iss` in authorization responses validated; Client ID Metadata Documents preferred where Dynamic Client Registration is deprecated.
- [REC] Tool descriptions pinned or reviewed; detect changes (tool poisoning); prevent cross-server name shadowing.
- [REC] Expose to the model only the needed tools.
- [REC] Proxy-style servers checked for confused-deputy issues (static client IDs, consent).

## Volatile facts (last verified 2026-11-06)

These change. Before relying on them, check the current source if web access is available, and mention in the report the date you verified them.

- OWASP Top 10 for LLM Applications: a 2026 edition exists (published around August 2026). OWASP Top 10 for Agentic Applications: published December 2025 (ASI01–ASI10). Mapping in `references/owasp-map.md`. Check genai.owasp.org for newer editions and remap if one exists.
- MCP specification: current revision 2026-07-28 (previous: 2025-11-25). Main changes in 2026-07-28: the protocol is stateless (no `initialize` handshake, no `Mcp-Session-Id`); `server/discover` added; server-initiated requests (elicitation, sampling, roots) replaced by multi round-trip requests with `input_required` results; Sampling, Roots and Logging deprecated; HTTP+SSE transport deprecated; Dynamic Client Registration deprecated in favor of Client ID Metadata Documents; list and read results carry `ttlMs` and `cacheScope`. Since 2025-06-18 the authorization model treats servers as OAuth resource servers with audience-bound tokens, Protected Resource Metadata, and no token passthrough. SDKs often lag behind the specification: check which revision the project's SDK implements. Source: modelcontextprotocol.io/specification.
- EU AI Act: transparency obligations for systems interacting with people were scheduled to apply from August 2026; implementation timelines have been under revision. Verify the current status before mentioning dates.
- Framework defaults (retries, timeouts, memory behavior and default conversation IDs, advisor order, tool-call iteration limits, scheduler pool size): never assumed; verify for the project's exact version (rule 7).
