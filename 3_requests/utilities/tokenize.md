---
icon: bars
---

# Tokenize

Convert text into token IDs using `.Tokenize()`.

## Basic Usage

```csharp
int[] tokens = await "Hello, world!"
    .Tokenize()
    .ExecuteAsync();

Debug.Log($"Token IDs: {string.Join(", ", tokens)}");
Debug.Log($"Token count: {tokens.Length}");
```

## Input Types

### String Input

```csharp
int[] tokens = await "Sample text"
    .Tokenize()
    .ExecuteAsync();
```

### Prompt Input

```csharp
var prompt = new Prompt("Analyze this text");
int[] tokens = await prompt
    .Tokenize()
    .ExecuteAsync();
```

## What Are Tokens?

Tokens are the basic units that AI models process. Understanding tokens helps you:

- **Estimate costs** - Most APIs charge per token
- **Manage limits** - Models have maximum token counts
- **Optimize prompts** - Shorter prompts = lower cost

### Token Examples

```csharp
// Simple word
"Hello" → [15496]

// Word with punctuation
"Hello!" → [15496, 0]

// Sentence
"Hello, world!" → [15496, 11, 1917, 0]
```

## Unity Integration Examples

### Example 1: Token Counter Display

```csharp
public class TokenCounterUI : MonoBehaviour
{
    [SerializeField] private TMP_InputField inputField;
    [SerializeField] private TMPro.TextMeshProUGUI countText;
    
    async void OnInputChanged(string text)
    {
        if (string.IsNullOrEmpty(text))
        {
            countText.text = "0 tokens";
            return;
        }
        
        int[] tokens = await text.Tokenize().ExecuteAsync();
        countText.text = $"{tokens.Length} tokens";
    }
}
```

### Example 2: Token Inspector

```csharp
public class TokenInspector : MonoBehaviour
{
    public async UniTask InspectText(string text)
    {
        int[] tokens = await text.Tokenize().ExecuteAsync();
        
        Debug.Log($"Text: {text}");
        Debug.Log($"Token Count: {tokens.Length}");
        Debug.Log($"Token IDs: {string.Join(", ", tokens)}");
    }
}
```

### Example 3: Analyze Prompt Efficiency

```csharp
public class PromptAnalyzer : MonoBehaviour
{
    public async UniTask<AnalysisResult> AnalyzePrompt(string prompt)
    {
        int[] tokens = await prompt.Tokenize().ExecuteAsync();
        
        return new AnalysisResult
        {
            TokenCount = tokens.Length,
            CharCount = prompt.Length,
            Efficiency = (float)prompt.Length / tokens.Length,
            EstimatedCost = CalculateCost(tokens.Length)
        };
    }
    
    float CalculateCost(int tokenCount)
    {
        // Example: $0.01 per 1000 tokens
        return (tokenCount / 1000f) * 0.01f;
    }
}
```

## Tokenization vs Token Counting

| Method | Returns | Use Case |
|--------|---------|----------|
| `.Tokenize()` | Token IDs (int[]) | Detailed analysis |
| `.CountTokens()` | Token count (int) | Quick count only |

**Use `.CountTokens()` instead** if you only need the count:

```csharp
// ✅ Better for just counting
int count = await text.CountTokens().ExecuteAsync();

// ❌ Wasteful if you only need count
int[] tokens = await text.Tokenize().ExecuteAsync();
int count = tokens.Length;
```

## Provider Support

### OpenAI

```csharp
int[] tokens = await text
    .Tokenize()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

### Anthropic

```csharp
int[] tokens = await text
    .Tokenize()
    .SetModel(AnthropicModel.Claude35Sonnet)
    .ExecuteAsync();
```

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Use for detailed token analysis
int[] tokens = await text.Tokenize().ExecuteAsync();
AnalyzeTokenDistribution(tokens);

// ✅ Cache results for repeated text
Dictionary<string, int[]> tokenCache = new();

// ✅ Use CountTokens() for simple counts
int count = await text.CountTokens().ExecuteAsync();
```

### ❌ Bad Practices

```csharp
// ❌ Don't tokenize in Update()
void Update()
{
    await text.Tokenize().ExecuteAsync();  // NO!
}

// ❌ Don't use Tokenize when CountTokens is enough
int[] tokens = await text.Tokenize().ExecuteAsync();
int count = tokens.Length;  // Use CountTokens() instead!
```

## Understanding Token Patterns

### Common Patterns

```csharp
// Spaces count as tokens
"Hello world" → 2 tokens
"Hello  world" → 3 tokens (extra space = extra token)

// Punctuation
"Hello!" → 2 tokens
"Hello." → 2 tokens

// Numbers
"123" → 1 token
"1234567890" → 2-3 tokens

// Unicode
"안녕하세요" → 5-10 tokens (language dependent)
```

## Next Steps

- [Count Tokens](count-tokens.md) - Quick token counting
- [Get Credits](get-credits.md) - Check account balance
