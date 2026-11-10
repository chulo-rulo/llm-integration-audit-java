# Report template

Write the report in the user's language, labels included (operating rule 11). Keep technical identifiers as they are.

## Sections

1. Summary: scope, mode, threat context (confirmed by the user or inferred), project maturity, overall assessment in three to five sentences, and the top 5 fixes in priority order. Mention the date the volatile facts were verified.
2. Flow map: Mermaid diagram with trust boundaries.
3. Coverage matrix: call sites × controls (columns as in Step 3; reduced columns in Quick mode).
4. Data store inventory (Full mode): store, real user key, retention, covered by deletion / export / backup.
5. Findings, ordered by severity:
   - Critical and High as full entries: ID, title, severity, confidence, OWASP reference (2026 ID, with the 2025 ID and ASI ID when useful), block, evidence (`file:line` and trace), impact, recommended fix (described, in the project's conventions), effort estimate.
   - Medium and Low in a table: ID, title, evidence, confidence, fix.
   - For demos and samples, add "expected in a demo, must be fixed before real use" where it applies.
6. Attack chains: findings that combine into a worse attack, each chain with its own severity and the IDs it uses. Omit the section if there are none.
7. Runtime checks pending: what could not be verified statically and how to verify it.
8. Blocks not applicable, one line each.
9. Good practices observed (keep them).
10. Incidental non-LLM issues, if any, briefly, with a suggestion of a general security review.
11. Status of the previous audit, only when one exists: each earlier finding as fixed, partly fixed (with the new finding that covers the rest), still open, or no longer applicable.

In the coverage matrix and the data store inventory, no cell stays empty or "—" without a reason: use a finding ID, "n/a" with the reason, a decision, or a runtime check.

Keep the findings list focused: one root cause per finding, findings closed by the same fix merged, minor hardening items in a compact table instead of full entries.

## When there are no findings, or only minor ones

A clean result is valid and must be just as traceable as a report with findings. Do not pad it with weak findings (operating rule 12). Include:

- What was reviewed: modules, call sites and the coverage matrix, so the reader can see nothing was skipped.
- Blocks covered, and blocks not applicable with the reason.
- What could not be verified (runtime behavior, provider console settings, environment variables, infrastructure in front of the app) and how to verify it.
- Residual risks that no code review closes: model behavior under new attacks, provider changes, prompts or tools added later.
- Good practices observed.
- When to re-audit: new tools or write actions, new data categories, new providers, multi-tenancy, or MCP exposure.
