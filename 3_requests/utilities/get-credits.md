---
icon: wallet
---

# Get Credits

Check your account balance and usage using `.GetCredits()`.

## Basic Usage

```csharp
var credits = await Api.OpenAI
    .GetCredits()
    .ExecuteAsync();

Debug.Log($"Balance: ${credits.Balance}");
Debug.Log($"Used: ${credits.Used}");
Debug.Log($"Total: ${credits.Total}");
```

## Response Properties

```csharp
public class CreditsResponse
{
    public float Balance { get; set; }   // Remaining balance
    public float Used { get; set; }      // Amount used
    public float Total { get; set; }     // Total allocated
}
```

## Unity Integration Examples

### Example 1: Balance Monitor

```csharp
public class BalanceMonitor : MonoBehaviour
{
    [SerializeField] private float warningThreshold = 10f;
    [SerializeField] private TMPro.TextMeshProUGUI balanceText;
    
    async void Start()
    {
        await CheckBalance();
    }
    
    async UniTask CheckBalance()
    {
        var credits = await Api.OpenAI.GetCredits().ExecuteAsync();
        
        balanceText.text = $"Balance: ${credits.Balance:F2}";
        
        if (credits.Balance < warningThreshold)
        {
            ShowLowBalanceWarning();
        }
    }
    
    void ShowLowBalanceWarning()
    {
        Debug.LogWarning("⚠️ Low balance! Please add credits.");
    }
}
```

### Example 2: Usage Dashboard

```csharp
public class UsageDashboard : MonoBehaviour
{
    [SerializeField] private TMPro.TextMeshProUGUI balanceText;
    [SerializeField] private TMPro.TextMeshProUGUI usedText;
    [SerializeField] private TMPro.TextMeshProUGUI totalText;
    [SerializeField] private UnityEngine.UI.Slider usageSlider;
    
    async void Start()
    {
        await RefreshDashboard();
    }
    
    public async UniTask RefreshDashboard()
    {
        var credits = await Api.OpenAI.GetCredits().ExecuteAsync();
        
        balanceText.text = $"${credits.Balance:F2}";
        usedText.text = $"${credits.Used:F2}";
        totalText.text = $"${credits.Total:F2}";
        
        float usagePercent = credits.Used / credits.Total;
        usageSlider.value = usagePercent;
    }
}
```

### Example 3: Pre-request Budget Check

```csharp
public class BudgetChecker : MonoBehaviour
{
    [SerializeField] private float minimumBalance = 1f;
    
    public async UniTask<bool> CanMakeRequest(float estimatedCost)
    {
        var credits = await Api.OpenAI.GetCredits().ExecuteAsync();
        
        if (credits.Balance < minimumBalance)
        {
            Debug.LogWarning("Balance too low to make request");
            return false;
        }
        
        if (credits.Balance < estimatedCost)
        {
            Debug.LogWarning($"Insufficient balance for this request (need ${estimatedCost:F2})");
            return false;
        }
        
        return true;
    }
}
```

### Example 4: Periodic Balance Check

```csharp
public class PeriodicBalanceChecker : MonoBehaviour
{
    [SerializeField] private float checkInterval = 300f; // 5 minutes
    [SerializeField] private float lowBalanceThreshold = 5f;
    
    async void Start()
    {
        InvokeRepeating(nameof(CheckBalanceAsync), 0f, checkInterval);
    }
    
    async void CheckBalanceAsync()
    {
        try
        {
            var credits = await Api.OpenAI.GetCredits().ExecuteAsync();
            
            Debug.Log($"Current Balance: ${credits.Balance:F2}");
            
            if (credits.Balance < lowBalanceThreshold)
            {
                OnLowBalance(credits);
            }
        }
        catch (Exception ex)
        {
            Debug.LogError($"Failed to check balance: {ex.Message}");
        }
    }
    
    void OnLowBalance(CreditsResponse credits)
    {
        Debug.LogWarning($"⚠️ Low balance: ${credits.Balance:F2}");
        // Show UI notification, send alert, etc.
    }
}
```

## Provider Support

### OpenAI

```csharp
var credits = await Api.OpenAI.GetCredits().ExecuteAsync();
```

### GroqCloud

```csharp
var credits = await Api.Groq.GetCredits().ExecuteAsync();
```

**Note:** Not all providers support credit/billing queries. Check provider documentation.

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Check balance periodically
async void Start()
{
    InvokeRepeating(nameof(CheckBalance), 0f, 300f);
}

// ✅ Verify before expensive operations
async UniTask<bool> VerifyBalanceBeforeRequest()
{
    var credits = await Api.OpenAI.GetCredits().ExecuteAsync();
    return credits.Balance > minimumRequired;
}

// ✅ Cache balance for short periods
private CreditsResponse cachedCredits;
private float lastCheckTime;

async UniTask<CreditsResponse> GetCachedCredits()
{
    if (Time.time - lastCheckTime > 60f)
    {
        cachedCredits = await Api.OpenAI.GetCredits().ExecuteAsync();
        lastCheckTime = Time.time;
    }
    return cachedCredits;
}

// ✅ Handle errors gracefully
try
{
    var credits = await Api.OpenAI.GetCredits().ExecuteAsync();
}
catch (Exception ex)
{
    Debug.LogError($"Failed to get credits: {ex.Message}");
}
```

### ❌ Bad Practices

```csharp
// ❌ Don't check in Update()
void Update()
{
    await Api.OpenAI.GetCredits().ExecuteAsync();  // NO!
}

// ❌ Don't ignore balance warnings
// Always handle low balance scenarios

// ❌ Don't check too frequently
// Cache results or use reasonable intervals
```

## Error Handling

```csharp
public async UniTask<CreditsResponse> GetCreditsWithFallback()
{
    try
    {
        return await Api.OpenAI.GetCredits().ExecuteAsync();
    }
    catch (AIApiException ex)
    {
        Debug.LogError($"API Error: {ex.Message}");
        return null;
    }
    catch (Exception ex)
    {
        Debug.LogError($"Unexpected error: {ex.Message}");
        return null;
    }
}
```

## Next Steps

- [Tokenize](tokenize.md) - Get token IDs
- [Count Tokens](count-tokens.md) - Count tokens for cost estimation
