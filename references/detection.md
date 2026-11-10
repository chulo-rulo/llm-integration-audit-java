# Detection patterns and where controls live

Used in Step 2 (stack detection) and Step 5 (looking for existing mitigations before reporting a gap).

## Detection patterns

Search imports, dependencies and configuration keys. One project often mixes several clients: record every one, with its version.

| Client | Look for |
|---|---|
| Spring AI | `org.springframework.ai`, `spring-ai-*` starters, `ChatClient`, `ChatModel`, `Advisor`, `ChatMemory`, `VectorStore`, `@Tool`, `ToolCallback`, `ToolContext`, `.entity(`, `spring.ai.*` properties |
| Spring AI MCP | `spring-ai-starter-mcp-server*`, `spring-ai-starter-mcp-client*`, `@McpTool`, `@McpResource`, `@McpPrompt`, `spring.ai.mcp.*` |
| LangChain4j | `dev.langchain4j`, `AiServices`, `@AiService`, `@SystemMessage`, `@UserMessage`, `@V`, `@Tool`, `@MemoryId`, `@ToolMemoryId`, `ChatMemoryProvider`, `ChatMemoryStore`, `ContentRetriever`, `EmbeddingStore`, `InputGuardrail`, `OutputGuardrail`, `McpToolProvider`, `langchain4j.*` properties |
| Quarkus LangChain4j | `io.quarkiverse.langchain4j`, `@RegisterAiService`, `quarkus.langchain4j.*` |
| Quarkus MCP Server | `io.quarkiverse.mcp`, `@Tool`, `@Resource`, `@Prompt` from that package |
| Micronaut / Helidon LangChain4j | `io.micronaut.langchain4j`, `io.helidon.integrations.langchain4j` |
| Semantic Kernel Java | `com.microsoft.semantickernel`, `@DefineKernelFunction` |
| Embabel | `com.embabel.agent`, `@Agent`, `@Action` |
| LangGraph4j | `org.bsc.langgraph4j` |
| Google ADK for Java | `com.google.adk`, `LlmAgent`, `FunctionTool`, `Runner`, session services |
| Koog (JetBrains, JVM) | `ai.koog`, `AIAgent`, `ToolRegistry`, strategies |
| MCP Java SDK | `io.modelcontextprotocol`, `McpServer`, `McpSyncClient`, `McpAsyncClient` |
| Anthropic SDK | `com.anthropic`, `AnthropicOkHttpClient` |
| OpenAI SDK | `com.openai`, `OpenAIOkHttpClient` (also used for OpenAI-compatible endpoints: check base URLs) |
| Google Gen AI SDK | `com.google.genai` |
| Vertex AI SDK (legacy) | `com.google.cloud.vertexai`; its generative AI module was deprecated in favor of `com.google.genai`: report continued use as a lifecycle item (B17) after checking its current status |
| AWS Bedrock | `software.amazon.awssdk.services.bedrockruntime` |
| Azure | `com.azure.ai.openai`, `com.azure.ai.inference` |
| Local models | Ollama (`ollama` starters or `:11434`), `com.github.tjake.jlama`, `ai.djl`, `ai.onnxruntime` |
| Raw HTTP | `RestClient`, `WebClient`, `java.net.http.HttpClient`, OkHttp, OpenFeign, `@HttpExchange` together with provider hosts or paths: `/v1/chat/completions`, `/v1/responses`, `/v1/messages`, `generativelanguage.googleapis.com`, `bedrock-runtime`, `openai.azure.com` |
| Client-side leaks | Provider hosts or key-like strings in frontend sources, `environment*.ts`, `.env*`, mobile resources |

Also check:

- Framework-created storage: `initialize-schema`, `schema.sql` inside starters, auto-DDL options of vector stores and chat memory repositories.
- Framework logging switches: `log-prompt`, `log-completion`, `logRequests`, `logResponses`, debug log levels on client packages.
- Retry configuration at every layer: framework properties, SDK builders, Spring Retry, Resilience4j, hand-written loops.

## Where controls usually live

Use this to find existing mitigations before reporting a gap (operating rule 5). Confirm the exact API for the detected version (rule 7).

- Spring AI: advisors (memory, RAG, safeguards, custom) and their order, default options on `ChatClient` builders, tool context, observation handlers and their properties, retry properties, HTTP client customizers, vector store filter expressions.
- LangChain4j: AI service annotations, input and output guardrails and their configuration, chat memory providers and stores, content retrievers with metadata filters, tool executors, `@ToolMemoryId`, MCP tool provider filters, model builders (timeouts, retries, max tokens), listeners.
- Agent frameworks (ADK, Koog, Embabel, LangGraph4j, Semantic Kernel): callbacks or interceptors around model and tool calls, session and state services, tool registries, planner and executor boundaries.
- SDKs and raw HTTP: client builders (timeouts, max retries), interceptors, wrappers written by the project.
- Cross-cutting: servlet filters, aspects, `TaskDecorator` and context propagation for async work, gateways or proxies in front of the provider, and a common exit point for user-facing messages.
