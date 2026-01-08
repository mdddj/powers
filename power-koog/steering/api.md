# Koog - API Reference

## Prompt API

### Creating Prompts

```kotlin
val prompt = prompt("my-prompt") {
    system("You are a helpful assistant.")
    user("Hello!")
    assistant("Hi there!")
}
```

### Multimodal Inputs

```kotlin
val prompt = prompt("multimodal") {
    user {
        text("Describe this image:")
        attachments {
            image(Path("photo.png"))
            audio(Path("audio.mp3"))
            file(Path("document.pdf"), mimeType = "application/pdf")
        }
    }
}
```

### Attachment Content Sources

```kotlin
// From URL
AttachmentContent.URL("https://example.com/image.png")

// From bytes
AttachmentContent.Binary.Bytes(byteArray)

// From Base64
AttachmentContent.Binary.Base64(base64String)

// Plain text
AttachmentContent.PlainText(textContent)
```

## Streaming API

### Stream Frame Types

- `StreamFrame.Append(text)`: Incremental text
- `StreamFrame.ToolCall(id, name, content)`: Tool invocation
- `StreamFrame.End(finishReason)`: End marker

### Working with Streams

```kotlin
llm.writeSession {
    val stream = requestLLMStreaming()
    
    stream.collect { frame ->
        when (frame) {
            is StreamFrame.Append -> print(frame.text)
            is StreamFrame.ToolCall -> println("Tool: ${frame.name}")
            is StreamFrame.End -> println("[END]")
        }
    }
}
```

### Text-Only Stream

```kotlin
val fullText = requestLLMStreaming().collectText()
// or
requestLLMStreaming().filterTextOnly().collect { print(it) }
```

## Structured Output

```kotlin
@Serializable
data class BookInfo(val title: String, val author: String, val year: Int)

llm.writeSession {
    val book = requestLLMStructured<BookInfo>()
    println("Title: ${book.title}")
}
```

## Spring Boot Integration

### Dependencies

```kotlin
implementation("ai.koog:koog-spring-boot-starter:0.5.4")
```

### Configuration (application.yml)

```yaml
ai:
  koog:
    openai:
      enabled: true
      api-key: ${OPENAI_API_KEY}
    anthropic:
      enabled: true
      api-key: ${ANTHROPIC_API_KEY}
    ollama:
      enabled: true
      base-url: http://localhost:11434
```

### Inject Executors

```kotlin
@Service
class AIService(
    private val openAIExecutor: SingleLLMPromptExecutor?,
    private val anthropicExecutor: SingleLLMPromptExecutor?
) {
    suspend fun generate(input: String): String {
        val prompt = prompt { user(input) }
        return openAIExecutor?.execute(prompt)?.text
            ?: throw IllegalStateException("No provider")
    }
}
```

## Ktor Integration

### Dependencies

```kotlin
implementation("ai.koog:koog-ktor:0.5.4")
```

### Plugin Installation

```kotlin
fun Application.module() {
    install(Koog) {
        // Configuration
    }
    
    routing {
        post("/chat") {
            val request = call.receive<ChatRequest>()
            val response = koogAgent.run(request.message)
            call.respond(ChatResponse(response))
        }
    }
}
```

## MCP Integration

### STDIO Transport

```kotlin
val process = ProcessBuilder("npx", "@some/mcp-server").start()
val transport = McpToolRegistryProvider.defaultStdioTransport(process)
val toolRegistry = McpToolRegistryProvider.fromTransport(
    transport = transport,
    name = "mcp-client",
    version = "1.0.0"
)

val agent = AIAgent(
    promptExecutor = executor,
    llmModel = model,
    toolRegistry = toolRegistry
)
```

### SSE Transport

```kotlin
val toolRegistry = McpToolRegistryProvider.fromTransport(
    transport = McpToolRegistryProvider.defaultSseTransport("http://localhost:8931")
)
```

## Event Handlers

```kotlin
val agent = AIAgent(executor, model, systemPrompt) {
    install(EventHandler) {
        onToolCallStarting { ctx ->
            println("Calling ${ctx.tool.name}")
        }
        onLLMStreamingFrameReceived { ctx ->
            (ctx.streamFrame as? StreamFrame.Append)?.let {
                print(it.text)
            }
        }
    }
}
```

## LLM Parameters

```kotlin
val agent = AIAgent(
    promptExecutor = executor,
    llmModel = model,
    temperature = 0.7,
    maxTokens = 1000
)
```
