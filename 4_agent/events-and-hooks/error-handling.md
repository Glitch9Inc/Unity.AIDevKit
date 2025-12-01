# Error Handling Events

Handle and recover from errors gracefully.

## Overview

Error Handling Events provide:

- Comprehensive error detection
- Error categorization
- Recovery strategies
- Error logging
- User notifications

## Basic Setup

### Subscribe to Error Events

```csharp
public class ErrorEventListener : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.onAgentError.AddListener(OnAgentError);
        agent.onRequestError.AddListener(OnRequestError);
        agent.onToolError.AddListener(OnToolError);
        agent.onAudioError.AddListener(OnAudioError);
    }
    
    void OnAgentError(string error)
    {
        Debug.LogError($"Agent error: {error}");
    }
    
    void OnRequestError(string error)
    {
        Debug.LogError($"Request error: {error}");
    }
    
    void OnToolError(string toolName, string error)
    {
        Debug.LogError($"Tool error ({toolName}): {error}");
    }
    
    void OnAudioError(string error)
    {
        Debug.LogError($"Audio error: {error}");
    }
}
```

## Error Categories

### Categorize and Handle Errors

```csharp
public class ErrorCategorizer : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.onAgentError.AddListener(OnAgentError);
    }
    
    void OnAgentError(string error)
    {
        ErrorCategory category = CategorizeError(error);
        
        Debug.LogError($"[{category}] {error}");
        
        HandleError(category, error);
    }
    
    ErrorCategory CategorizeError(string error)
    {
        if (error.Contains("api_key") || error.Contains("unauthorized"))
            return ErrorCategory.Authentication;
        
        if (error.Contains("timeout") || error.Contains("network"))
            return ErrorCategory.Network;
        
        if (error.Contains("rate_limit"))
            return ErrorCategory.RateLimit;
        
        if (error.Contains("invalid") || error.Contains("format"))
            return ErrorCategory.Validation;
        
        if (error.Contains("quota") || error.Contains("insufficient"))
            return ErrorCategory.Quota;
        
        return ErrorCategory.Unknown;
    }
    
    void HandleError(ErrorCategory category, string error)
    {
        switch (category)
        {
            case ErrorCategory.Authentication:
                HandleAuthenticationError(error);
                break;
                
            case ErrorCategory.Network:
                HandleNetworkError(error);
                break;
                
            case ErrorCategory.RateLimit:
                HandleRateLimitError(error);
                break;
                
            case ErrorCategory.Validation:
                HandleValidationError(error);
                break;
                
            case ErrorCategory.Quota:
                HandleQuotaError(error);
                break;
                
            case ErrorCategory.Unknown:
                HandleUnknownError(error);
                break;
        }
    }
    
    void HandleAuthenticationError(string error)
    {
        Debug.LogError("🔐 Authentication error");
        ShowErrorDialog("Authentication Error", "Please check your API key.");
    }
    
    void HandleNetworkError(string error)
    {
        Debug.LogError("🌐 Network error");
        ShowErrorDialog("Network Error", "Please check your internet connection.");
        RetryWithBackoff().Forget();
    }
    
    void HandleRateLimitError(string error)
    {
        Debug.LogError("⏱️ Rate limit error");
        ShowErrorDialog("Rate Limit", "Too many requests. Please wait.");
    }
    
    void HandleValidationError(string error)
    {
        Debug.LogError("⚠️ Validation error");
        ShowErrorDialog("Invalid Input", error);
    }
    
    void HandleQuotaError(string error)
    {
        Debug.LogError("💳 Quota error");
        ShowErrorDialog("Quota Exceeded", "Your usage limit has been reached.");
    }
    
    void HandleUnknownError(string error)
    {
        Debug.LogError("❓ Unknown error");
        ShowErrorDialog("Error", "An unexpected error occurred.");
    }
    
    async UniTaskVoid RetryWithBackoff()
    {
        await UniTask.Delay(TimeSpan.FromSeconds(5));
        // Retry logic
    }
    
    void ShowErrorDialog(string title, string message)
    {
        // Show error UI
    }
}

public enum ErrorCategory
{
    Authentication,
    Network,
    RateLimit,
    Validation,
    Quota,
    Unknown
}
```

## Retry Strategies

### Implement Smart Retry

```csharp
public class RetryManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int maxRetries = 3;
    [SerializeField] private float baseDelay = 2f;
    
    private Dictionary<string, int> retryCount = new();
    
    void Start()
    {
        agent.onRequestError.AddListener(OnRequestError);
    }
    
    void OnRequestError(string error)
    {
        string requestId = agent.CurrentRequestId;
        
        if (ShouldRetry(requestId, error))
        {
            RetryRequest(requestId, error).Forget();
        }
        else
        {
            HandleFailedRequest(requestId, error);
        }
    }
    
    bool ShouldRetry(string requestId, string error)
    {
        // Get retry count
        if (!retryCount.ContainsKey(requestId))
        {
            retryCount[requestId] = 0;
        }
        
        if (retryCount[requestId] >= maxRetries)
        {
            return false;
        }
        
        // Only retry on transient errors
        return error.Contains("timeout") ||
               error.Contains("network") ||
               error.Contains("503") ||
               error.Contains("rate_limit");
    }
    
    async UniTaskVoid RetryRequest(string requestId, string error)
    {
        retryCount[requestId]++;
        int attempt = retryCount[requestId];
        
        // Exponential backoff
        float delay = baseDelay * Mathf.Pow(2, attempt - 1);
        
        Debug.Log($"🔄 Retrying in {delay}s (attempt {attempt}/{maxRetries})");
        
        await UniTask.Delay(TimeSpan.FromSeconds(delay));
        
        try
        {
            await agent.RetryLastRequestAsync();
            
            // Success - reset count
            retryCount.Remove(requestId);
            Debug.Log("✓ Retry successful");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Retry failed: {ex.Message}");
        }
    }
    
    void HandleFailedRequest(string requestId, string error)
    {
        Debug.LogError($"❌ Request failed after {maxRetries} retries");
        
        ShowErrorNotification("Request failed. Please try again later.");
        
        retryCount.Remove(requestId);
    }
    
    void ShowErrorNotification(string message)
    {
        // Show error UI
    }
}
```

## Error Logging

### Comprehensive Error Logging

```csharp
public class ErrorLogger : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private string logPath = "ErrorLogs";
    [SerializeField] private int maxLogSize = 1000;
    
    private List<ErrorLog> errorLogs = new();
    
    void Start()
    {
        agent.onAgentError.AddListener(OnAgentError);
        agent.onRequestError.AddListener(OnRequestError);
        agent.onToolError.AddListener(OnToolError);
    }
    
    void OnAgentError(string error)
    {
        LogError("Agent", error);
    }
    
    void OnRequestError(string error)
    {
        LogError("Request", error);
    }
    
    void OnToolError(string toolName, string error)
    {
        LogError($"Tool:{toolName}", error);
    }
    
    void LogError(string source, string error)
    {
        var log = new ErrorLog
        {
            Timestamp = DateTime.Now,
            Source = source,
            Error = error,
            StackTrace = System.Environment.StackTrace
        };
        
        errorLogs.Add(log);
        
        Debug.LogError($"[{source}] {error}");
        
        // Limit log size
        if (errorLogs.Count > maxLogSize)
        {
            errorLogs.RemoveAt(0);
        }
        
        // Save periodically
        if (errorLogs.Count % 10 == 0)
        {
            SaveLogs();
        }
    }
    
    void SaveLogs()
    {
        string json = JsonUtility.ToJson(new ErrorLogCollection { logs = errorLogs }, true);
        
        string filePath = Path.Combine(
            Application.persistentDataPath,
            logPath,
            $"errors_{DateTime.Now:yyyyMMdd}.json"
        );
        
        Directory.CreateDirectory(Path.GetDirectoryName(filePath));
        File.WriteAllText(filePath, json);
        
        Debug.Log($"💾 Error logs saved: {filePath}");
    }
    
    void OnApplicationQuit()
    {
        SaveLogs();
    }
}

[Serializable]
public class ErrorLog
{
    public DateTime Timestamp;
    public string Source;
    public string Error;
    public string StackTrace;
}

[Serializable]
public class ErrorLogCollection
{
    public List<ErrorLog> logs;
}
```

## Error Recovery

### Automatic Recovery

```csharp
public class ErrorRecovery : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int consecutiveErrorThreshold = 3;
    
    private int consecutiveErrors;
    private DateTime lastError;
    
    void Start()
    {
        agent.onAgentError.AddListener(OnAgentError);
        agent.onRequestCompleted.AddListener(OnRequestCompleted);
    }
    
    void OnAgentError(string error)
    {
        consecutiveErrors++;
        lastError = DateTime.Now;
        
        Debug.LogError($"Error #{consecutiveErrors}: {error}");
        
        if (consecutiveErrors >= consecutiveErrorThreshold)
        {
            Debug.LogError("❌ Multiple consecutive errors - attempting recovery");
            AttemptRecovery().Forget();
        }
    }
    
    void OnRequestCompleted()
    {
        consecutiveErrors = 0;
    }
    
    async UniTaskVoid AttemptRecovery()
    {
        Debug.Log("🔄 Attempting recovery...");
        
        // Stop agent
        agent.Stop();
        
        await UniTask.Delay(TimeSpan.FromSeconds(2));
        
        // Reinitialize
        agent.Initialize();
        
        await UniTask.Delay(TimeSpan.FromSeconds(1));
        
        // Restart
        agent.Start();
        
        consecutiveErrors = 0;
        
        Debug.Log("✓ Recovery complete");
    }
}
```

## User Notifications

### Error Notification System

```csharp
public class ErrorNotificationSystem : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject errorDialog;
    [SerializeField] private TMP_Text errorTitle;
    [SerializeField] private TMP_Text errorMessage;
    [SerializeField] private Button retryButton;
    [SerializeField] private Button dismissButton;
    
    private string lastError;
    
    void Start()
    {
        agent.onAgentError.AddListener(OnAgentError);
        
        retryButton.onClick.AddListener(OnRetryClicked);
        dismissButton.onClick.AddListener(OnDismissClicked);
    }
    
    void OnAgentError(string error)
    {
        lastError = error;
        
        ErrorInfo info = ParseError(error);
        
        ShowErrorDialog(info);
    }
    
    ErrorInfo ParseError(string error)
    {
        var info = new ErrorInfo
        {
            Title = "Error",
            Message = error,
            CanRetry = false
        };
        
        if (error.Contains("network") || error.Contains("timeout"))
        {
            info.Title = "Network Error";
            info.Message = "Unable to connect. Please check your internet connection.";
            info.CanRetry = true;
        }
        else if (error.Contains("api_key"))
        {
            info.Title = "Authentication Error";
            info.Message = "Invalid API key. Please check your settings.";
            info.CanRetry = false;
        }
        else if (error.Contains("rate_limit"))
        {
            info.Title = "Rate Limit";
            info.Message = "Too many requests. Please wait a moment.";
            info.CanRetry = true;
        }
        
        return info;
    }
    
    void ShowErrorDialog(ErrorInfo info)
    {
        errorTitle.text = info.Title;
        errorMessage.text = info.Message;
        retryButton.gameObject.SetActive(info.CanRetry);
        
        errorDialog.SetActive(true);
    }
    
    async void OnRetryClicked()
    {
        errorDialog.SetActive(false);
        
        try
        {
            await agent.RetryLastRequestAsync();
        }
        catch (Exception ex)
        {
            Debug.LogError($"Retry failed: {ex.Message}");
        }
    }
    
    void OnDismissClicked()
    {
        errorDialog.SetActive(false);
    }
}

public class ErrorInfo
{
    public string Title { get; set; }
    public string Message { get; set; }
    public bool CanRetry { get; set; }
}
```

## Analytics Integration

### Track Error Metrics

```csharp
public class ErrorAnalytics : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Dictionary<string, int> errorCounts = new();
    private int totalErrors;
    
    void Start()
    {
        agent.onAgentError.AddListener(OnAgentError);
    }
    
    void OnAgentError(string error)
    {
        totalErrors++;
        
        string errorType = GetErrorType(error);
        
        if (!errorCounts.ContainsKey(errorType))
        {
            errorCounts[errorType] = 0;
        }
        errorCounts[errorType]++;
        
        // Send to analytics
        TrackError(errorType, error);
        
        // Log summary every 10 errors
        if (totalErrors % 10 == 0)
        {
            LogErrorSummary();
        }
    }
    
    string GetErrorType(string error)
    {
        if (error.Contains("network")) return "Network";
        if (error.Contains("api_key")) return "Authentication";
        if (error.Contains("rate_limit")) return "RateLimit";
        if (error.Contains("timeout")) return "Timeout";
        if (error.Contains("invalid")) return "Validation";
        return "Unknown";
    }
    
    void TrackError(string errorType, string error)
    {
        // Send to analytics service
        Debug.Log($"📊 Analytics: {errorType} error");
    }
    
    void LogErrorSummary()
    {
        Debug.Log("=== Error Summary ===");
        Debug.Log($"Total errors: {totalErrors}");
        
        foreach (var kvp in errorCounts.OrderByDescending(x => x.Value))
        {
            float percentage = (float)kvp.Value / totalErrors * 100;
            Debug.Log($"{kvp.Key}: {kvp.Value} ({percentage:F1}%)");
        }
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class ErrorHandlingManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("UI")]
    [SerializeField] private GameObject errorDialog;
    [SerializeField] private TMP_Text errorTitle;
    [SerializeField] private TMP_Text errorMessage;
    [SerializeField] private Button retryButton;
    
    [Header("Settings")]
    [SerializeField] private int maxRetries = 3;
    [SerializeField] private float baseRetryDelay = 2f;
    [SerializeField] private bool enableLogging = true;
    
    private Dictionary<string, int> retryCount = new();
    private List<ErrorLog> errorLogs = new();
    private int consecutiveErrors;
    
    void Start()
    {
        RegisterAllEvents();
        
        retryButton.onClick.AddListener(OnRetryClicked);
    }
    
    void RegisterAllEvents()
    {
        agent.onAgentError.AddListener(OnAgentError);
        agent.onRequestError.AddListener(OnRequestError);
        agent.onToolError.AddListener(OnToolError);
        agent.onAudioError.AddListener(OnAudioError);
        agent.onRequestCompleted.AddListener(OnRequestCompleted);
        
        Debug.Log("✓ Error handling events registered");
    }
    
    // Error Events
    
    void OnAgentError(string error)
    {
        HandleError("Agent", error);
    }
    
    void OnRequestError(string error)
    {
        HandleError("Request", error);
        
        if (ShouldRetry(error))
        {
            RetryRequest(error).Forget();
        }
    }
    
    void OnToolError(string toolName, string error)
    {
        HandleError($"Tool:{toolName}", error);
    }
    
    void OnAudioError(string error)
    {
        HandleError("Audio", error);
    }
    
    void OnRequestCompleted()
    {
        consecutiveErrors = 0;
    }
    
    // Error Handling
    
    void HandleError(string source, string error)
    {
        consecutiveErrors++;
        
        if (enableLogging)
        {
            LogError(source, error);
        }
        
        Debug.LogError($"[{source}] {error}");
        
        ErrorCategory category = CategorizeError(error);
        ShowErrorDialog(category, error);
        
        // Check for recovery need
        if (consecutiveErrors >= 3)
        {
            Debug.LogError("Multiple consecutive errors - attempting recovery");
            AttemptRecovery().Forget();
        }
    }
    
    ErrorCategory CategorizeError(string error)
    {
        if (error.Contains("api_key")) return ErrorCategory.Authentication;
        if (error.Contains("network") || error.Contains("timeout")) return ErrorCategory.Network;
        if (error.Contains("rate_limit")) return ErrorCategory.RateLimit;
        if (error.Contains("invalid")) return ErrorCategory.Validation;
        return ErrorCategory.Unknown;
    }
    
    void LogError(string source, string error)
    {
        errorLogs.Add(new ErrorLog
        {
            Timestamp = DateTime.Now,
            Source = source,
            Error = error,
            StackTrace = System.Environment.StackTrace
        });
    }
    
    // Retry Logic
    
    bool ShouldRetry(string error)
    {
        return error.Contains("timeout") ||
               error.Contains("network") ||
               error.Contains("rate_limit");
    }
    
    async UniTaskVoid RetryRequest(string error)
    {
        string requestId = agent.CurrentRequestId;
        
        if (!retryCount.ContainsKey(requestId))
        {
            retryCount[requestId] = 0;
        }
        
        retryCount[requestId]++;
        int attempt = retryCount[requestId];
        
        if (attempt > maxRetries)
        {
            Debug.LogError($"❌ Max retries ({maxRetries}) reached");
            return;
        }
        
        float delay = baseRetryDelay * Mathf.Pow(2, attempt - 1);
        
        Debug.Log($"🔄 Retrying in {delay}s (attempt {attempt}/{maxRetries})");
        
        await UniTask.Delay(TimeSpan.FromSeconds(delay));
        
        try
        {
            await agent.RetryLastRequestAsync();
            retryCount.Remove(requestId);
            Debug.Log("✓ Retry successful");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Retry failed: {ex.Message}");
        }
    }
    
    // UI
    
    void ShowErrorDialog(ErrorCategory category, string error)
    {
        errorTitle.text = GetErrorTitle(category);
        errorMessage.text = GetErrorMessage(category, error);
        retryButton.gameObject.SetActive(category == ErrorCategory.Network || category == ErrorCategory.RateLimit);
        
        errorDialog.SetActive(true);
    }
    
    string GetErrorTitle(ErrorCategory category)
    {
        return category switch
        {
            ErrorCategory.Authentication => "Authentication Error",
            ErrorCategory.Network => "Network Error",
            ErrorCategory.RateLimit => "Rate Limit",
            ErrorCategory.Validation => "Invalid Input",
            _ => "Error"
        };
    }
    
    string GetErrorMessage(ErrorCategory category, string error)
    {
        return category switch
        {
            ErrorCategory.Authentication => "Please check your API key in settings.",
            ErrorCategory.Network => "Please check your internet connection and try again.",
            ErrorCategory.RateLimit => "Too many requests. Please wait a moment.",
            ErrorCategory.Validation => error,
            _ => "An unexpected error occurred. Please try again."
        };
    }
    
    async void OnRetryClicked()
    {
        errorDialog.SetActive(false);
        
        try
        {
            await agent.RetryLastRequestAsync();
        }
        catch (Exception ex)
        {
            Debug.LogError($"Manual retry failed: {ex.Message}");
        }
    }
    
    // Recovery
    
    async UniTaskVoid AttemptRecovery()
    {
        Debug.Log("🔄 Attempting recovery...");
        
        agent.Stop();
        await UniTask.Delay(TimeSpan.FromSeconds(2));
        
        agent.Initialize();
        await UniTask.Delay(TimeSpan.FromSeconds(1));
        
        agent.Start();
        
        consecutiveErrors = 0;
        
        Debug.Log("✓ Recovery complete");
    }
    
    void OnDestroy()
    {
        if (agent != null)
        {
            agent.onAgentError.RemoveListener(OnAgentError);
            agent.onRequestError.RemoveListener(OnRequestError);
            agent.onToolError.RemoveListener(OnToolError);
            agent.onAudioError.RemoveListener(OnAudioError);
            agent.onRequestCompleted.RemoveListener(OnRequestCompleted);
        }
    }
}
```

## Next Steps

- [Agent Lifecycle Hooks](agent-hooks.md)
- [Text Delta Events](text-delta.md)
- [Tool Call Events](tool-calls.md)
- [Audio Events](audio.md)
- [Status Events](status.md)
