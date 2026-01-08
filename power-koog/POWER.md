---
name: "koog"
displayName: "Koog AI Agent Framework"
description: "Build and run AI agents entirely in idiomatic Kotlin using the Koog framework by JetBrains. Supports multiple LLM providers, tools, strategies, MCP integration, and more."
keywords: ["koog", "kotlin", "ai agent", "llm", "agent framework", "mcp", "tools", "openai", "anthropic", "gemini", "ollama", "jetbrains", "spring", "ktor", "android", "jvm", "multi-agent"]
---

# Koog AI Agent Framework

Koog is the official Kotlin framework by JetBrains for building predictable, fault-tolerant and enterprise-ready AI agents across all platforms – from backend services to Android and iOS, JVM, and even in-browser environments.

**Latest Version:** 0.5.4 (December 2025)
**Repository:** [JetBrains/koog](https://github.com/JetBrains/koog)
**Documentation:** [docs.koog.ai](https://docs.koog.ai)
**API Reference:** [api.koog.ai](https://api.koog.ai)

## When to Use This Power

Activate this power when:
- Building AI agents in Kotlin
- Working with Koog framework APIs
- Integrating with LLM providers (OpenAI, Anthropic, Google, Ollama, Bedrock, etc.)
- Implementing agent strategies, tools, or workflows
- Working with MCP (Model Context Protocol) or A2A (Agent-to-Agent) protocol
- Using Koog with Spring Boot or Ktor
- Debugging or tracing Koog agents

## Quick Start

### Installation (Gradle Kotlin DSL)

```kotlin
dependencies {
    implementation("ai.koog:koog-agents:0.5.4")
}

repositories {
    mavenCentral()
}
```

### Create a Simple Agent

```kotlin
import ai.koog.agents.core.agent.AIAgent
import ai.koog.prompt.executor.clients.openai.OpenAIModels
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    val apiKey = System.getenv("OPENAI_API_KEY")
    
    val agent = AIAgent(
        promptExecutor = simpleOpenAIExecutor(apiKey),
        systemPrompt = "You are a helpful assistant.",
        llmModel = OpenAIModels.Chat.GPT4o
    )
    
    val result = agent.run("Hello! How can you help me?")
    println(result)
}
```

## Key Features

- **Multiplatform**: JVM, JS, WasmJS, Android, iOS targets
- **Enterprise-ready**: Spring Boot and Ktor integrations
- **Fault-tolerant**: Built-in retries and agent persistence
- **Intelligent history compression**: Optimize token usage
- **OpenTelemetry support**: W&B Weave, Langfuse integration
- **MCP integration**: Use Model Context Protocol tools
- **A2A Protocol**: Agent-to-Agent communication
- **Streaming API**: Real-time response processing
- **Graph workflows**: Flexible agent behavior design

## Supported LLM Providers

| Provider | Environment Variable | Models |
|----------|---------------------|--------|
| OpenAI | `OPENAI_API_KEY` | GPT-4o, GPT-5, GPT-5.1, GPT-5 Pro |
| Anthropic | `ANTHROPIC_API_KEY` | Claude Opus 4.1, Claude Opus 4.5, Sonnet 4.5 |
| Google | `GOOGLE_API_KEY` | Gemini 2.5 Pro, Gemini 2.5 Flash |
| DeepSeek | `DEEPSEEK_API_KEY` | deepseek-chat |
| OpenRouter | `OPENROUTER_API_KEY` | Various models |
| AWS Bedrock | AWS credentials | Nova, Claude, etc. |
| Ollama | None (local) | llama3.2, etc. |
| MistralAI | `MISTRAL_API_KEY` | Mistral models |
| DashScope | `DASHSCOPE_API_KEY` | Qwen models |

## When to Load Steering Files

- Setting up a new Koog project → `getting-started.md`
- Understanding core concepts (nodes, strategies, sessions, tools) → `concepts.md`
- Working with APIs (Prompt API, Streaming API, Spring Boot, Ktor) → `api.md`
- Looking for code examples → `examples.md`
- Advanced topics (MCP, A2A, OpenTelemetry, persistence, RAG) → `advanced.md`
- Checking version history and changes → `changelog.md`

## Documentation Coverage

This power covers the following topics from official Koog documentation:

**Agent Types**: Basic agents, Functional agents, Complex workflow agents

**Core Functionality**: Prompts, Tools (built-in, annotation-based, class-based), Strategies, Events

**Advanced Usage**: History compression, Agent persistence, Structured output, Streaming API, Embeddings, RAG

**Integrations**: MCP, Spring Boot, Ktor, OpenTelemetry (Langfuse, Weave), A2A Protocol

**Features**: EventHandler, Tracing, AgentMemory, Persistence, Tokenizer

For detailed documentation, visit [docs.koog.ai](https://docs.koog.ai)

## Recent Changes (v0.5.4)

- Better error reporting with `LLMClientException`
- Support for OpenAI GPT-5.1, GPT-5 Pro
- Support for Anthropic Claude Opus 4.5
- Bedrock support in Ktor
- Improved `ReadFileTool`

## Resources

- [Official Documentation](https://docs.koog.ai)
- [API Reference](https://api.koog.ai)
- [GitHub Repository](https://github.com/JetBrains/koog)
- [Slack Channel](https://docs.koog.ai/koog-slack-channel/)
- [Issue Tracker](https://youtrack.jetbrains.com/issues/KG)
