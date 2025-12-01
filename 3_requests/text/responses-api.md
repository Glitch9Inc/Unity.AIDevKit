---
icon: bolt-lightning
---

# Responses API

The Responses API provides the most advanced text generation capabilities with support for complex reasoning, tool calling, and multi-modal inputs.

## Basic Usage

```csharp
string response = await "Write a technical specification"
    .GENResponse()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

## Input Types

### 1. String Input

```csharp
string response = await "Explain quantum mechanics"
    .GENResponse()
    .ExecuteAsync();
```

### 2. ConversationItem Input

```csharp
var userMessage = new UserMessage("Hello, AI!");
string response = await userMessage
    .GENResponse()
    .ExecuteAsync();
```

### 3. Prompt Input

```csharp
var prompt = new Prompt("Explain {topic} in simple terms");
string response = await prompt
    .GENResponse()
    .ExecuteAsync();
```

## Key Features

### 1. Advanced Reasoning

The Responses API excels at complex reasoning tasks:

```csharp
string analysis = await @"
Analyze the following code and suggest improvements:
public class Player {
    public int health;
    public void TakeDamage(int amount) {
        health -= amount;
    }
}"
    .GENResponse()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

### 2. Tool Calling Support

Built-in support for function calling:

```csharp
var response = await "What's the weather in Tokyo?"
    .GENResponse()
    .SetTools(weatherTools)
    .ExecuteAsync();

// Response may contain tool calls that need to be executed
```

### 3. Multi-Modal Input

Support for text, images, and other content types:

```csharp
var message = new UserMessage();
message.Content.AddText("What's in this image?");
message.Content.AddImage(texture);

string response = await message
    .GENResponse()
    .ExecuteAsync();
```

## Configuration

### Model Selection

```csharp
string response = await "Complex task"
    .GENResponse()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

### Temperature & Creativity

```csharp
string creative = await "Write a story"
    .GENResponse()
    .SetTemperature(1.5f)
    .ExecuteAsync();

string factual = await "What is 2+2?"
    .GENResponse()
    .SetTemperature(0.0f)
    .ExecuteAsync();
```

### Max Tokens

```csharp
string summary = await "Summarize this long text..."
    .GENResponse()
    .SetMaxTokens(200)
    .ExecuteAsync();
```

## Streaming

Get real-time responses as they're generated:

```csharp
await "Tell me a long story"
    .GENResponse()
    .StreamAsync(
        onToken: token => Debug.Log(token),
        onComplete: response => Debug.Log("Complete!")
    );
```

## Provider Support

| Provider | Support | Models |
|----------|---------|--------|
| OpenAI | ✅ Full | GPT-4o, GPT-4 |
| Anthropic | ✅ Full | Claude 3.5 Sonnet |
| Google Gemini | ⚠️ Partial | Gemini 1.5 Pro |
| OpenRouter | ✅ Full | Various |

**Note:** Not all providers support all Responses API features. Check provider documentation for limitations.

## Differences from Chat Completions

| Feature | Chat Completions | Responses API |
|---------|-----------------|---------------|
| Tool Calling | Basic | Advanced |
| Multi-turn | Manual | Automatic |
| Context Window | Standard | Extended |
| Reasoning | Standard | Enhanced |
| Complexity | Simple | Advanced |

## When to Use

### ✅ Use Responses API for

- Complex reasoning tasks
- Multi-step problem solving
- Tool calling workflows
- Long-form content generation
- Advanced features

### ❌ Use Chat Completions for

- Simple Q&A
- Basic chatbots
- When maximum provider compatibility is needed
- Cost-sensitive applications (Responses API may be more expensive)

## Examples

### Example 1: Complex Analysis

```csharp
async UniTask<string> AnalyzeCode(string code)
{
    return await $@"
Analyze this code for:
1. Performance issues
2. Security vulnerabilities
3. Best practices violations

Code:
{code}
"
        .GENResponse()
        .SetModel(OpenAIModel.GPT4o)
        .SetTemperature(0.3f)
        .ExecuteAsync();
}
```

### Example 2: Multi-Step Reasoning

```csharp
async UniTask<string> SolveProblem(string problem)
{
    return await $@"
Solve this step by step:
{problem}

Show your reasoning at each step.
"
        .GENResponse()
        .SetModel(OpenAIModel.GPT4o)
        .ExecuteAsync();
}
```

### Example 3: With Context

```csharp
var conversation = new ConversationItem[]
{
    new SystemMessage("You are a Unity expert"),
    new UserMessage("How do I optimize my game?"),
    new AssistantMessage("Here are some tips..."),
    new UserMessage("Tell me more about object pooling")
};

string response = await conversation.Last()
    .GENResponse()
    .ExecuteAsync();
```

## Best Practices

### ✅ Do

- Use for complex, multi-step tasks
- Leverage tool calling when available
- Set appropriate max tokens for cost control
- Use streaming for long responses

### ❌ Don't

- Use for simple tasks (wasteful)
- Ignore provider limitations
- Forget error handling
- Assume all providers support all features

## Next Steps

- [Code Generation](code-generation.md) - Specialized code generation
- [Structured Output](structured-output.md) - Type-safe JSON responses
- [Chat Completions](chat-completions.md) - Simpler alternative
