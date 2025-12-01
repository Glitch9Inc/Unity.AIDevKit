---
icon: calculator
---

# Count Tokens

Count tokens in text using `.CountTokens()`.

## Basic Usage

```csharp
int count = await "Hello, world!"
    .CountTokens()
    .ExecuteAsync();

Debug.Log($"Token count: {count}");
```

## Input Types

### String Input

```csharp
int count = await "Sample text"
    .CountTokens()
    .ExecuteAsync();
```

### Message Input

```csharp
var message = new UserMessage("Hello, AI!");
int count = await message
    .CountTokens()
    .ExecuteAsync();
```

## Unity Integration Examples

### Example 1: Budget Manager

```csharp
public class TokenBudgetManager : MonoBehaviour
{
    [SerializeField] private int maxTokens = 1000;
    
    public async UniTask<bool> IsWithinBudget(string text)
    {
        int count = await text.CountTokens().ExecuteAsync();
        return count <= maxTokens;
    }
    
    public async UniTask<string> TrimToBudget(string text)
    {
        int count = await text.CountTokens().ExecuteAsync();
        
        if (count <= maxTokens)
            return text;
        
        // Trim text to fit budget
        float ratio = (float)maxTokens / count;
        int targetLength = (int)(text.Length * ratio);
        return text.Substring(0, targetLength);
    }
}
```

### Example 2: Cost Estimator

```csharp
public class CostEstimator : MonoBehaviour
{
    private const float COST_PER_1K_TOKENS = 0.01f;
    
    public async UniTask<float> EstimateCost(string prompt, string expectedResponse)
    {
        int inputTokens = await prompt.CountTokens().ExecuteAsync();
        int outputTokens = await expectedResponse.CountTokens().ExecuteAsync();
        
        int totalTokens = inputTokens + outputTokens;
        float cost = (totalTokens / 1000f) * COST_PER_1K_TOKENS;
        
        Debug.Log($"Input: {inputTokens} tokens");
        Debug.Log($"Output: {outputTokens} tokens");
        Debug.Log($"Estimated cost: ${cost:F4}");
        
        return cost;
    }
}
```

### Example 3: Real-time Token Counter

```csharp
public class RealtimeTokenCounter : MonoBehaviour
{
    [SerializeField] private TMP_InputField inputField;
    [SerializeField] private TMPro.TextMeshProUGUI counterText;
    [SerializeField] private int warningThreshold = 500;
    [SerializeField] private int maxTokens = 1000;
    
    private float lastUpdateTime;
    private const float UPDATE_INTERVAL = 0.5f;
    
    void Start()
    {
        inputField.onValueChanged.AddListener(OnTextChanged);
    }
    
    void OnTextChanged(string text)
    {
        if (Time.time - lastUpdateTime < UPDATE_INTERVAL)
            return;
        
        lastUpdateTime = Time.time;
        UpdateTokenCountAsync(text).Forget();
    }
    
    async UniTaskVoid UpdateTokenCountAsync(string text)
    {
        if (string.IsNullOrEmpty(text))
        {
            counterText.text = "0 tokens";
            counterText.color = Color.white;
            return;
        }
        
        int count = await text.CountTokens().ExecuteAsync();
        
        counterText.text = $"{count} / {maxTokens} tokens";
        
        if (count >= maxTokens)
            counterText.color = Color.red;
        else if (count >= warningThreshold)
            counterText.color = Color.yellow;
        else
            counterText.color = Color.white;
    }
}
```

### Example 4: Conversation Token Tracker

```csharp
public class ConversationTokenTracker : MonoBehaviour
{
    private int totalTokens = 0;
    private const int MAX_CONTEXT = 4000;
    
    public async UniTask<bool> CanAddMessage(string message)
    {
        int messageTokens = await message.CountTokens().ExecuteAsync();
        return totalTokens + messageTokens <= MAX_CONTEXT;
    }
    
    public async UniTask AddMessage(string message)
    {
        int messageTokens = await message.CountTokens().ExecuteAsync();
        totalTokens += messageTokens;
        
        Debug.Log($"Added message: {messageTokens} tokens");
        Debug.Log($"Total: {totalTokens} / {MAX_CONTEXT} tokens");
    }
    
    public void Reset()
    {
        totalTokens = 0;
    }
}
```

## Provider Support

### OpenAI

```csharp
int count = await text
    .CountTokens()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();
```

### Anthropic

```csharp
int count = await text
    .CountTokens()
    .SetModel(AnthropicModel.Claude35Sonnet)
    .ExecuteAsync();
```

### Google

```csharp
int count = await text
    .CountTokens()
    .SetModel(GoogleModel.Gemini15Pro)
    .ExecuteAsync();
```

## Token Counting vs Tokenization

| Method | Returns | Performance | Use Case |
|--------|---------|-------------|----------|
| `.CountTokens()` | int | Faster | Budget checks |
| `.Tokenize()` | int[] | Slower | Detailed analysis |

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Check before sending request
int count = await prompt.CountTokens().ExecuteAsync();
if (count > maxTokens)
{
    prompt = TrimPrompt(prompt, maxTokens);
}

// ✅ Cache counts for static text
Dictionary<string, int> countCache = new();

// ✅ Use for cost estimation
float cost = (count / 1000f) * costPer1K;

// ✅ Implement token budgets
if (await IsWithinBudget(text))
{
    await SendRequest(text);
}
```

### ❌ Bad Practices

```csharp
// ❌ Don't count in Update()
void Update()
{
    await text.CountTokens().ExecuteAsync();  // NO!
}

// ❌ Don't ignore token limits
// Always check before sending

// ❌ Don't count unnecessarily
// Cache results for repeated text
```

## Performance Tips

```csharp
// ✅ Good - cache results
Dictionary<string, int> cache = new();

async UniTask<int> GetCachedTokenCount(string text)
{
    if (!cache.ContainsKey(text))
    {
        cache[text] = await text.CountTokens().ExecuteAsync();
    }
    return cache[text];
}

// ✅ Good - batch operations
var tasks = texts.Select(t => t.CountTokens().ExecuteAsync());
int[] counts = await UniTask.WhenAll(tasks);
```

## Common Token Counts

| Text | Approximate Tokens |
|------|-------------------|
| Single word | 1-2 |
| Short sentence | 10-20 |
| Paragraph | 50-100 |
| Page of text | 300-500 |
| Article | 1000-2000 |

## Next Steps

- [Tokenize](tokenize.md) - Get detailed token IDs
- [Get Credits](get-credits.md) - Check account balance
