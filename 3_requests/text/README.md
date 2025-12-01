---
icon: align-left
---

# Text Generation

AI Dev Kit provides multiple methods for generating text content, each optimized for different use cases.

## Available Methods

### 1. Chat Completions (`.GENCompletion()`)

General-purpose chat and text generation:

```csharp
string response = await "Explain quantum computing"
    .GENCompletion()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

**Best for:**

- ✅ Conversational AI
- ✅ General text generation
- ✅ Multi-turn conversations
- ✅ Wide provider support

### 2. Responses API (`.GENResponse()`)

Most capable text generation with advanced features:

```csharp
string response = await "Write a technical spec"
    .GENResponse()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

**Best for:**

- ✅ Complex reasoning tasks
- ✅ Long-form content
- ✅ Tool calling support
- ✅ Latest AI capabilities

### 3. Code Generation (`.GENCode()`)

Specialized for code generation and refactoring:

```csharp
string code = await "Create a C# quicksort algorithm"
    .GENCode()
    .ExecuteAsync();
```

**Best for:**

- ✅ Code generation
- ✅ Code explanation
- ✅ Refactoring suggestions
- ✅ Bug fixes

### 4. Structured Output (`.GENStruct<T>()`)

Generate JSON mapped to strongly-typed C# classes:

```csharp
UserProfile profile = await "Create profile for John Doe, age 30"
    .GENStruct<UserProfile>()
    .ExecuteAsync();
```

**Best for:**

- ✅ Parsing AI output into C# objects
- ✅ Data extraction
- ✅ Form filling
- ✅ Type-safe responses

## Quick Comparison

| Method | Use Case | Provider Support | Complexity |
|--------|----------|-----------------|------------|
| `GENCompletion()` | General chat | Most providers | Simple |
| `GENResponse()` | Advanced features | Limited providers | Advanced |
| `GENCode()` | Code generation | Most providers | Simple |
| `GENStruct<T>()` | JSON output | Most providers | Medium |

## Basic Examples

### Example 1: Simple Question

```csharp
string answer = await "What is 2+2?"
    .GENCompletion()
    .ExecuteAsync();

Debug.Log(answer); // "2+2 equals 4."
```

### Example 2: Story Generation

```csharp
string story = await "Write a short sci-fi story about AI"
    .GENResponse()
    .SetModel(OpenAIModel.GPT4o)
    .SetMaxTokens(500)
    .ExecuteAsync();

Debug.Log(story);
```

### Example 3: Code Generation

```csharp
string code = await "Create a Unity script that rotates an object"
    .GENCode()
    .ExecuteAsync();

Debug.Log(code);
```

### Example 4: Structured Data

```csharp
[JsonSchema]
public class Character
{
    public string Name { get; set; }
    public int Level { get; set; }
    public string Class { get; set; }
}

Character hero = await "Generate a level 5 warrior named Aragorn"
    .GENStruct<Character>()
    .ExecuteAsync();

Debug.Log($"{hero.Name} (Lv.{hero.Level} {hero.Class})");
```

## Configuration Options

All text generation methods support common configuration:

```csharp
string response = await "Your prompt"
    .GENCompletion()
    .SetModel(OpenAIModel.GPT4o)           // Model selection
    .SetTemperature(0.7f)                   // Creativity (0.0-2.0)
    .SetMaxTokens(1000)                     // Max response length
    .SetSystemMessage("You are helpful")   // Context
    .SetTopP(0.9f)                          // Nucleus sampling
    .SetFrequencyPenalty(0.5f)              // Reduce repetition
    .SetPresencePenalty(0.5f)               // Encourage diversity
    .ExecuteAsync();
```

## Next Steps

- [Chat Completions](chat-completions.md) - Detailed chat API guide
- [Responses API](responses-api.md) - Advanced features
- [Code Generation](code-generation.md) - Code-specific options
- [Structured Output](structured-output.md) - JSON schema guide
