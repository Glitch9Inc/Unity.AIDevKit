# Tool Call Events

Monitor and respond to tool execution events.

## Overview

Tool Call Events provide:

- Tool execution monitoring
- Call lifecycle tracking
- Result handling
- Error detection
- Performance metrics

## Basic Setup

### Subscribe to Tool Events

```csharp
public class ToolEventListener : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.onToolCallRequested.AddListener(OnToolCallRequested);
        agent.onToolCallExecuting.AddListener(OnToolCallExecuting);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
        agent.onToolCallError.AddListener(OnToolCallError);
    }
    
    void OnToolCallRequested(ToolCall call)
    {
        Debug.Log($"🔧 Tool requested: {call.FunctionName}");
    }
    
    void OnToolCallExecuting(ToolCall call)
    {
        Debug.Log($"⚙️ Executing: {call.FunctionName}");
    }
    
    void OnToolCallCompleted(ToolCall call, string result)
    {
        Debug.Log($"✓ Tool completed: {call.FunctionName}");
        Debug.Log($"Result: {result}");
    }
    
    void OnToolCallError(ToolCall call, string error)
    {
        Debug.LogError($"❌ Tool error: {call.FunctionName} - {error}");
    }
}
```

## Tool Call Lifecycle

### Track Complete Lifecycle

```csharp
public class ToolCallTracker : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Dictionary<string, ToolCallInfo> activeCalls = new();
    
    void Start()
    {
        agent.onToolCallRequested.AddListener(OnToolCallRequested);
        agent.onToolCallExecuting.AddListener(OnToolCallExecuting);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
        agent.onToolCallError.AddListener(OnToolCallError);
    }
    
    void OnToolCallRequested(ToolCall call)
    {
        var info = new ToolCallInfo
        {
            Id = call.Id,
            FunctionName = call.FunctionName,
            Arguments = call.Arguments,
            RequestedAt = DateTime.Now,
            Status = ToolCallStatus.Requested
        };
        
        activeCalls[call.Id] = info;
        
        Debug.Log($"🔧 [{call.Id}] Tool requested: {call.FunctionName}");
        LogToolCall(info);
    }
    
    void OnToolCallExecuting(ToolCall call)
    {
        if (activeCalls.TryGetValue(call.Id, out var info))
        {
            info.ExecutingAt = DateTime.Now;
            info.Status = ToolCallStatus.Executing;
            
            Debug.Log($"⚙️ [{call.Id}] Executing: {call.FunctionName}");
        }
    }
    
    void OnToolCallCompleted(ToolCall call, string result)
    {
        if (activeCalls.TryGetValue(call.Id, out var info))
        {
            info.CompletedAt = DateTime.Now;
            info.Result = result;
            info.Status = ToolCallStatus.Completed;
            
            float duration = (float)(info.CompletedAt - info.ExecutingAt).TotalMilliseconds;
            
            Debug.Log($"✓ [{call.Id}] Completed: {call.FunctionName} ({duration}ms)");
            LogToolCall(info);
            
            activeCalls.Remove(call.Id);
        }
    }
    
    void OnToolCallError(ToolCall call, string error)
    {
        if (activeCalls.TryGetValue(call.Id, out var info))
        {
            info.CompletedAt = DateTime.Now;
            info.Error = error;
            info.Status = ToolCallStatus.Error;
            
            Debug.LogError($"❌ [{call.Id}] Error: {call.FunctionName} - {error}");
            LogToolCall(info);
            
            activeCalls.Remove(call.Id);
        }
    }
    
    void LogToolCall(ToolCallInfo info)
    {
        // Log to analytics or database
    }
}

public class ToolCallInfo
{
    public string Id { get; set; }
    public string FunctionName { get; set; }
    public string Arguments { get; set; }
    public DateTime RequestedAt { get; set; }
    public DateTime ExecutingAt { get; set; }
    public DateTime CompletedAt { get; set; }
    public string Result { get; set; }
    public string Error { get; set; }
    public ToolCallStatus Status { get; set; }
}

public enum ToolCallStatus
{
    Requested,
    Executing,
    Completed,
    Error
}
```

## UI Updates

### Show Tool Execution in UI

```csharp
public class ToolCallUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Transform toolCallContainer;
    [SerializeField] private GameObject toolCallPrefab;
    
    private Dictionary<string, GameObject> activeToolUIs = new();
    
    void Start()
    {
        agent.onToolCallRequested.AddListener(OnToolCallRequested);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
        agent.onToolCallError.AddListener(OnToolCallError);
    }
    
    void OnToolCallRequested(ToolCall call)
    {
        // Create UI element
        GameObject toolUI = Instantiate(toolCallPrefab, toolCallContainer);
        
        var text = toolUI.GetComponentInChildren<TMP_Text>();
        text.text = $"🔧 {call.FunctionName}...";
        
        var image = toolUI.GetComponent<Image>();
        image.color = Color.yellow;
        
        activeToolUIs[call.Id] = toolUI;
    }
    
    void OnToolCallCompleted(ToolCall call, string result)
    {
        if (activeToolUIs.TryGetValue(call.Id, out var toolUI))
        {
            var text = toolUI.GetComponentInChildren<TMP_Text>();
            text.text = $"✓ {call.FunctionName}";
            
            var image = toolUI.GetComponent<Image>();
            image.color = Color.green;
            
            // Remove after delay
            RemoveToolUI(call.Id).Forget();
        }
    }
    
    void OnToolCallError(ToolCall call, string error)
    {
        if (activeToolUIs.TryGetValue(call.Id, out var toolUI))
        {
            var text = toolUI.GetComponentInChildren<TMP_Text>();
            text.text = $"❌ {call.FunctionName}";
            
            var image = toolUI.GetComponent<Image>();
            image.color = Color.red;
            
            // Remove after delay
            RemoveToolUI(call.Id).Forget();
        }
    }
    
    async UniTaskVoid RemoveToolUI(string callId)
    {
        await UniTask.Delay(TimeSpan.FromSeconds(3));
        
        if (activeToolUIs.TryGetValue(callId, out var toolUI))
        {
            Destroy(toolUI);
            activeToolUIs.Remove(callId);
        }
    }
}
```

## Performance Monitoring

### Track Tool Performance

```csharp
public class ToolPerformanceMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Dictionary<string, List<float>> toolExecutionTimes = new();
    private Dictionary<string, int> toolCallCounts = new();
    private Dictionary<string, int> toolErrorCounts = new();
    
    void Start()
    {
        agent.onToolCallExecuting.AddListener(OnToolCallExecuting);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
        agent.onToolCallError.AddListener(OnToolCallError);
    }
    
    void OnToolCallExecuting(ToolCall call)
    {
        // Store start time
        call.Metadata["startTime"] = DateTime.Now;
    }
    
    void OnToolCallCompleted(ToolCall call, string result)
    {
        if (call.Metadata.TryGetValue("startTime", out var startTimeObj) &&
            startTimeObj is DateTime startTime)
        {
            float duration = (float)(DateTime.Now - startTime).TotalMilliseconds;
            
            // Record execution time
            if (!toolExecutionTimes.ContainsKey(call.FunctionName))
            {
                toolExecutionTimes[call.FunctionName] = new List<float>();
            }
            toolExecutionTimes[call.FunctionName].Add(duration);
            
            // Record call count
            if (!toolCallCounts.ContainsKey(call.FunctionName))
            {
                toolCallCounts[call.FunctionName] = 0;
            }
            toolCallCounts[call.FunctionName]++;
            
            Debug.Log($"⏱️ {call.FunctionName} completed in {duration}ms");
        }
    }
    
    void OnToolCallError(ToolCall call, string error)
    {
        // Record error count
        if (!toolErrorCounts.ContainsKey(call.FunctionName))
        {
            toolErrorCounts[call.FunctionName] = 0;
        }
        toolErrorCounts[call.FunctionName]++;
    }
    
    public void LogStatistics()
    {
        Debug.Log("=== Tool Performance Statistics ===");
        
        foreach (var kvp in toolExecutionTimes)
        {
            string functionName = kvp.Key;
            List<float> times = kvp.Value;
            
            float avgTime = times.Average();
            float minTime = times.Min();
            float maxTime = times.Max();
            int callCount = toolCallCounts[functionName];
            int errorCount = toolErrorCounts.ContainsKey(functionName) ? 
                toolErrorCounts[functionName] : 0;
            
            Debug.Log($"{functionName}:");
            Debug.Log($"  Calls: {callCount}");
            Debug.Log($"  Errors: {errorCount}");
            Debug.Log($"  Avg time: {avgTime:F2}ms");
            Debug.Log($"  Min time: {minTime:F2}ms");
            Debug.Log($"  Max time: {maxTime:F2}ms");
        }
    }
}
```

## Tool Call Filtering

### Filter Specific Tools

```csharp
public class ToolCallFilter : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private string[] toolsToMonitor = { "web_search", "file_search" };
    
    void Start()
    {
        agent.onToolCallRequested.AddListener(OnToolCallRequested);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
    }
    
    void OnToolCallRequested(ToolCall call)
    {
        if (ShouldMonitor(call.FunctionName))
        {
            Debug.Log($"📊 Monitoring tool: {call.FunctionName}");
            Debug.Log($"Arguments: {call.Arguments}");
        }
    }
    
    void OnToolCallCompleted(ToolCall call, string result)
    {
        if (ShouldMonitor(call.FunctionName))
        {
            Debug.Log($"📊 Tool result: {call.FunctionName}");
            Debug.Log($"Result: {result.Substring(0, Math.Min(100, result.Length))}...");
        }
    }
    
    bool ShouldMonitor(string functionName)
    {
        return toolsToMonitor.Contains(functionName);
    }
}
```

## Approval Workflow

### Request User Approval

```csharp
public class ToolApprovalSystem : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject approvalDialog;
    [SerializeField] private TMP_Text approvalText;
    
    private ToolCall pendingCall;
    
    void Start()
    {
        agent.onToolCallRequested.AddListener(OnToolCallRequested);
    }
    
    void OnToolCallRequested(ToolCall call)
    {
        if (RequiresApproval(call))
        {
            RequestApproval(call);
        }
    }
    
    bool RequiresApproval(ToolCall call)
    {
        // Require approval for sensitive operations
        string[] sensitiveTools = { "delete_file", "execute_code", "send_email" };
        return sensitiveTools.Contains(call.FunctionName);
    }
    
    void RequestApproval(ToolCall call)
    {
        pendingCall = call;
        
        approvalText.text = $"Allow {call.FunctionName}?\n\n{call.Arguments}";
        approvalDialog.SetActive(true);
        
        Debug.Log($"⚠️ Approval required: {call.FunctionName}");
    }
    
    public void ApproveToolCall()
    {
        if (pendingCall != null)
        {
            Debug.Log($"✓ Approved: {pendingCall.FunctionName}");
            
            // Execute tool
            agent.ToolController.ExecuteToolCallAsync(pendingCall).Forget();
            
            approvalDialog.SetActive(false);
            pendingCall = null;
        }
    }
    
    public void DenyToolCall()
    {
        if (pendingCall != null)
        {
            Debug.Log($"❌ Denied: {pendingCall.FunctionName}");
            
            // Send error to agent
            agent.ToolController.SubmitToolOutputAsync(
                pendingCall.Id,
                "User denied tool execution"
            ).Forget();
            
            approvalDialog.SetActive(false);
            pendingCall = null;
        }
    }
}
```

## Logging and Analytics

### Comprehensive Logging

```csharp
public class ToolCallLogger : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private string logPath = "ToolCalls";
    
    private List<ToolCallLog> logs = new();
    
    void Start()
    {
        agent.onToolCallRequested.AddListener(OnToolCallRequested);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
        agent.onToolCallError.AddListener(OnToolCallError);
    }
    
    void OnToolCallRequested(ToolCall call)
    {
        var log = new ToolCallLog
        {
            Id = call.Id,
            FunctionName = call.FunctionName,
            Arguments = call.Arguments,
            Timestamp = DateTime.Now,
            Status = "Requested"
        };
        
        logs.Add(log);
    }
    
    void OnToolCallCompleted(ToolCall call, string result)
    {
        var log = logs.Find(l => l.Id == call.Id);
        if (log != null)
        {
            log.Result = result;
            log.CompletedAt = DateTime.Now;
            log.Duration = (log.CompletedAt - log.Timestamp).TotalMilliseconds;
            log.Status = "Completed";
        }
    }
    
    void OnToolCallError(ToolCall call, string error)
    {
        var log = logs.Find(l => l.Id == call.Id);
        if (log != null)
        {
            log.Error = error;
            log.CompletedAt = DateTime.Now;
            log.Duration = (log.CompletedAt - log.Timestamp).TotalMilliseconds;
            log.Status = "Error";
        }
    }
    
    public void SaveLogs()
    {
        string json = JsonUtility.ToJson(new ToolCallLogCollection { logs = logs }, true);
        
        string filePath = Path.Combine(
            Application.persistentDataPath,
            logPath,
            $"tool_calls_{DateTime.Now:yyyyMMdd_HHmmss}.json"
        );
        
        Directory.CreateDirectory(Path.GetDirectoryName(filePath));
        File.WriteAllText(filePath, json);
        
        Debug.Log($"💾 Tool call logs saved: {filePath}");
    }
    
    void OnApplicationQuit()
    {
        SaveLogs();
    }
}

[Serializable]
public class ToolCallLog
{
    public string Id;
    public string FunctionName;
    public string Arguments;
    public DateTime Timestamp;
    public DateTime CompletedAt;
    public double Duration;
    public string Result;
    public string Error;
    public string Status;
}

[Serializable]
public class ToolCallLogCollection
{
    public List<ToolCallLog> logs;
}
```

## Error Recovery

### Automatic Retry

```csharp
public class ToolCallRetry : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int maxRetries = 3;
    
    private Dictionary<string, int> retryCount = new();
    
    void Start()
    {
        agent.onToolCallError.AddListener(OnToolCallError);
    }
    
    void OnToolCallError(ToolCall call, string error)
    {
        if (ShouldRetry(call, error))
        {
            RetryToolCall(call).Forget();
        }
        else
        {
            Debug.LogError($"❌ Tool call failed after {maxRetries} retries: {call.FunctionName}");
        }
    }
    
    bool ShouldRetry(ToolCall call, string error)
    {
        // Get retry count
        if (!retryCount.ContainsKey(call.Id))
        {
            retryCount[call.Id] = 0;
        }
        
        if (retryCount[call.Id] >= maxRetries)
        {
            return false;
        }
        
        // Only retry on transient errors
        return error.Contains("timeout") ||
               error.Contains("network") ||
               error.Contains("rate_limit");
    }
    
    async UniTaskVoid RetryToolCall(ToolCall call)
    {
        retryCount[call.Id]++;
        
        int delay = retryCount[call.Id] * 2;
        Debug.Log($"🔄 Retrying {call.FunctionName} in {delay}s (attempt {retryCount[call.Id]}/{maxRetries})");
        
        await UniTask.Delay(TimeSpan.FromSeconds(delay));
        
        await agent.ToolController.ExecuteToolCallAsync(call);
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class ToolCallEventManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Transform toolCallUIContainer;
    [SerializeField] private GameObject toolCallItemPrefab;
    
    [Header("Settings")]
    [SerializeField] private bool enableLogging = true;
    [SerializeField] private bool enablePerformanceMonitoring = true;
    
    private Dictionary<string, GameObject> activeToolUIs = new();
    private Dictionary<string, float> executionTimes = new();
    private List<ToolCallLog> callLogs = new();
    
    void Start()
    {
        RegisterEvents();
    }
    
    void RegisterEvents()
    {
        agent.onToolCallRequested.AddListener(OnToolCallRequested);
        agent.onToolCallExecuting.AddListener(OnToolCallExecuting);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
        agent.onToolCallError.AddListener(OnToolCallError);
        
        Debug.Log("✓ Tool call events registered");
    }
    
    void OnToolCallRequested(ToolCall call)
    {
        if (enableLogging)
        {
            Debug.Log($"🔧 Tool requested: {call.FunctionName}");
            Debug.Log($"Arguments: {call.Arguments}");
        }
        
        // Create UI
        CreateToolCallUI(call);
        
        // Log
        LogToolCall(call, "Requested");
    }
    
    void OnToolCallExecuting(ToolCall call)
    {
        if (enableLogging)
            Debug.Log($"⚙️ Executing: {call.FunctionName}");
        
        // Record start time
        if (enablePerformanceMonitoring)
        {
            executionTimes[call.Id] = Time.realtimeSinceStartup;
        }
        
        // Update UI
        UpdateToolCallUI(call, "Executing", Color.yellow);
    }
    
    void OnToolCallCompleted(ToolCall call, string result)
    {
        // Calculate duration
        float duration = 0;
        if (enablePerformanceMonitoring && executionTimes.ContainsKey(call.Id))
        {
            duration = (Time.realtimeSinceStartup - executionTimes[call.Id]) * 1000f;
            executionTimes.Remove(call.Id);
        }
        
        if (enableLogging)
        {
            Debug.Log($"✓ Tool completed: {call.FunctionName}");
            Debug.Log($"Duration: {duration:F2}ms");
            Debug.Log($"Result: {result.Substring(0, Math.Min(100, result.Length))}...");
        }
        
        // Update UI
        UpdateToolCallUI(call, "Completed", Color.green);
        RemoveToolCallUI(call.Id).Forget();
        
        // Log
        LogToolCall(call, "Completed", result, duration);
    }
    
    void OnToolCallError(ToolCall call, string error)
    {
        if (enableLogging)
            Debug.LogError($"❌ Tool error: {call.FunctionName} - {error}");
        
        // Update UI
        UpdateToolCallUI(call, "Error", Color.red);
        RemoveToolCallUI(call.Id).Forget();
        
        // Log
        LogToolCall(call, "Error", error: error);
        
        // Cleanup
        if (executionTimes.ContainsKey(call.Id))
        {
            executionTimes.Remove(call.Id);
        }
    }
    
    void CreateToolCallUI(ToolCall call)
    {
        GameObject toolUI = Instantiate(toolCallItemPrefab, toolCallUIContainer);
        
        var text = toolUI.GetComponentInChildren<TMP_Text>();
        text.text = $"🔧 {call.FunctionName}";
        
        activeToolUIs[call.Id] = toolUI;
    }
    
    void UpdateToolCallUI(ToolCall call, string status, Color color)
    {
        if (activeToolUIs.TryGetValue(call.Id, out var toolUI))
        {
            var text = toolUI.GetComponentInChildren<TMP_Text>();
            text.text = $"{GetStatusIcon(status)} {call.FunctionName}";
            
            var image = toolUI.GetComponent<Image>();
            image.color = color;
        }
    }
    
    async UniTaskVoid RemoveToolCallUI(string callId)
    {
        await UniTask.Delay(TimeSpan.FromSeconds(3));
        
        if (activeToolUIs.TryGetValue(callId, out var toolUI))
        {
            Destroy(toolUI);
            activeToolUIs.Remove(callId);
        }
    }
    
    string GetStatusIcon(string status)
    {
        return status switch
        {
            "Executing" => "⚙️",
            "Completed" => "✓",
            "Error" => "❌",
            _ => "🔧"
        };
    }
    
    void LogToolCall(ToolCall call, string status, string result = null, float duration = 0, string error = null)
    {
        var log = new ToolCallLog
        {
            Id = call.Id,
            FunctionName = call.FunctionName,
            Arguments = call.Arguments,
            Timestamp = DateTime.Now,
            Status = status,
            Result = result,
            Duration = duration,
            Error = error
        };
        
        callLogs.Add(log);
    }
    
    void OnDestroy()
    {
        if (agent != null)
        {
            agent.onToolCallRequested.RemoveListener(OnToolCallRequested);
            agent.onToolCallExecuting.RemoveListener(OnToolCallExecuting);
            agent.onToolCallCompleted.RemoveListener(OnToolCallCompleted);
            agent.onToolCallError.RemoveListener(OnToolCallError);
        }
    }
}
```

## Next Steps

- [Agent Lifecycle Hooks](agent-hooks.md)
- [Text Delta Events](text-delta.md)
- [Audio Events](audio.md)
- [Status Events](status.md)
- [Error Handling Events](error-handling.md)
