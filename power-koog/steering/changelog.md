# Koog - Changelog

## 0.5.4 (December 2025)

### Improvements
- Better error reporting: LLM clients now throw `LLMClientException` instead of `IllegalStateException`
- Support for **OpenAI GPT-5.1** and **GPT-5 Pro**
- Support for **Anthropic Claude Opus 4.5**
- Bedrock support in Ktor
- Improved `ReadFileTool`

### Bug Fixes
- Fix `McpTool` serialization
- Fix file tools API
- Fix reasoning message handling in strategy
- Fix timeout in `JvmShellCommandExecutor`

## 0.5.3 (November 2025)

### New Features
- **Reasoning messages support**
- Get models list request for OpenAI-based clients

### Improvements
- Subgraph execution events in pipeline and features
- `systemPrompt` and `temperature` now optional

## 0.5.2 (October 2025)

### New Features
- `subtask` extension for non-graph agents
- **MistralAI LLM Client**

### Improvements
- Unified content parts list in messages
- JVM target set to 11 for older JVM support
- Multi-responses in `subgraphWithTask` API

## 0.5.1 (October 2025)

### Improvements
- **GPT-5 Codex** model support
- **DashScope (Qwen)** LLM client
- Filters in PersistenceProvider
- Additional Bedrock auth options

## 0.5.0 (October 2025)

### Major Features
- **Full A2A Protocol Support** with Kotlin SDK
- **Non-Graph API for Strategies**
- **Agent Persistence and Checkpointing** with rollback
- **Tool API Improvements**: Auto-generated `ToolDescriptor`
- **AIAgentService** for managing multiple agents
- **LLM as a Judge** component
- **Tool Calling loop with Structured Output**

## 0.4.2 (September 2025)

### Improvements
- KMP targets for agents-mcp
- LLM client retry in Spring Boot
- Claude Opus 4.1 support
- Gemini 2.5 Flash Lite support
- Java-compatible Prompt Executor
- Postgres persistence provider

## 0.4.0 (August 2025)

### Major Features
- **Langfuse Integration**
- **W&B Weave Integration**
- **Ktor Integration** via plugin
- **iOS Target Support**
- **Upgraded Structured Output**
- **GPT5 Support**
- **Retryable LLM Clients**

## 0.3.0 (July 2025)

### Major Features
- **Agent Persistence and Checkpoints**
- **Vector Document Storage** (RAG)
- **OpenTelemetry Support**
- **Content Moderation**
- **Parallel Node Execution**
- **Spring Integration**
- **AWS Bedrock Support**
- **WebAssembly Support**

## 0.2.0 (June 2025)

### Features
- Media types (image/audio/document) support
- Token count and timestamp in responses
- LLM caching capability (Anthropic)
- Groq, Meta, Alibaba configurations

## 0.1.0 (May 2025)

Initial release with:
- Pure Kotlin implementation
- MCP integration
- Embedding capabilities
- Custom tool creation
- History compression
- Streaming API
- Graph workflows
- JVM and JS targets
