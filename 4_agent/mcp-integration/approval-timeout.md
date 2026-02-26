---
icon: clock
---

# Approval Timeout

Handle timeout scenarios for approval requests.

## Overview

Approval Timeout management allows you to:

- Set timeout durations for approval requests
- Handle expired approvals
- Implement auto-responses
- Queue timeout actions
- Notify users of timeouts

## Basic Setup

### Configure Timeout

```csharp
var mcpController = agent.MCPController;

// Set default timeout (in seconds)
mcpController.ApprovalTimeout = 30; // 30 seconds

// Listen for timeout events
mcpController.onApprovalTimeout.AddListener(HandleApprovalTimeout);
```

## Timeout Configuration

### Per-Tool Timeout

```csharp
public class TimeoutConfiguration : MonoBehaviour
{
    private Dictionary<string, int> toolTimeouts = new()
    {
        { "read_file", 10 },        // Quick approval needed
        { "write_file", 30 },       // Standard timeout
        { "delete_file", 60 },      // More time for dangerous ops
        { "execute_command", 45 }
    };
    
    public int GetTimeoutForTool(string toolName)
    {
        return toolTimeouts.TryGetValue(toolName, out int timeout) 
            ? timeout 
            : 30; // Default 30 seconds
    }
}
```

### Dynamic Timeout

```csharp
public int CalculateTimeout(ApprovalRequest request)
{
    // Base timeout
    int timeout = 30;
    
    // Increase for complex operations
    if (request.Arguments != null && request.Arguments.Count > 5)
        timeout += 10;
    
    // Increase for sensitive operations
    if (IsSensitiveOperation(request.ToolName))
        timeout += 20;
    
    // Decrease for read-only operations
    if (IsReadOnlyOperation(request.ToolName))
        timeout -= 10;
    
    return Math.Max(10, Math.Min(timeout, 120)); // 10-120 seconds
}

bool IsSensitiveOperation(string toolName)
{
    string[] sensitive = { "delete", "remove", "execute", "format" };
    return sensitive.Any(op => toolName.Contains(op));
}

bool IsReadOnlyOperation(string toolName)
{
    string[] readOnly = { "read", "get", "list", "search" };
    return readOnly.Any(op => toolName.StartsWith(op));
}
```

## Timeout Handling

### Handle Timeout Event

```csharp
void HandleApprovalTimeout(ApprovalRequest request)
{
    Debug.LogWarning($"⏱️ Approval timeout: {request.ToolName}");
    
    // Get default action for this tool
    var action = GetTimeoutAction(request.ToolName);
    
    switch (action)
    {
        case TimeoutAction.Deny:
            DenyExpiredRequest(request);
            break;
        
        case TimeoutAction.Approve:
            ApproveExpiredRequest(request);
            break;
        
        case TimeoutAction.Retry:
            RetryApprovalRequest(request);
            break;
    }
}

public enum TimeoutAction
{
    Deny,
    Approve,
    Retry
}
```

### Timeout Actions

```csharp
public class TimeoutActionManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Dictionary<string, TimeoutAction> actions = new()
    {
        // Safe operations: approve on timeout
        { "read_file", TimeoutAction.Approve },
        { "list_files", TimeoutAction.Approve },
        { "get_status", TimeoutAction.Approve },
        
        // Dangerous operations: deny on timeout
        { "delete_file", TimeoutAction.Deny },
        { "write_file", TimeoutAction.Deny },
        { "execute_command", TimeoutAction.Deny },
        
        // Retry for uncertain operations
        { "unknown", TimeoutAction.Retry }
    };
    
    public TimeoutAction GetAction(string toolName)
    {
        return actions.TryGetValue(toolName, out var action)
            ? action
            : TimeoutAction.Deny; // Default: deny
    }
    
    public async void ExecuteTimeoutAction(ApprovalRequest request)
    {
        var action = GetAction(request.ToolName);
        
        switch (action)
        {
            case TimeoutAction.Approve:
                Debug.Log($"✓ Auto-approved after timeout: {request.ToolName}");
                await agent.MCPController.ApproveToolCallAsync(request.Id);
                break;
            
            case TimeoutAction.Deny:
                Debug.Log($"✗ Auto-denied after timeout: {request.ToolName}");
                await agent.MCPController.DenyToolCallAsync(request.Id);
                break;
            
            case TimeoutAction.Retry:
                Debug.Log($"🔄 Retrying approval: {request.ToolName}");
                await RetryApproval(request);
                break;
        }
    }
    
    async UniTask RetryApproval(ApprovalRequest request)
    {
        // Retry with extended timeout
        request.TimeoutSeconds += 15;
        await agent.MCPController.RequestApprovalAsync(request);
    }
}
```

## Timeout UI

### Show Timeout Warning

```csharp
public class TimeoutWarningUI : MonoBehaviour
{
    [SerializeField] private GameObject warningPanel;
    [SerializeField] private TMP_Text timerText;
    [SerializeField] private Image timerBar;
    
    private float remainingTime;
    private float totalTime;
    private bool isActive;
    
    public void StartTimer(float seconds)
    {
        totalTime = seconds;
        remainingTime = seconds;
        isActive = true;
        warningPanel.SetActive(true);
    }
    
    void Update()
    {
        if (!isActive) return;
        
        remainingTime -= Time.deltaTime;
        
        // Update UI
        timerText.text = $"{Mathf.CeilToInt(remainingTime)}s";
        timerBar.fillAmount = remainingTime / totalTime;
        
        // Change color as time runs out
        if (remainingTime < 10)
            timerBar.color = Color.red;
        else if (remainingTime < 20)
            timerBar.color = Color.yellow;
        
        // Timeout reached
        if (remainingTime <= 0)
        {
            OnTimeout();
        }
    }
    
    void OnTimeout()
    {
        isActive = false;
        warningPanel.SetActive(false);
    }
    
    public void Stop()
    {
        isActive = false;
        warningPanel.SetActive(false);
    }
}
```

### Approval Dialog with Timeout

```csharp
public class TimedApprovalDialog : MonoBehaviour
{
    [SerializeField] private TMP_Text titleText;
    [SerializeField] private TMP_Text messageText;
    [SerializeField] private TMP_Text timerText;
    [SerializeField] private Button approveButton;
    [SerializeField] private Button denyButton;
    [SerializeField] private Image timerBar;
    
    private UniTaskCompletionSource<bool?> completionSource;
    private float remainingTime;
    private float totalTime;
    private bool isWaiting;
    
    void Start()
    {
        approveButton.onClick.AddListener(() => Respond(true));
        denyButton.onClick.AddListener(() => Respond(false));
    }
    
    void Update()
    {
        if (!isWaiting) return;
        
        remainingTime -= Time.deltaTime;
        
        // Update timer display
        timerText.text = $"Time remaining: {Mathf.CeilToInt(remainingTime)}s";
        timerBar.fillAmount = remainingTime / totalTime;
        
        // Color warning
        if (remainingTime < 5)
            timerBar.color = Color.red;
        else if (remainingTime < 10)
            timerBar.color = Color.yellow;
        else
            timerBar.color = Color.green;
        
        // Timeout
        if (remainingTime <= 0)
        {
            Timeout();
        }
    }
    
    public void Configure(ApprovalRequest request, int timeoutSeconds)
    {
        titleText.text = "Approve Tool Execution?";
        messageText.text = $@"
<b>Tool:</b> {request.ToolName}
<b>Server:</b> {request.ServerName}

<b>This request will timeout in {timeoutSeconds} seconds</b>
";
        
        totalTime = timeoutSeconds;
        remainingTime = timeoutSeconds;
    }
    
    public UniTask<bool?> WaitForResponse()
    {
        completionSource = new UniTaskCompletionSource<bool?>();
        isWaiting = true;
        return completionSource.Task;
    }
    
    void Respond(bool approved)
    {
        isWaiting = false;
        completionSource?.TrySetResult(approved);
    }
    
    void Timeout()
    {
        isWaiting = false;
        completionSource?.TrySetResult(null); // null = timeout
    }
}
```

## Retry Logic

### Automatic Retry

```csharp
public class ApprovalRetryHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int maxRetries = 3;
    [SerializeField] private float retryDelaySeconds = 5;
    
    private Dictionary<string, int> retryCount = new();
    
    async void HandleTimeout(ApprovalRequest request)
    {
        string key = request.Id;
        int count = retryCount.GetValueOrDefault(key, 0);
        
        if (count < maxRetries)
        {
            retryCount[key] = count + 1;
            
            Debug.Log($"🔄 Retry {count + 1}/{maxRetries} for: {request.ToolName}");
            
            // Wait before retry
            await UniTask.Delay((int)(retryDelaySeconds * 1000));
            
            // Retry with extended timeout
            request.TimeoutSeconds += 10;
            await agent.MCPController.RequestApprovalAsync(request);
        }
        else
        {
            Debug.Log($"✗ Max retries reached, denying: {request.ToolName}");
            await agent.MCPController.DenyToolCallAsync(request.Id);
            
            retryCount.Remove(key);
        }
    }
}
```

### User-Prompted Retry

```csharp
public async UniTask<bool> ShowRetryDialog(ApprovalRequest request, int attemptNumber)
{
    var dialog = Instantiate(retryDialogPrefab);
    var ui = dialog.GetComponent<RetryDialogUI>();
    
    ui.SetMessage($@"
Approval request timed out.

Tool: {request.ToolName}
Attempt: {attemptNumber}/{maxRetries}

Would you like to retry?
");
    
    bool retry = await ui.WaitForResponse();
    
    Destroy(dialog);
    
    return retry;
}
```

## Notification System

### Timeout Notifications

```csharp
public class TimeoutNotification : MonoBehaviour
{
    [SerializeField] private GameObject notificationPrefab;
    [SerializeField] private Transform notificationContainer;
    
    public void ShowTimeoutNotification(ApprovalRequest request)
    {
        var notification = Instantiate(notificationPrefab, notificationContainer);
        var text = notification.GetComponentInChildren<TMP_Text>();
        
        text.text = $"⏱️ Approval timeout: {request.ToolName}";
        
        // Auto-destroy after 3 seconds
        Destroy(notification, 3f);
    }
    
    public void ShowAutoResponseNotification(ApprovalRequest request, bool approved)
    {
        var notification = Instantiate(notificationPrefab, notificationContainer);
        var text = notification.GetComponentInChildren<TMP_Text>();
        
        string action = approved ? "approved" : "denied";
        text.text = $"Automatically {action}: {request.ToolName}";
        
        Destroy(notification, 3f);
    }
}
```

## Timeout Analytics

### Track Timeouts

```csharp
public class TimeoutAnalytics : MonoBehaviour
{
    private List<TimeoutRecord> timeouts = new();
    
    public void RecordTimeout(ApprovalRequest request, TimeoutAction action)
    {
        timeouts.Add(new TimeoutRecord
        {
            Timestamp = DateTime.Now,
            ToolName = request.ToolName,
            ServerName = request.ServerName,
            TimeoutSeconds = request.TimeoutSeconds,
            Action = action
        });
    }
    
    public Dictionary<string, int> GetTimeoutStatistics()
    {
        return timeouts
            .GroupBy(t => t.ToolName)
            .ToDictionary(g => g.Key, g => g.Count());
    }
    
    public float GetAverageTimeoutDuration()
    {
        return timeouts.Average(t => t.TimeoutSeconds);
    }
    
    public List<TimeoutRecord> GetRecentTimeouts(int count = 10)
    {
        return timeouts.TakeLast(count).ToList();
    }
}

[System.Serializable]
public class TimeoutRecord
{
    public DateTime Timestamp;
    public string ToolName;
    public string ServerName;
    public int TimeoutSeconds;
    public TimeoutAction Action;
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class TimeoutManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject timedDialogPrefab;
    [SerializeField] private Transform dialogContainer;
    
    [Header("Settings")]
    [SerializeField] private int defaultTimeout = 30;
    [SerializeField] private int maxRetries = 3;
    
    private TimeoutConfiguration configuration;
    private TimeoutActionManager actionManager;
    private TimeoutAnalytics analytics;
    private Dictionary<string, int> retryCount = new();
    
    void Start()
    {
        configuration = GetComponent<TimeoutConfiguration>();
        actionManager = GetComponent<TimeoutActionManager>();
        analytics = GetComponent<TimeoutAnalytics>();
        
        SetupTimeoutSystem();
    }
    
    void SetupTimeoutSystem()
    {
        var mcpController = agent.MCPController;
        
        // Set default timeout
        mcpController.ApprovalTimeout = defaultTimeout;
        
        // Listen for events
        mcpController.onApprovalRequired.AddListener(HandleApprovalWithTimeout);
        mcpController.onApprovalTimeout.AddListener(HandleTimeout);
        
        Debug.Log("✓ Timeout system ready");
    }
    
    async void HandleApprovalWithTimeout(ApprovalRequest request)
    {
        // Determine timeout for this tool
        int timeout = configuration.GetTimeoutForTool(request.ToolName);
        request.TimeoutSeconds = timeout;
        
        Debug.Log($"🔒 Approval required: {request.ToolName} (timeout: {timeout}s)");
        
        // Show timed dialog
        var result = await ShowTimedApprovalDialog(request, timeout);
        
        if (result.HasValue)
        {
            // User responded in time
            if (result.Value)
            {
                await agent.MCPController.ApproveToolCallAsync(request.Id);
            }
            else
            {
                await agent.MCPController.DenyToolCallAsync(request.Id);
            }
        }
        // If null, timeout will be handled by HandleTimeout
    }
    
    async UniTask<bool?> ShowTimedApprovalDialog(ApprovalRequest request, int timeoutSeconds)
    {
        var dialog = Instantiate(timedDialogPrefab, dialogContainer);
        var ui = dialog.GetComponent<TimedApprovalDialog>();
        
        ui.Configure(request, timeoutSeconds);
        
        bool? result = await ui.WaitForResponse();
        
        Destroy(dialog);
        
        return result;
    }
    
    async void HandleTimeout(ApprovalRequest request)
    {
        Debug.LogWarning($"⏱️ Approval timeout: {request.ToolName}");
        
        // Check retry count
        string key = request.Id;
        int count = retryCount.GetValueOrDefault(key, 0);
        
        if (count < maxRetries)
        {
            // Retry
            retryCount[key] = count + 1;
            Debug.Log($"🔄 Retry {count + 1}/{maxRetries}");
            
            await UniTask.Delay(2000);
            
            // Extend timeout for retry
            request.TimeoutSeconds += 10;
            await agent.MCPController.RequestApprovalAsync(request);
        }
        else
        {
            // Execute timeout action
            var action = actionManager.GetAction(request.ToolName);
            
            Debug.Log($"Executing timeout action: {action}");
            actionManager.ExecuteTimeoutAction(request);
            
            // Record analytics
            analytics.RecordTimeout(request, action);
            
            // Clean up
            retryCount.Remove(key);
        }
    }
}
```

## Next Steps

- [Access Token Service](access-token-service.md)
- [Approval Handlers](approval-handlers.md)
- [OAuth Providers](oauth-providers.md)
