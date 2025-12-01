---
icon: comments
---

# Chat Completions

The Chat Completions API is the most widely supported text generation method across AI providers.

## Basic Usage

### Simple Text Input

```csharp
string response = await "Hello, AI!"
    .GENCompletion()
    .ExecuteAsync();
```

### With Model Selection

```csharp
string response = await "Explain photosynthesis"
    .GENCompletion()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

## Input Types

### 1. String Input

Direct text prompt:

```csharp
string response = await "What is AI?"
    .GENCompletion()
    .ExecuteAsync();
```

### 2. Message Input

Use `Message` object for more control:

```csharp
var message = new UserMessage("Explain quantum computing");
string response = await message
    .GENCompletion()
    .ExecuteAsync();
```

### 3. Prompt Input

Use `Prompt` object for reusable prompts:

```csharp
var prompt = new Prompt("Explain {topic}");
string response = await prompt
    .GENCompletion()
    .SetVariable("topic", "machine learning")
    .ExecuteAsync();
```

## Configuration

### Temperature

Controls randomness (0.0 = deterministic, 2.0 = very creative):

```csharp
// Creative writing
string poem = await "Write a poem about stars"
    .GENCompletion()
    .SetTemperature(1.5f)
    .ExecuteAsync();

// Factual answers
string fact = await "What is 2+2?"
    .GENCompletion()
    .SetTemperature(0.0f)
    .ExecuteAsync();
```

### Max Tokens

Limit response length:

```csharp
string summary = await "Summarize the history of AI"
    .GENCompletion()
    .SetMaxTokens(100) // Short summary
    .ExecuteAsync();
```

### System Message

Set context and behavior:

```csharp
string response = await "How do I start?"
    .GENCompletion()
    .SetSystemMessage("You are a helpful Unity game development tutor")
    .ExecuteAsync();
```

### Top P (Nucleus Sampling)

Alternative to temperature (0.0-1.0):

```csharp
string response = await "Generate a character name"
    .GENCompletion()
    .SetTopP(0.9f)
    .ExecuteAsync();
```

### Frequency Penalty

Reduce word repetition (-2.0 to 2.0):

```csharp
string response = await "List 10 unique animals"
    .GENCompletion()
    .SetFrequencyPenalty(1.0f) // Avoid repetition
    .ExecuteAsync();
```

### Presence Penalty

Encourage topic diversity (-2.0 to 2.0):

```csharp
string response = await "Write about technology"
    .GENCompletion()
    .SetPresencePenalty(1.0f) // Diverse topics
    .ExecuteAsync();
```

## Multi-Turn Conversations

Build conversation history:

```csharp
var conversation = new List<Message>
{
    new SystemMessage("You are a helpful assistant"),
    new UserMessage("Hello!"),
    new AssistantMessage("Hi! How can I help?"),
    new UserMessage("What's the weather?")
};

string response = await new ChatCompletionRequest(conversation)
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

## Streaming

Get real-time token-by-token responses:

```csharp
await "Tell me a story"
    .GENCompletion()
    .SetModel(OpenAIModel.GPT4o)
    .StreamAsync(
        onToken: token => Debug.Log(token),
        onComplete: response => Debug.Log("Done!")
    );
```

## Provider Support

| Provider | Support | Models |
|----------|---------|--------|
| OpenAI | ✅ Full | GPT-4o, GPT-4, GPT-3.5 |
| Anthropic | ✅ Full | Claude 3.5, Claude 3 |
| Google Gemini | ✅ Full | Gemini 1.5, Gemini 1.0 |
| OpenRouter | ✅ Full | All models |
| Groq | ✅ Full | Llama 3, Mixtral |
| xAI | ✅ Full | Grok |
| Perplexity | ✅ Full | All models |
| Azure OpenAI | ✅ Full | GPT-4, GPT-3.5 |
| Ollama | ✅ Full | All local models |

## Examples

### Example 1: Chatbot Response

```csharp
string response = await userInput
    .GENCompletion()
    .SetModel(OpenAIModel.GPT4o)
    .SetSystemMessage("You are a friendly game NPC")
    .SetTemperature(0.8f)
    .ExecuteAsync();

npcDialogue.text = response;
```

### Example 2: FAQ System

```csharp
async UniTask<string> AnswerFAQ(string question)
{
    return await question
        .GENCompletion()
        .SetModel(OpenAIModel.GPT4oMini)
        .SetSystemMessage("Answer questions about our Unity plugin")
        .SetTemperature(0.3f)
        .SetMaxTokens(200)
        .ExecuteAsync();
}
```

### Example 3: Contextual Help

```csharp
async UniTask<string> GetContextualHelp(string topic)
{
    return await $"Explain {topic} for Unity beginners"
        .GENCompletion()
        .SetModel(OpenAIModel.GPT4o)
        .SetSystemMessage("You are a Unity tutorial assistant")
        .SetTemperature(0.5f)
        .ExecuteAsync();
}
```

### Example 4: Translation

```csharp
async UniTask<string> TranslateText(string text, string targetLang)
{
    return await $"Translate to {targetLang}: {text}"
        .GENCompletion()
        .SetModel(OpenAIModel.GPT4oMini)
        .SetTemperature(0.0f)
        .ExecuteAsync();
}
```

## Error Handling

```csharp
try
{
    string response = await "Hello"
        .GENCompletion()
        .ExecuteAsync();
}
catch (AIApiException ex)
{
    Debug.LogError($"API Error: {ex.Message}");
    // Handle: rate limit, invalid API key, model not found, etc.
}
catch (System.Net.Http.HttpRequestException ex)
{
    Debug.LogError($"Network Error: {ex.Message}");
    // Handle: no internet connection, timeout, etc.
}
```

## Best Practices

### ✅ Do

- Use lower temperature (0.0-0.3) for factual answers
- Use higher temperature (0.7-1.5) for creative content
- Set reasonable max tokens to control costs
- Cache responses when possible
- Handle errors gracefully

### ❌ Don't

- Set temperature above 2.0 (unstable results)
- Use very high frequency/presence penalties (degrades quality)
- Forget to set system messages for context
- Ignore error handling
- Call API in tight loops without rate limiting

## Next Steps

- [Responses API](responses-api.md) - More advanced features
- [Code Generation](code-generation.md) - Specialized for code
- [Structured Output](structured-output.md) - Type-safe JSON
