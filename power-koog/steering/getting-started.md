# Koog - Getting Started

## Prerequisites

- JDK 17 or higher
- Kotlin/JVM project with Gradle or Maven
- kotlinx-coroutines 1.10.2 and kotlinx-serialization 1.8.1
- API key for your preferred LLM provider (not required for Ollama)

## Installation

### Gradle (Kotlin DSL)

```kotlin
dependencies {
    implementation("ai.koog:koog-agents:0.5.4")
}

repositories {
    mavenCentral()
}
```

### Gradle (Groovy)

```groovy
dependencies {
    implementation 'ai.koog:koog-agents:0.5.4'
}

repositories {
    mavenCentral()
}
```

### Maven

```xml
<dependency>
    <groupId>ai.koog</groupId>
    <artifactId>koog-agents-jvm</artifactId>
    <version>0.5.4</version>
</dependency>
```

## Set API Keys

```bash
# OpenAI
export OPENAI_API_KEY=your_key

# Anthropic
export ANTHROPIC_API_KEY=your_key

# Google
export GOOGLE_API_KEY=your_key

# DeepSeek
export DEEPSEEK_API_KEY=your_key

# OpenRouter
export OPENROUTER_API_KEY=your_key

# AWS Bedrock
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret
```

## Create Your First Agent

### OpenAI

```kotlin
import ai.koog.agents.core.agent.AIAgent
import ai.koog.prompt.executor.clients.openai.OpenAIModels
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    val agent = AIAgent(
        promptExecutor = simpleOpenAIExecutor(System.getenv("OPENAI_API_KEY")),
        systemPrompt = "You are a helpful assistant.",
        llmModel = OpenAIModels.Chat.GPT4o
    )
    
    println(agent.run("What is Kotlin?"))
}
```

### Anthropic

```kotlin
import ai.koog.prompt.executor.clients.anthropic.AnthropicModels
import ai.koog.prompt.executor.llms.all.simpleAnthropicExecutor

val agent = AIAgent(
    promptExecutor = simpleAnthropicExecutor(System.getenv("ANTHROPIC_API_KEY")),
    systemPrompt = "You are a helpful assistant.",
    llmModel = AnthropicModels.Claude.Opus4_1
)
```

### Google Gemini

```kotlin
import ai.koog.prompt.executor.clients.google.GoogleModels
import ai.koog.prompt.executor.llms.all.simpleGoogleExecutor

val agent = AIAgent(
    promptExecutor = simpleGoogleExecutor(System.getenv("GOOGLE_API_KEY")),
    systemPrompt = "You are a helpful assistant.",
    llmModel = GoogleModels.Gemini2_5Pro
)
```

### Ollama (Local)

```kotlin
import ai.koog.prompt.executor.llms.all.simpleOllamaExecutor

val agent = AIAgent(
    promptExecutor = simpleOllamaExecutor(),
    systemPrompt = "You are a helpful assistant.",
    llmModel = "llama3.2"
)
```

## Supported Targets

- JVM (JDK 17+)
- JavaScript
- WebAssembly (WasmJS)
- iOS
- Android

## What's Next

- [Key Features](https://docs.koog.ai/key-features/)
- [Basic Agents](https://docs.koog.ai/basic-agents/)
- [Tools Overview](https://docs.koog.ai/tools-overview/)
- [Complex Workflow Agents](https://docs.koog.ai/complex-workflow-agents/)
