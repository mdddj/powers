# Koog - Examples

## Basic Agent

```kotlin
fun main() = runBlocking {
    val agent = AIAgent(
        promptExecutor = simpleOpenAIExecutor(System.getenv("OPENAI_API_KEY")),
        systemPrompt = "You are a helpful assistant.",
        llmModel = OpenAIModels.Chat.GPT4o
    )
    
    println(agent.run("Tell me a joke"))
}
```

## Agent with Tools

```kotlin
@Tool("calculator")
fun calculate(@ToolParam("expression") expr: String): String {
    // Evaluate expression
    return result
}

val agent = AIAgent(
    promptExecutor = executor,
    llmModel = model,
    systemPrompt = "You can use the calculator tool.",
    toolRegistry = ToolRegistry {
        tool(::calculate)
    }
)
```

## MCP - Playwright Browser Automation

```kotlin
val process = ProcessBuilder(
    "npx", "@playwright/mcp@latest", "--port", "8931"
).start()

val toolRegistry = McpToolRegistryProvider.fromTransport(
    transport = McpToolRegistryProvider.defaultSseTransport("http://localhost:8931")
)

val agent = AIAgent(
    promptExecutor = simpleOpenAIExecutor(apiKey),
    llmModel = OpenAIModels.Chat.GPT4o,
    toolRegistry = toolRegistry
)

agent.run("Open browser, navigate to jetbrains.com, click AI in toolbar")
```

## MCP - Google Maps

```kotlin
val process = ProcessBuilder(
    "docker", "run", "-i",
    "-e", "GOOGLE_MAPS_API_KEY=$googleMapsApiKey",
    "mcp/google-maps"
).start()

val toolRegistry = McpToolRegistryProvider.fromTransport(
    transport = McpToolRegistryProvider.defaultStdioTransport(process)
)

val agent = AIAgent(
    promptExecutor = executor,
    llmModel = model,
    toolRegistry = toolRegistry
)

agent.run("Find the elevation of JetBrains office in Munich")
```

## OpenTelemetry - Weave

```kotlin
val agent = AIAgent(
    promptExecutor = simpleOpenAIExecutor(apiKey),
    llmModel = OpenAIModels.Reasoning.GPT4oMini,
    systemPrompt = "You are a code assistant."
) {
    install(OpenTelemetry) {
        addWeaveExporter(
            weaveEntity = System.getenv("WEAVE_ENTITY"),
            weaveProjectName = "koog-tracing"
        )
    }
}
```

## OpenTelemetry - Langfuse

```kotlin
val agent = AIAgent(executor, model, systemPrompt) {
    install(OpenTelemetry) {
        addLangfuseExporter(
            publicKey = System.getenv("LANGFUSE_PUBLIC_KEY"),
            secretKey = System.getenv("LANGFUSE_SECRET_KEY")
        )
    }
}
```

## OpenTelemetry - Jaeger

```kotlin
val agent = AIAgent(executor, model, systemPrompt) {
    install(OpenTelemetry) {
        addSpanExporter(LoggingSpanExporter.create())
        addSpanExporter(
            OtlpGrpcSpanExporter.builder()
                .setEndpoint("http://localhost:4317")
                .build()
        )
    }
}
```

## Agent Persistence

```kotlin
val agent = AIAgent(executor, model, systemPrompt) {
    install(Persistence) {
        // Configuration
    }
}

// Save state
val snapshot = agent.saveSnapshot()

// Restore state
agent.restoreSnapshot(snapshot)
```

## Multimodal - Image Attachments

```kotlin
val prompt = prompt("instagram-post") {
    system("You write creative Instagram descriptions.")
    user {
        markdown {
            +"Create a post for these photos:"
            h2("Requirements")
            bulleted {
                item("Funny and creative")
                item("Include relevant hashtags")
            }
        }
        attachments {
            image(Path("images/photo1.png"))
            image(Path("images/photo2.png"))
        }
    }
}

val result = executor.execute(prompt, OpenAIModels.Chat.GPT4o)
```

## Parallel Node Execution

```kotlin
val strategy = strategy<String, String>("parallel") {
    val nodeA by node<String, String> { /* task A */ }
    val nodeB by node<String, String> { /* task B */ }
    val nodeMerge by node<List<String>, String> { results ->
        results.joinToString("\n")
    }
    
    edge(nodeStart forwardTo parallel(nodeA, nodeB))
    edge(parallel(nodeA, nodeB) forwardTo nodeMerge)
    edge(nodeMerge forwardTo nodeFinish)
}
```

## Available Examples

| Example | Description |
|---------|-------------|
| Attachments | Multimodal prompts with images |
| Banking | AI banking assistant with routing |
| BedrockAgent | AWS Bedrock integration |
| Calculator | Simple calculator agent |
| Chess | AI chess player with tools |
| GoogleMapsMcp | Location and elevation queries |
| PlaywrightMcp | Browser automation |
| UnityMcp | Unity game engine integration |
| OpenTelemetry | Tracing with Jaeger |
| Weave | W&B Weave tracing |
| Langfuse | Langfuse observability |
| VacuumAgent | Simple reactive agent |
| Guesser | Number guessing game |

See [examples on GitHub](https://github.com/JetBrains/koog/tree/develop/examples)
