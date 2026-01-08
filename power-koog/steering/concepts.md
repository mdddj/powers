# Koog - Core Concepts

## Agent Types

### Basic Agents
Minimal configuration for simple tasks:

```kotlin
val agent = AIAgent(
    promptExecutor = simpleOpenAIExecutor(apiKey),
    systemPrompt = "You are a helpful assistant.",
    llmModel = OpenAIModels.Chat.GPT4o
)
```

### Functional Agents
Lightweight, non-graph agents with simple loop control:

```kotlin
val agent = functionalAgent(executor, model) {
    // Custom control flow
}
```

### Complex Workflow Agents
Custom strategies with graph-based workflows:

```kotlin
val agent = AIAgent(
    promptExecutor = executor,
    llmModel = model,
    strategy = myCustomStrategy,
    toolRegistry = ToolRegistry { /* tools */ }
)
```

## Node Architecture

Nodes are building blocks of agent workflows.

### Basic Node

```kotlin
val myNode by node<String, Int>("string_length") { input ->
    input.length
}
```

### Node with Arguments

```kotlin
fun AIAgentSubgraphBuilderBase<*, *>.myNode(
    arg1: String,
    arg2: Int
): AIAgentNodeDelegate<Input, Output> = node("my_node") { input ->
    // Use arg1, arg2
    input
}
```

### Node Transformations

```kotlin
val transformedNode = myNode.transform { result ->
    result.uppercase()
}
```

## Strategies

Strategies define execution flow using graphs.

### Pre-defined Nodes

- `nodeCallLLM`: Process input, generate responses/tool calls
- `nodeExecuteTool`: Execute tools
- `nodeTrimHistory`: Optimize memory
- `nodeSendToolResult`: Send tool results to LLM
- `nodeLLMCompressHistory`: Compress history

### Custom Strategy

```kotlin
val myStrategy = strategy<String, String>("my-strategy") {
    val nodeProcess by node<String, String> { input ->
        input.uppercase()
    }
    
    edge(nodeStart forwardTo nodeProcess)
    edge(nodeProcess forwardTo nodeFinish)
}
```

## LLM Sessions

### Write Session (modifies history)

```kotlin
llm.writeSession {
    appendPrompt {
        user("Hello!")
    }
    val response = requestLLM()
}
```

### Read Session (read-only)

```kotlin
llm.readSession {
    val currentPrompt = prompt
    val availableTools = tools
}
```

### Request Methods

```kotlin
llm.writeSession {
    requestLLM()                    // Basic request
    requestLLMWithoutTools()        // No tools
    requestLLMMultiple()            // Multiple responses
    requestLLMStreaming()           // Streaming
    requestLLMStructured<T>()       // Structured output
}
```

## Tools

### Annotation-based

```kotlin
@Tool("calculator")
fun calculate(
    @ToolParam("expression") expr: String
): String = eval(expr)
```

### Class-based

```kotlin
class MyTool : SimpleTool<MyArgs, MyResult>() {
    override val descriptor = ToolDescriptor(
        name = "my_tool",
        description = "Does something"
    )
    
    override suspend fun doExecute(args: MyArgs): MyResult {
        // Implementation
    }
}
```

## Agent Features

Install features to extend agent capabilities:

```kotlin
val agent = AIAgent(executor, model, systemPrompt) {
    install(EventHandler) { /* config */ }
    install(OpenTelemetry) { /* config */ }
    install(AgentMemory) { /* config */ }
    install(Persistence) { /* config */ }
}
```

### Available Features

- **EventHandler**: Monitor and respond to events
- **Tracing**: Comprehensive execution tracing
- **AgentMemory**: Cross-conversation memory
- **OpenTelemetry**: Observability (W&B Weave, Langfuse)
- **Persistence**: Save/restore agent state
- **Tokenizer**: Token counting

## Best Practices

1. Keep nodes focused on single operations
2. Use descriptive node names
3. Handle errors gracefully
4. Use environment variables for API keys
5. Compress history for long conversations
6. Use appropriate session types (read vs write)
