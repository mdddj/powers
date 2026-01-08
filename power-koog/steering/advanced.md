# Koog - Advanced Topics

## A2A Protocol (Agent-to-Agent)

### A2A Server Dependencies

```kotlin
dependencies {
    implementation("ai.koog:agents-features-a2a-server:$koogVersion")
    implementation("ai.koog:a2a-transport-server-jsonrpc-http:$koogVersion")
    implementation("io.ktor:ktor-server-netty:$ktorVersion")
}
```

### A2A Client Dependencies

```kotlin
dependencies {
    implementation("ai.koog:agents-features-a2a-client:$koogVersion")
    implementation("ai.koog:a2a-transport-client-jsonrpc-http:$koogVersion")
    implementation("io.ktor:ktor-client-cio:$ktorVersion")
}
```

### A2A Server Feature

```kotlin
val agent = AIAgent(executor, toolRegistry, strategy) {
    install(A2AAgentServer) {
        this.context = context
        this.eventProcessor = eventProcessor
    }
}
```

## Agent Persistence

Save and restore agent state at checkpoints.

```kotlin
val agent = AIAgent(executor, model, systemPrompt) {
    install(Persistence) {
        provider = LocalFilePersistenceProvider(Path("./checkpoints"))
    }
}

// Save checkpoint
agent.saveCheckpoint("checkpoint-1")

// Restore from checkpoint
agent.restoreCheckpoint("checkpoint-1")

// Roll back tool side-effects
install(Persistence) {
    rollbackToolRegistry = RollbackToolRegistry {
        // Define rollback actions
    }
}
```

## History Compression

Reduce token usage in long conversations.

```kotlin
val strategy = strategy<String, String>("with-compression") {
    val nodeTrimHistory by node<Unit, Unit> {
        llm.writeSession {
            rewritePrompt {
                // Keep only system prompt and latest message
                val systemMsg = prompt.messages.firstOrNull { it is Message.System }
                val lastMsg = prompt.messages.lastOrNull()
                systemMsg?.let { message(it) }
                lastMsg?.let { message(it) }
            }
        }
    }
}
```

### Built-in Compression

```kotlin
llm.writeSession {
    compressHistory(CompressionStrategy.FactRetrieval)
}
```

## Subgraphs

Create reusable workflow components.

```kotlin
val mySubgraph = subgraph<String, String>("my-subgraph") {
    val nodeProcess by node<String, String> { input ->
        input.uppercase()
    }
    
    edge(nodeStart forwardTo nodeProcess)
    edge(nodeProcess forwardTo nodeFinish)
}

// Use in main strategy
val strategy = strategy<String, String>("main") {
    edge(nodeStart forwardTo mySubgraph)
    edge(mySubgraph forwardTo nodeFinish)
}
```

### subgraphWithTask

```kotlin
val taskSubgraph = subgraphWithTask<Input, Output>("task") {
    tools {
        tool(MyTool())
    }
    // Automatically deduces final step
}
```

## Data Transfer Between Nodes

Use AIAgentStorage for type-safe data passing.

```kotlin
val KEY_USER_ID = AIAgentStorageKey<String>("user_id")

val nodeSetUserId by node<String, Unit> { userId ->
    storage[KEY_USER_ID] = userId
}

val nodeGetUserId by node<Unit, String> {
    storage[KEY_USER_ID] ?: error("User ID not set")
}
```

## Content Moderation

```kotlin
val moderationResult = moderationClient.moderate(content)
if (moderationResult.flagged) {
    println("Flagged: ${moderationResult.categories}")
}
```

## Embeddings

```kotlin
val embedding = embeddingClient.embed("Hello world")
val similarity = embedding1.cosineSimilarity(embedding2)
```

## RAG (Ranked Document Storage)

```kotlin
val storage = InMemoryVectorStorage()
storage.store(document, embedding)

val results = storage.search(queryEmbedding, topK = 5)
```

## AIAgentService

Manage multiple running agents.

```kotlin
val service = AIAgentService(agentFactory)

// Create agent tool from service
val agentTool = service.createAgentTool("sub-agent")
```

## Retry Component

```kotlin
val retrySubgraph = subgraphWithRetry<Input, Output>("retry") {
    maxRetries = 3
    onFeedback { error ->
        // Adjust based on error
    }
}
```

## LLM as a Judge

```kotlin
val judge = LLMAsJudge(executor, model)
val score = judge.evaluate(response, criteria)
```

## Reasoning Messages

Support for reasoning/thinking in responses (v0.5.3+):

```kotlin
// Reasoning messages are automatically handled in strategy
// Access via Message.Reasoning type
```

## Testing

Test agent pipelines, subgraphs, and tool interactions.

### Dependencies

```kotlin
testImplementation("ai.koog:agents-test:0.5.4")
```

### Mock LLM Responses

```kotlin
val mockLLMApi = getMockExecutor(toolRegistry) {
    mockLLMAnswer("Hello!") onRequestContains "Hello"
    mockLLMAnswer("I don't know.").asDefaultResponse
}
```

### Mock Tool Calls

```kotlin
mockLLMToolCall(MyTool, MyTool.Args("value")) onRequestEquals "Do task"
mockTool(MyTool) alwaysReturns "Result"
mockTool(MyTool) returns "Specific" onArguments MyTool.Args("specific")
```

### Enable Testing Mode

```kotlin
val agent = AIAgent(mockLLMApi, toolRegistry, model) {
    withTesting()
}
```

## Tracing

Debug and monitor agent execution.

```kotlin
val agent = AIAgent(executor, model, systemPrompt) {
    install(Tracing) {
        addMessageProcessor(TraceFeatureMessageLogWriter(logger))
        addMessageProcessor(TraceFeatureMessageFileWriter(outputPath))
    }
}
```

### Message Filtering

```kotlin
install(Tracing) {
    messageFilter = { event ->
        event is ToolCallEvent || event is LLMCallEvent
    }
}
```

## Troubleshooting

### Common Issues

1. **LLMClientException**: Check API key and network
2. **No qualifying bean**: Verify Spring configuration
3. **Tool not found**: Ensure tool is registered
4. **Timeout**: Increase timeout or check network
5. **History too large**: Use history compression

### Best Practices

1. Use environment variables for secrets
2. Implement fallback logic for multiple providers
3. Wrap executor calls in try-catch
4. Use mocks in tests
5. Monitor with OpenTelemetry
6. Compress history for long conversations
