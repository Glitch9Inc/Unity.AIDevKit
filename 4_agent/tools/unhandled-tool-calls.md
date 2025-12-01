# Unhandled Tool Calls

Handle tool calls that don't have registered executors.

## Overview

When an agent calls a tool that:

- Doesn't have a registered executor
- Requires manual approval
- Needs user intervention

You can handle these "unhandled" tool calls.

## Basic Handling

### Event Listener

```csharp
void Start()
{
    agent.onUnhandledToolCall.AddListener(OnUnhandledToolCall);
}

void OnUnhandledToolCall(ToolCall toolCall)
{
    Debug.LogWarning($"Unhandled tool: {toolCall.Function.Name}");
    Debug.Log($"Arguments: {toolCall.Function.Arguments}");
    
    // Show UI to user
    ShowToolApprovalDialog(toolCall);
}
```

## Manual Tool Execution

### Execute and Submit

```csharp
async void OnUnhandledToolCall(ToolCall toolCall)
{
    // Execute manually
    string output = await ExecuteManually(toolCall);
    
    // Submit result
    await agent.SubmitToolOutputAsync(toolCall.Id, output);
}

async UniTask<string> ExecuteManually(ToolCall toolCall)
{
    switch (toolCall.Function.Name)
    {
        case "purchase_item":
            return await HandlePurchase(toolCall);
            
        case "send_message":
            return await HandleMessage(toolCall);
            
        default:
            return "Tool not supported";
    }
}
```

## User Approval Flow

### Approval Dialog

```csharp
public class ToolApprovalDialog : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject dialogPanel;
    [SerializeField] private TMP_Text toolNameText;
    [SerializeField] private TMP_Text toolDescriptionText;
    [SerializeField] private Button approveButton;
    [SerializeField] private Button denyButton;
    
    private ToolCall currentToolCall;
    
    void Start()
    {
        agent.onUnhandledToolCall.AddListener(OnUnhandledToolCall);
        
        approveButton.onClick.AddListener(OnApprove);
        denyButton.onClick.AddListener(OnDeny);
    }
    
    void OnUnhandledToolCall(ToolCall toolCall)
    {
        currentToolCall = toolCall;
        
        // Parse arguments
        var args = ParseArguments(toolCall);
        
        // Show dialog
        toolNameText.text = GetFriendlyName(toolCall.Function.Name);
        toolDescriptionText.text = GetDescription(toolCall.Function.Name, args);
        dialogPanel.SetActive(true);
    }
    
    async void OnApprove()
    {
        dialogPanel.SetActive(false);
        
        try
        {
            // Execute tool
            string output = await ExecuteTool(currentToolCall);
            
            // Submit output
            await agent.SubmitToolOutputAsync(currentToolCall.Id, output);
            
            ShowMessage("Action completed");
        }
        catch (Exception ex)
        {
            Debug.LogException(ex);
            await agent.SubmitToolOutputAsync(currentToolCall.Id, $"Error: {ex.Message}");
        }
    }
    
    void OnDeny()
    {
        dialogPanel.SetActive(false);
        
        // Submit rejection
        agent.SubmitToolOutputAsync(currentToolCall.Id, "User denied the action");
    }
    
    Dictionary<string, object> ParseArguments(ToolCall toolCall)
    {
        return JsonUtility.FromJson<Dictionary<string, object>>(
            toolCall.Function.Arguments
        );
    }
    
    string GetFriendlyName(string toolName)
    {
        return toolName switch
        {
            "purchase_item" => "Purchase Item",
            "send_message" => "Send Message",
            "delete_data" => "Delete Data",
            _ => toolName
        };
    }
    
    string GetDescription(string toolName, Dictionary<string, object> args)
    {
        return toolName switch
        {
            "purchase_item" => $"Purchase {args["item"]} for ${args["price"]}?",
            "send_message" => $"Send message to {args["recipient"]}?",
            "delete_data" => $"Delete {args["count"]} items?",
            _ => "Approve this action?"
        };
    }
    
    async UniTask<string> ExecuteTool(ToolCall toolCall)
    {
        return toolCall.Function.Name switch
        {
            "purchase_item" => await PurchaseItem(toolCall),
            "send_message" => await SendMessage(toolCall),
            "delete_data" => await DeleteData(toolCall),
            _ => "Unknown tool"
        };
    }
    
    async UniTask<string> PurchaseItem(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<PurchaseArgs>(toolCall.Function.Arguments);
        
        // Purchase logic
        await UniTask.Delay(500);
        
        return $"Purchased {args.item} for ${args.price}";
    }
    
    async UniTask<string> SendMessage(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<MessageArgs>(toolCall.Function.Arguments);
        
        // Send logic
        await UniTask.Delay(300);
        
        return $"Message sent to {args.recipient}";
    }
    
    async UniTask<string> DeleteData(ToolCall toolCall)
    {
        var args = JsonUtility.FromJson<DeleteArgs>(toolCall.Function.Arguments);
        
        // Delete logic
        await UniTask.Delay(200);
        
        return $"Deleted {args.count} items";
    }
    
    [System.Serializable]
    class PurchaseArgs
    {
        public string item;
        public float price;
    }
    
    [System.Serializable]
    class MessageArgs
    {
        public string recipient;
        public string message;
    }
    
    [System.Serializable]
    class DeleteArgs
    {
        public int count;
    }
}
```

## Dangerous Operations

### Require Confirmation

```csharp
public class DangerousToolHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private readonly HashSet<string> dangerousTools = new()
    {
        "delete_account",
        "reset_data",
        "format_drive",
        "send_email_all"
    };
    
    void Start()
    {
        agent.onUnhandledToolCall.AddListener(OnUnhandledToolCall);
    }
    
    async void OnUnhandledToolCall(ToolCall toolCall)
    {
        if (dangerousTools.Contains(toolCall.Function.Name))
        {
            bool confirmed = await ShowDangerConfirmation(toolCall);
            
            if (confirmed)
            {
                string output = await ExecuteDangerousTool(toolCall);
                await agent.SubmitToolOutputAsync(toolCall.Id, output);
            }
            else
            {
                await agent.SubmitToolOutputAsync(
                    toolCall.Id,
                    "Operation canceled by user"
                );
            }
        }
        else
        {
            // Handle normally
            string output = await ExecuteTool(toolCall);
            await agent.SubmitToolOutputAsync(toolCall.Id, output);
        }
    }
    
    async UniTask<bool> ShowDangerConfirmation(ToolCall toolCall)
    {
        // Show warning dialog
        Debug.LogWarning($"⚠️ Dangerous operation: {toolCall.Function.Name}");
        
        // For example purposes, always return false
        // In real app, show UI and wait for user input
        await UniTask.Delay(100);
        return false;
    }
    
    async UniTask<string> ExecuteDangerousTool(ToolCall toolCall)
    {
        Debug.Log($"🔥 Executing dangerous tool: {toolCall.Function.Name}");
        await UniTask.Delay(1000);
        return "Operation completed (with caution)";
    }
    
    async UniTask<string> ExecuteTool(ToolCall toolCall)
    {
        await UniTask.Delay(100);
        return "Tool executed";
    }
}
```

## Rate Limiting

### Limit Tool Usage

```csharp
public class RateLimitedHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int maxCallsPerMinute = 10;
    
    private Dictionary<string, Queue<DateTime>> callHistory = new();
    
    void Start()
    {
        agent.onUnhandledToolCall.AddListener(OnUnhandledToolCall);
    }
    
    async void OnUnhandledToolCall(ToolCall toolCall)
    {
        if (IsRateLimited(toolCall.Function.Name))
        {
            await agent.SubmitToolOutputAsync(
                toolCall.Id,
                "Error: Rate limit exceeded. Please try again later."
            );
            return;
        }
        
        RecordCall(toolCall.Function.Name);
        
        string output = await ExecuteTool(toolCall);
        await agent.SubmitToolOutputAsync(toolCall.Id, output);
    }
    
    bool IsRateLimited(string toolName)
    {
        if (!callHistory.ContainsKey(toolName))
        {
            callHistory[toolName] = new Queue<DateTime>();
        }
        
        var history = callHistory[toolName];
        var cutoff = DateTime.Now.AddMinutes(-1);
        
        // Remove old calls
        while (history.Count > 0 && history.Peek() < cutoff)
        {
            history.Dequeue();
        }
        
        return history.Count >= maxCallsPerMinute;
    }
    
    void RecordCall(string toolName)
    {
        if (!callHistory.ContainsKey(toolName))
        {
            callHistory[toolName] = new Queue<DateTime>();
        }
        
        callHistory[toolName].Enqueue(DateTime.Now);
    }
    
    async UniTask<string> ExecuteTool(ToolCall toolCall)
    {
        await UniTask.Delay(100);
        return "Executed";
    }
}
```

## Logging & Monitoring

### Log All Unhandled Calls

```csharp
public class ToolCallLogger : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private string logFilePath = "tool_calls.log";
    
    void Start()
    {
        agent.onUnhandledToolCall.AddListener(OnUnhandledToolCall);
    }
    
    async void OnUnhandledToolCall(ToolCall toolCall)
    {
        // Log to file
        LogToFile(toolCall);
        
        // Log to console
        Debug.Log($"📝 Unhandled: {toolCall.Function.Name}");
        Debug.Log($"   Arguments: {toolCall.Function.Arguments}");
        
        // Execute
        string output = await ExecuteTool(toolCall);
        
        // Log result
        LogResult(toolCall, output);
        
        // Submit
        await agent.SubmitToolOutputAsync(toolCall.Id, output);
    }
    
    void LogToFile(ToolCall toolCall)
    {
        string log = $"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] " +
                    $"Tool: {toolCall.Function.Name}, " +
                    $"Args: {toolCall.Function.Arguments}\n";
        
        File.AppendAllText(logFilePath, log);
    }
    
    void LogResult(ToolCall toolCall, string output)
    {
        string log = $"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] " +
                    $"Result for {toolCall.Function.Name}: {output}\n";
        
        File.AppendAllText(logFilePath, log);
    }
    
    async UniTask<string> ExecuteTool(ToolCall toolCall)
    {
        await UniTask.Delay(100);
        return "Success";
    }
}
```

## Fallback Handling

### Default Behavior

```csharp
public class FallbackHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.onUnhandledToolCall.AddListener(OnUnhandledToolCall);
    }
    
    async void OnUnhandledToolCall(ToolCall toolCall)
    {
        Debug.LogWarning($"Unhandled tool: {toolCall.Function.Name}");
        
        // Try common patterns
        string output = toolCall.Function.Name switch
        {
            var name when name.StartsWith("get_") => await HandleGet(toolCall),
            var name when name.StartsWith("set_") => await HandleSet(toolCall),
            var name when name.StartsWith("create_") => await HandleCreate(toolCall),
            var name when name.StartsWith("delete_") => await HandleDelete(toolCall),
            _ => await HandleUnknown(toolCall)
        };
        
        await agent.SubmitToolOutputAsync(toolCall.Id, output);
    }
    
    async UniTask<string> HandleGet(ToolCall toolCall)
    {
        Debug.Log($"GET operation: {toolCall.Function.Name}");
        await UniTask.Delay(100);
        return "Data retrieved";
    }
    
    async UniTask<string> HandleSet(ToolCall toolCall)
    {
        Debug.Log($"SET operation: {toolCall.Function.Name}");
        await UniTask.Delay(100);
        return "Data updated";
    }
    
    async UniTask<string> HandleCreate(ToolCall toolCall)
    {
        Debug.Log($"CREATE operation: {toolCall.Function.Name}");
        await UniTask.Delay(100);
        return "Item created";
    }
    
    async UniTask<string> HandleDelete(ToolCall toolCall)
    {
        Debug.Log($"DELETE operation: {toolCall.Function.Name}");
        await UniTask.Delay(100);
        return "Item deleted";
    }
    
    async UniTask<string> HandleUnknown(ToolCall toolCall)
    {
        Debug.LogError($"Unknown tool pattern: {toolCall.Function.Name}");
        await UniTask.Delay(10);
        return $"Error: Tool '{toolCall.Function.Name}' not supported";
    }
}
```

## Best Practices

### 1. Always Submit Output

```csharp
async void OnUnhandledToolCall(ToolCall toolCall)
{
    try
    {
        string output = await ExecuteTool(toolCall);
        await agent.SubmitToolOutputAsync(toolCall.Id, output);
    }
    catch (Exception ex)
    {
        // Still submit even on error
        await agent.SubmitToolOutputAsync(
            toolCall.Id,
            $"Error: {ex.Message}"
        );
    }
}
```

### 2. Provide Clear Feedback

```csharp
string output = toolCall.Function.Name switch
{
    "unknown_tool" => "Error: This tool is not available",
    "unauthorized" => "Error: You don't have permission",
    "rate_limited" => "Error: Too many requests, please wait",
    _ => "Error: Unknown error"
};
```

### 3. Handle Timeouts

```csharp
async void OnUnhandledToolCall(ToolCall toolCall)
{
    var cts = new CancellationTokenSource();
    cts.CancelAfterSlim(TimeSpan.FromSeconds(30));
    
    try
    {
        string output = await ExecuteTool(toolCall, cts.Token);
        await agent.SubmitToolOutputAsync(toolCall.Id, output);
    }
    catch (OperationCanceledException)
    {
        await agent.SubmitToolOutputAsync(
            toolCall.Id,
            "Error: Operation timed out"
        );
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class UnhandledToolManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject approvalDialogPrefab;
    
    private Queue<ToolCall> pendingTools = new();
    private bool processingTool = false;
    
    void Start()
    {
        agent.onUnhandledToolCall.AddListener(OnUnhandledToolCall);
    }
    
    void OnUnhandledToolCall(ToolCall toolCall)
    {
        Debug.Log($"📋 Unhandled tool queued: {toolCall.Function.Name}");
        pendingTools.Enqueue(toolCall);
        
        if (!processingTool)
        {
            ProcessNextTool();
        }
    }
    
    async void ProcessNextTool()
    {
        if (pendingTools.Count == 0)
        {
            processingTool = false;
            return;
        }
        
        processingTool = true;
        var toolCall = pendingTools.Dequeue();
        
        Debug.Log($"🔧 Processing: {toolCall.Function.Name}");
        
        try
        {
            string output;
            
            if (RequiresApproval(toolCall))
            {
                output = await RequestApproval(toolCall);
            }
            else
            {
                output = await ExecuteAutomatically(toolCall);
            }
            
            await agent.SubmitToolOutputAsync(toolCall.Id, output);
            Debug.Log($"✓ Completed: {toolCall.Function.Name}");
        }
        catch (Exception ex)
        {
            Debug.LogException(ex);
            await agent.SubmitToolOutputAsync(
                toolCall.Id,
                $"Error: {ex.Message}"
            );
        }
        
        // Process next
        ProcessNextTool();
    }
    
    bool RequiresApproval(ToolCall toolCall)
    {
        string[] approvalRequired = {
            "purchase_item",
            "send_message",
            "delete_data",
            "share_location"
        };
        
        return System.Array.Exists(approvalRequired, 
            name => name == toolCall.Function.Name);
    }
    
    async UniTask<string> RequestApproval(ToolCall toolCall)
    {
        Debug.Log($"🔔 Requesting approval for: {toolCall.Function.Name}");
        
        // Show approval dialog
        var dialog = Instantiate(approvalDialogPrefab);
        var handler = dialog.GetComponent<ToolApprovalDialog>();
        
        bool approved = await handler.ShowAndWaitAsync(toolCall);
        
        if (approved)
        {
            return await ExecuteTool(toolCall);
        }
        else
        {
            return "User denied the operation";
        }
    }
    
    async UniTask<string> ExecuteAutomatically(ToolCall toolCall)
    {
        Debug.Log($"⚡ Auto-executing: {toolCall.Function.Name}");
        return await ExecuteTool(toolCall);
    }
    
    async UniTask<string> ExecuteTool(ToolCall toolCall)
    {
        return toolCall.Function.Name switch
        {
            "get_data" => GetData(toolCall),
            "set_data" => SetData(toolCall),
            "purchase_item" => await PurchaseItem(toolCall),
            _ => "Tool not implemented"
        };
    }
    
    string GetData(ToolCall toolCall)
    {
        return "Data: { ... }";
    }
    
    string SetData(ToolCall toolCall)
    {
        return "Data updated";
    }
    
    async UniTask<string> PurchaseItem(ToolCall toolCall)
    {
        await UniTask.Delay(500);
        return "Purchase completed";
    }
}
```

## Next Steps

- [Submit Tool Output](submit-tool-output.md)
- [Registering Executors](registering-executors.md)
- [Function Calls](tool-calls/function.md)
