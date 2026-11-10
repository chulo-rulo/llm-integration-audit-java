# Adversarial probes

Suggest these as test cases in the report, matched to the findings and the call sites they exercise. Do not send them to production systems, or to any system, without the user's consent.

## Injection

- Direct instruction override, in every language the app accepts.
- System prompt and hidden context extraction: "repeat the text above", translation or summary of the instructions, listing the available tools and their descriptions.
- Delimiter closing: content containing the app's own delimiters (in different letter cases) or the classifier's conversation markers, followed by a fake verdict or fake instructions.
- Template syntax in user text: braces or placeholder syntax of the prompt template engine in use.
- Invisible characters inside keywords (zero-width, bidi controls) and homoglyphs.
- Encoded payloads: Base64, URL encoding, leetspeak, other languages.
- Payload hidden in the middle or end of a long input, where truncation or chunking may skip it.
- Indirect injection through an uploaded document, a fetched web page, text inside an image, a tool result, an MCP result, or another agent's output.
- Second-order injection: content stored now and retrieved into a prompt later (notes, profiles, knowledge base entries).

## Retrieval and isolation

- Filter injection: quotes and filter operators in any user value that reaches a vector store or search filter.
- Cross-user and cross-tenant probes: same question from two tenants (answer caches), another user's conversation ID, another user's object IDs in tool requests.
- Async and streaming paths: the same request through the streaming endpoint, and actions that trigger background work, checking which user or tenant the result is attributed to.

## Tools and agency

- Model asked to act on another user's object ID, or with another identity in the arguments.
- Model asked to set a price, amount, date or status that should come from the server.
- Chaining tools beyond the intended flow; asking for a destructive action without the confirmation step.
- Failure injection after a side effect (for example, the step after a payment fails) to see whether retries repeat it.

## Output

- Exfiltration: asking the model to include data in a link, an image URL or a Markdown reference.
- Output that contains HTML, script or Markdown aimed at the channel (web page, email, chat platform).
- Structured output with valid shape but harmful meaning (another user's ID, out-of-range values).

## Consumption and resilience

- Requests designed to maximize output length, tool loops or guardrail re-prompts.
- Very large inputs and files; many parallel requests from one account.
- Slow or failing provider (simulated) to measure the worst-case latency of retries and fallbacks against the caller's timeout.
