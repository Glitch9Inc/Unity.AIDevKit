---
icon: wrench
---

# Utilities

AI Dev Kit provides utility functions for tokenization, token counting, and billing.

## Available Utilities

### Tokenization

Convert text to token IDs:

```csharp
int[] tokens = await "Hello, world!"
    .Tokenize()
    .ExecuteAsync();

Debug.Log($"Token count: {tokens.Length}");
```

### Token Counting

Count tokens without getting IDs:

```csharp
int count = await "Long text content..."
    .CountTokens()
    .ExecuteAsync();

Debug.Log($"Tokens: {count}");
```

### Credits / Billing

Check account credits:

```csharp
var credits = await Api.OpenAI
    .GetCredits()
    .ExecuteAsync();

Debug.Log($"Remaining: ${credits.Balance}");
```

## Common Use Cases

### Check Cost Before Request

```csharp
public async UniTask<bool> CheckCostAndProceed(string prompt)
{
    int tokenCount = await prompt.CountTokens().ExecuteAsync();
    float estimatedCost = CalculateCost(tokenCount);
    
    if (estimatedCost > maxCost)
    {
        Debug.LogWarning($"Cost too high: ${estimatedCost}");
        return false;
    }
    
    return true;
}
```

### Token Budget Management

```csharp
public async UniTask<string> TrimToTokenBudget(string text, int maxTokens)
{
    int tokens = await text.CountTokens().ExecuteAsync();
    
    if (tokens <= maxTokens)
        return text;
    
    // Trim text to fit budget
    return TrimText(text, maxTokens);
}
```

### Billing Monitor

```csharp
public class BillingMonitor : MonoBehaviour
{
    async void Start()
    {
        var credits = await Api.OpenAI.GetCredits().ExecuteAsync();
        
        if (credits.Balance < warningThreshold)
        {
            ShowLowBalanceWarning();
        }
    }
}
```

## Next Steps

- [Tokenize](tokenize.md) - Convert text to tokens
- [Count Tokens](count-tokens.md) - Count tokens
- [Get Credits](get-credits.md) - Check billing
