# Approval Handlers

Handle approval requests for MCP tool executions.

## Overview

Approval Handlers allow you to:

- Control which MCP tools require approval
- Implement custom approval logic
- Show approval dialogs to users
- Track approval history
- Set approval policies

## Basic Setup

### Enable Approval System

```csharp
var mcpController = agent.MCPController;

// Require approval for all MCP tools
mcpController.RequireApproval = true;

// Listen for approval requests
mcpController.onApprovalRequired.AddListener(HandleApprovalRequest);
```

## Approval Requests

### Handle Approval Request

```csharp
async void HandleApprovalRequest(ApprovalRequest request)
{
    Debug.Log($"Approval needed: {request.ToolName}");
    Debug.Log($"Server: {request.ServerName}");
    Debug.Log($"Arguments: {request.Arguments}");
    
    // Show dialog to user
    bool approved = await ShowApprovalDialog(request);
    
    if (approved)
    {
        await mcpController.ApproveToolCallAsync(request.Id);
    }
    else
    {
        await mcpController.DenyToolCallAsync(request.Id);
    }
}
```

### Approval Dialog

```csharp
public class ApprovalDialog : MonoBehaviour
{
    [SerializeField] private GameObject dialogPrefab;
    [SerializeField] private Transform dialogContainer;
    
    public async UniTask<bool> ShowApprovalDialog(ApprovalRequest request)
    {
        var dialog = Instantiate(dialogPrefab, dialogContainer);
        var handler = dialog.GetComponent<ApprovalDialogUI>();
        
        // Configure dialog
        handler.SetTitle($"Approve Tool Execution?");
        handler.SetMessage($@"
Tool: {request.ToolName}
Server: {request.ServerName}

{FormatArguments(request.Arguments)}

Allow this action?
");
        
        // Wait for user response
        bool approved = await handler.WaitForResponse();
        
        Destroy(dialog);
        
        return approved;
    }
    
    string FormatArguments(Dictionary<string, object> arguments)
    {
        if (arguments == null || arguments.Count == 0)
            return "No arguments";
        
        return string.Join("\n", arguments.Select(kvp =>
            $"  {kvp.Key}: {kvp.Value}"));
    }
}
```

### Approval Dialog UI

```csharp
public class ApprovalDialogUI : MonoBehaviour
{
    [SerializeField] private TMP_Text titleText;
    [SerializeField] private TMP_Text messageText;
    [SerializeField] private Button approveButton;
    [SerializeField] private Button denyButton;
    
    private UniTaskCompletionSource<bool> completionSource;
    
    void Start()
    {
        approveButton.onClick.AddListener(() => Respond(true));
        denyButton.onClick.AddListener(() => Respond(false));
    }
    
    public void SetTitle(string title)
    {
        titleText.text = title;
    }
    
    public void SetMessage(string message)
    {
        messageText.text = message;
    }
    
    public UniTask<bool> WaitForResponse()
    {
        completionSource = new UniTaskCompletionSource<bool>();
        return completionSource.Task;
    }
    
    void Respond(bool approved)
    {
        completionSource?.TrySetResult(approved);
    }
}
```

## Approval Policies

### Define Approval Policies

```csharp
public class ApprovalPolicyManager : MonoBehaviour
{
    private Dictionary<string, ApprovalPolicy> policies = new();
    
    public void SetupPolicies()
    {
        // Always approve safe operations
        policies["read_file"] = ApprovalPolicy.AutoApprove;
        policies["list_files"] = ApprovalPolicy.AutoApprove;
        policies["get_*"] = ApprovalPolicy.AutoApprove;
        
        // Always require approval for dangerous operations
        policies["delete_file"] = ApprovalPolicy.RequireApproval;
        policies["write_file"] = ApprovalPolicy.RequireApproval;
        policies["execute_*"] = ApprovalPolicy.RequireApproval;
        
        // Deny certain operations
        policies["format_disk"] = ApprovalPolicy.AlwaysDeny;
    }
    
    public ApprovalPolicy GetPolicy(string toolName)
    {
        // Exact match
        if (policies.TryGetValue(toolName, out var policy))
            return policy;
        
        // Wildcard match
        foreach (var kvp in policies)
        {
            if (kvp.Key.EndsWith("*"))
            {
                string prefix = kvp.Key.Substring(0, kvp.Key.Length - 1);
                if (toolName.StartsWith(prefix))
                    return kvp.Value;
            }
        }
        
        // Default: require approval
        return ApprovalPolicy.RequireApproval;
    }
}

public enum ApprovalPolicy
{
    AutoApprove,
    RequireApproval,
    AlwaysDeny
}
```

### Policy-Based Handler

```csharp
public class PolicyBasedApprovalHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private ApprovalPolicyManager policyManager;
    
    void Start()
    {
        policyManager.SetupPolicies();
        
        agent.MCPController.onApprovalRequired.AddListener(HandleApprovalWithPolicy);
    }
    
    async void HandleApprovalWithPolicy(ApprovalRequest request)
    {
        var policy = policyManager.GetPolicy(request.ToolName);
        
        switch (policy)
        {
            case ApprovalPolicy.AutoApprove:
                Debug.Log($"✓ Auto-approved: {request.ToolName}");
                await agent.MCPController.ApproveToolCallAsync(request.Id);
                break;
            
            case ApprovalPolicy.AlwaysDeny:
                Debug.Log($"✗ Auto-denied: {request.ToolName}");
                await agent.MCPController.DenyToolCallAsync(request.Id);
                break;
            
            case ApprovalPolicy.RequireApproval:
                bool approved = await ShowApprovalDialog(request);
                
                if (approved)
                {
                    await agent.MCPController.ApproveToolCallAsync(request.Id);
                }
                else
                {
                    await agent.MCPController.DenyToolCallAsync(request.Id);
                }
                break;
        }
    }
    
    async UniTask<bool> ShowApprovalDialog(ApprovalRequest request)
    {
        // Show dialog and wait for response
        return false; // Placeholder
    }
}
```

## Approval History

### Track Approvals

```csharp
public class ApprovalHistory : MonoBehaviour
{
    private List<ApprovalRecord> history = new();
    
    public void RecordApproval(ApprovalRequest request, bool approved)
    {
        history.Add(new ApprovalRecord
        {
            Timestamp = DateTime.Now,
            ToolName = request.ToolName,
            ServerName = request.ServerName,
            Arguments = request.Arguments,
            Approved = approved
        });
        
        Debug.Log($"Recorded approval: {request.ToolName} = {approved}");
    }
    
    public List<ApprovalRecord> GetHistory(int count = 100)
    {
        return history.TakeLast(count).ToList();
    }
    
    public List<ApprovalRecord> GetHistoryForTool(string toolName)
    {
        return history.Where(r => r.ToolName == toolName).ToList();
    }
    
    public void ClearHistory()
    {
        history.Clear();
    }
}

[System.Serializable]
public class ApprovalRecord
{
    public DateTime Timestamp;
    public string ToolName;
    public string ServerName;
    public Dictionary<string, object> Arguments;
    public bool Approved;
}
```

### History UI

```csharp
public class ApprovalHistoryUI : MonoBehaviour
{
    [SerializeField] private ApprovalHistory history;
    [SerializeField] private GameObject recordPrefab;
    [SerializeField] private Transform recordsContainer;
    
    public void ShowHistory()
    {
        // Clear existing
        foreach (Transform child in recordsContainer)
        {
            Destroy(child.gameObject);
        }
        
        // Display history
        var records = history.GetHistory();
        
        foreach (var record in records)
        {
            CreateRecordItem(record);
        }
    }
    
    void CreateRecordItem(ApprovalRecord record)
    {
        var recordObj = Instantiate(recordPrefab, recordsContainer);
        
        var timeText = recordObj.transform.Find("Time").GetComponent<TMP_Text>();
        var toolText = recordObj.transform.Find("Tool").GetComponent<TMP_Text>();
        var statusText = recordObj.transform.Find("Status").GetComponent<TMP_Text>();
        
        timeText.text = record.Timestamp.ToString("HH:mm:ss");
        toolText.text = $"{record.ServerName}.{record.ToolName}";
        statusText.text = record.Approved ? "✓ Approved" : "✗ Denied";
        statusText.color = record.Approved ? Color.green : Color.red;
    }
}
```

## Conditional Approval

### Context-Based Approval

```csharp
public class ContextualApprovalHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    async void HandleApprovalRequest(ApprovalRequest request)
    {
        // Auto-approve based on context
        if (ShouldAutoApprove(request))
        {
            Debug.Log($"✓ Auto-approved based on context");
            await agent.MCPController.ApproveToolCallAsync(request.Id);
            return;
        }
        
        // Show dialog for uncertain cases
        bool approved = await ShowApprovalDialog(request);
        
        if (approved)
        {
            await agent.MCPController.ApproveToolCallAsync(request.Id);
        }
        else
        {
            await agent.MCPController.DenyToolCallAsync(request.Id);
        }
    }
    
    bool ShouldAutoApprove(ApprovalRequest request)
    {
        // Check file path safety
        if (request.ToolName.Contains("file"))
        {
            if (request.Arguments.TryGetValue("path", out var path))
            {
                string pathStr = path.ToString();
                
                // Deny system paths
                if (pathStr.Contains("System32") || pathStr.Contains("/etc/"))
                    return false;
                
                // Approve safe paths
                if (pathStr.StartsWith(Application.dataPath))
                    return true;
            }
        }
        
        // Check read-only operations
        if (request.ToolName.StartsWith("read_") || 
            request.ToolName.StartsWith("list_") ||
            request.ToolName.StartsWith("get_"))
        {
            return true;
        }
        
        return false;
    }
}
```

### User Preference-Based

```csharp
public class PreferenceBasedApproval : MonoBehaviour
{
    [SerializeField] private bool alwaysApproveReadOperations = true;
    [SerializeField] private bool alwaysApproveKnownServers = true;
    [SerializeField] private List<string> trustedServers = new();
    
    bool ShouldAutoApprove(ApprovalRequest request)
    {
        // Check user preferences
        if (alwaysApproveReadOperations && IsReadOperation(request.ToolName))
            return true;
        
        if (alwaysApproveKnownServers && trustedServers.Contains(request.ServerName))
            return true;
        
        return false;
    }
    
    bool IsReadOperation(string toolName)
    {
        string[] readOps = { "read", "get", "list", "search", "find" };
        return readOps.Any(op => toolName.StartsWith(op));
    }
}
```

## Bulk Approval

### Approve Multiple Requests

```csharp
public class BulkApprovalHandler : MonoBehaviour
{
    private Queue<ApprovalRequest> pendingRequests = new();
    
    void OnApprovalRequired(ApprovalRequest request)
    {
        pendingRequests.Enqueue(request);
        
        if (pendingRequests.Count == 1)
        {
            ShowBulkApprovalDialog();
        }
    }
    
    async void ShowBulkApprovalDialog()
    {
        if (pendingRequests.Count == 0)
            return;
        
        var requests = pendingRequests.ToList();
        
        // Show all pending approvals
        bool approveAll = await ShowBulkDialog(requests);
        
        // Process all requests
        while (pendingRequests.Count > 0)
        {
            var request = pendingRequests.Dequeue();
            
            if (approveAll)
            {
                await agent.MCPController.ApproveToolCallAsync(request.Id);
            }
            else
            {
                await agent.MCPController.DenyToolCallAsync(request.Id);
            }
        }
    }
    
    async UniTask<bool> ShowBulkDialog(List<ApprovalRequest> requests)
    {
        // Show dialog with all pending requests
        // Return true if user approves all
        return false; // Placeholder
    }
}
```

## Error Handling

### Handle Approval Errors

```csharp
agent.MCPController.onApprovalError.AddListener((requestId, error) =>
{
    Debug.LogError($"Approval error: {error}");
    
    if (error.Contains("timeout"))
    {
        ShowMessage("Approval request timed out");
    }
    else if (error.Contains("expired"))
    {
        ShowMessage("Approval request expired");
    }
    else
    {
        ShowMessage("Failed to process approval");
    }
});
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class ApprovalManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject approvalDialogPrefab;
    [SerializeField] private Transform dialogContainer;
    
    private ApprovalPolicyManager policyManager;
    private ApprovalHistory history;
    
    void Start()
    {
        policyManager = GetComponent<ApprovalPolicyManager>();
        history = GetComponent<ApprovalHistory>();
        
        SetupApprovalSystem();
    }
    
    void SetupApprovalSystem()
    {
        var mcpController = agent.MCPController;
        
        // Configure policies
        policyManager.SetupPolicies();
        
        // Listen for approval requests
        mcpController.onApprovalRequired.AddListener(HandleApprovalRequest);
        mcpController.onApprovalCompleted.AddListener(OnApprovalCompleted);
        mcpController.onApprovalError.AddListener(OnApprovalError);
        
        Debug.Log("✓ Approval system ready");
    }
    
    async void HandleApprovalRequest(ApprovalRequest request)
    {
        Debug.Log($"🔒 Approval requested: {request.ToolName}");
        
        // Check policy
        var policy = policyManager.GetPolicy(request.ToolName);
        
        bool approved;
        
        switch (policy)
        {
            case ApprovalPolicy.AutoApprove:
                approved = true;
                Debug.Log($"✓ Auto-approved: {request.ToolName}");
                break;
            
            case ApprovalPolicy.AlwaysDeny:
                approved = false;
                Debug.Log($"✗ Auto-denied: {request.ToolName}");
                break;
            
            default:
                approved = await ShowApprovalDialog(request);
                break;
        }
        
        // Record decision
        history.RecordApproval(request, approved);
        
        // Send decision
        if (approved)
        {
            await agent.MCPController.ApproveToolCallAsync(request.Id);
        }
        else
        {
            await agent.MCPController.DenyToolCallAsync(request.Id);
        }
    }
    
    async UniTask<bool> ShowApprovalDialog(ApprovalRequest request)
    {
        var dialog = Instantiate(approvalDialogPrefab, dialogContainer);
        var ui = dialog.GetComponent<ApprovalDialogUI>();
        
        ui.SetTitle("Approve Tool Execution?");
        ui.SetMessage($@"
<b>Tool:</b> {request.ToolName}
<b>Server:</b> {request.ServerName}

<b>Arguments:</b>
{FormatArguments(request.Arguments)}

Do you want to allow this action?
");
        
        bool approved = await ui.WaitForResponse();
        
        Destroy(dialog);
        
        return approved;
    }
    
    string FormatArguments(Dictionary<string, object> arguments)
    {
        if (arguments == null || arguments.Count == 0)
            return "None";
        
        return string.Join("\n", arguments.Select(kvp =>
            $"  • {kvp.Key}: {kvp.Value}"));
    }
    
    void OnApprovalCompleted(string requestId, bool approved)
    {
        Debug.Log($"✓ Approval processed: {requestId} = {approved}");
    }
    
    void OnApprovalError(string requestId, string error)
    {
        Debug.LogError($"Approval error ({requestId}): {error}");
    }
}
```

## Next Steps

- [Approval Timeout](approval-timeout.md)
- [Access Token Service](access-token-service.md)
- [OAuth Providers](oauth-providers.md)
