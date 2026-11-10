# OWASP mapping

Use the 2026 IDs first and add the 2025 ID in parentheses when the reader may know the older list. Check genai.owasp.org for newer editions (see Volatile facts in SKILL.md).

## OWASP Top 10 for LLM Applications, 2025 → 2026

| 2025 | 2026 | Risk | Blocks |
|---|---|---|---|
| LLM01 | LLM01 | Prompt Injection | B04, B05, B06, B15, B16, B19 |
| LLM02 | LLM02 | Sensitive Information Disclosure | B02, B07, B08, B10, B11, B15 |
| LLM06 | LLM03 | Excessive Agency | B06, B18, B19 |
| LLM03 | LLM04 | Supply Chain | B02, B14, B19 |
| LLM04 | LLM05 | Data and Model Poisoning | B05 (partly; mostly organizational) |
| LLM10 | LLM06 | Unbounded Consumption | B03, B09, B10, B18 |
| LLM09 | LLM07 | Misinformation | B05, B07 |
| LLM07 | LLM08 | System Prompt Leakage (2026: Hidden Context Exposure, wider scope: tool schemas, flow rules, any hidden context) | B04, B15 |
| LLM08 | LLM09 | Vector and Embedding Weaknesses | B05 |
| LLM05 | LLM10 | Improper Output Handling | B07, B12, B15 |

## OWASP Top 10 for Agentic Applications (December 2025)

| ID | Risk | Blocks |
|---|---|---|
| ASI01 | Agent Goal Hijack | B04, B05, B15, B16 |
| ASI02 | Tool Misuse and Exploitation | B06, B19 |
| ASI03 | Identity and Privilege Abuse | B06, B19 |
| ASI04 | Agentic Supply Chain Vulnerabilities | B14, B19 |
| ASI05 | Unexpected Code Execution | B06, B07 |
| ASI06 | Memory and Context Poisoning | B05, B08 |
| ASI07 | Insecure Inter-Agent Communication | B06, B19 |
| ASI08 | Cascading Failures | B09, B18 |
| ASI09 | Human-Agent Trust Exploitation | B06, B07 |
| ASI10 | Rogue Agents | B18 |

Blocks B01, B13 and B17 have no direct OWASP entry: they support the others (inventory, tests, model lifecycle).
