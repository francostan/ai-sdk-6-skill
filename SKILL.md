---
name: ai-sdk-6-skill
description: Comprehensive expert guide for building, migrating, and debugging server-side AI applications with Vercel AI SDK 6. Covers production-grade agents (ToolLoopAgent), human-in-the-loop safety, MCP integrations, RAG with reranking, structured outputs, and deep observability with DevTools.
---

# ai-sdk-6-skill

Instructions for the agent to follow when this skill is activated.

## When to use

- **Architecting Agents**: When designing reusable, multi-step agents that operate across different interfaces (API, Chat UI, Background Jobs) using `ToolLoopAgent`.
- **Enforcing Safety**: When implementing "Human-in-the-Loop" workflows for sensitive tool executions (e.g., database writes, payment processing) using `needsApproval`.
- **Connecting External Data**: When integrating Model Context Protocol (MCP) servers, performing advanced RAG with `rerank`, or using provider-specific tools (Anthropic Memory, Google Grounding).
- **Debugging & Optimization**: When troubleshooting complex agent traces via DevTools, optimizing token usage with `toModelOutput`, or analyzing raw provider finish reasons.
- **Migration**: When upgrading legacy AI SDK 5 codebases to v6 standards.

## Instructions

1.  **Standardize on `ToolLoopAgent`**
    - Adopt `ToolLoopAgent` as the default primitive for multi-step workflows (automatically handles the "call model → run tools → append results" loop, defaulting to 20 steps).
    - Define agents in dedicated modules (e.g., `agents/support-agent.ts`) to centralize model config, system instructions, and tool definitions.
    - Export strict types for UI integration: `export type MyAgentUIMessage = InferAgentUIMessage<typeof myAgent>;`.
    - Use `callOptionsSchema` to type-safe dynamic request context (User ID, subscription tier) and inject it via `prepareCall`.

2.  **Implement Robust Tooling & Safety**
    - **Validation**: Enable `strict: true` for tools where the provider supports it to guarantee schema adherence.
    - **Clarity**: Provide `inputExamples` (vital for Anthropic models) to disambiguate complex schema requirements.
    - **Safety Gates**: Mark destructive tools with `needsApproval: true` (or a dynamic async predicate). In your UI/API, handle the `approval-requested` state before executing the tool.
    - **Context Hygiene**: Use `toModelOutput` to return rich data (complete JSON objects) to your app application but send minimal summaries to the LLM context, saving tokens and reducing noise.

3.  **Deploy Model Context Protocol (MCP)**
    - Use the stable `@ai-sdk/mcp` package to connect to external MCP servers.
    - Configure `OAuthClientProvider` for secure, authenticated connections to third-party services.
    - Handle `elicitation` requests where the server interrupts execution to ask the user for clarification or missing details.

4.  **Enhance Retrieval (RAG)**
    - Move beyond simple vector search. Implement a two-stage retrieval: fetch a broad set of candidates, then use `rerank` (supporting Cohere, Bedrock, etc.) to reorder them by relevance.
    - Feed only the top N reranked results to the model context to reduce hallucinations and latency.

5.  **Utilize Provider-Native Tools**
    - Leverage specialized tools instead of building custom equivalents where possible:
      - **Anthropic**: `computer` (beta), `memory` (key-value store), `code_execution` (sandboxed analysis).
      - **OpenAI**: `file_search`, `code_interpreter`.
      - **Google**: `google_maps` (grounding), `vertex_rag_store`.
      - **xAI**: `web_search` (with image understanding), `x_search`.

6.  **Structured Output & Multi-Modal Flows**
    - Combine `ToolLoopAgent` with `Output.object` or `Output.json` when the final result must be machine-readable (e.g., extracting data after a research session).
    - For image editing, use `generateImage` with reference images passed via the `prompt.images` array (URL/Buffer/Base64) for style transfer or inpainting tasks.

7.  **Observability & Debugging**
    - **DevTools**: Wrap your model with `devToolsMiddleware` and run `npx @ai-sdk/devtools` (port 4983) to inspect the full trace: inputs, tool calls, raw provider payloads, and exact token costs.
    - **Usage Analytics**: log `usage.inputTokenDetails` (cache hits vs. misses) and `usage.outputTokenDetails` to optimize long-running agent costs.
    - **Raw Signals**: Check `rawFinishReason` to handle provider-specific stop conditions that don't map cleanly to the standard SDK enum.

8.  **Migration Path**
    - Run `npx @ai-sdk/codemod v6` to automate the upgrade.
    - Manually verify any custom `middleware` or complex `streamUI` implementations, as these APIs have evolved to support the new agentic patterns.
    - **Custom Middleware**: Ensure your custom middleware is compatible with the new middleware system, which now supports asynchronous operations and more flexible error handling.
