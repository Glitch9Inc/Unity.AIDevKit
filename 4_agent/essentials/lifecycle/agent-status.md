---
icon: circle-info
---

# Agent Status

Understanding agent lifecycle states and transitions.

## AgentStatus Enum

```csharp
public enum AgentStatus
{
    None,
    Initializing,
    InitializationFailed,
    LoadingChat,
    RetrievingAccessTokens,
    Ready,
    WaitingForToolOutput,
    Processing,
    Cancelling,
    Terminating
}
```

## Status Descriptions

### None

**Initial state before any initialization.**

- Agent just created
- No setup performed yet
- Cannot process requests

When to expect:

- Immediately after `new Agent()`
- Before `InitializeAsync()` called

### Initializing

**Agent performing one-time setup.**

- Validating configuration
- Setting up services
- Preparing resources

Typical duration: 100-500ms

### InitializationFailed

**Initialization encountered an error.**

Common causes:

- Invalid API key
- Invalid model ID
- Network error
- Configuration error

Actions:

- Check error message
- Fix configuration
- Retry initialization

### LoadingChat

**Loading conversation history.**

- Reading from conversation store
- Restoring message history
- Loading conversation metadata

Typical duration: 50-200ms (local), 200-1000ms (remote)

### RetrievingAccessTokens

**Fetching API keys or OAuth tokens.**

- Retrieving stored credentials
- Refreshing OAuth tokens
- Validating access permissions

Typical duration: 100-500ms

### Ready

**Agent idle and ready for input.**

- Initialization complete
- Can accept new messages
- Waiting for user input

This is the normal idle state.

### WaitingForToolOutput

**Waiting for external tool output.**

- Tool call made but not handled locally
- UnhandledToolCallBehaviour = SubmitToolOutput
- Waiting for `SubmitToolOutput()` call

Timeout: Configurable via `SubmitToolOutputTimeoutSeconds`

### Processing

**Actively processing a request.**

- Sending API request
- Receiving response
- Executing tools
- Generating output

Duration varies based on:

- Model speed
- Response length
- Tool execution time

### Cancelling

**Cancellation in progress.**

- User called `CancelAsync()`
- Stopping ongoing work
- Cleaning up resources

Typical duration: 50-200ms

### Terminating

**Agent shutting down.**

- `Dispose()` called
- Saving conversation
- Releasing resources

After this, agent cannot be reused.

## Status Transitions

```
None → Initializing → LoadingChat → Ready → Processing → Ready
       ↓                                        ↓
       InitializationFailed            WaitingForToolOutput → Ready
                                                 ↓
                                            Cancelling → Ready
                                                 
Any State → Terminating
```

## Monitoring Status

### Subscribe to Changes

```csharp
agent.onStatusChanged.AddListener(status =>
{
    Debug.Log($"Status changed to: {status}");
    UpdateUI(status);
});
```

### Check Current Status

```csharp
if (agent.Status == AgentStatus.Ready)
{
    // Safe to send message
    await agent.SendAsync("Hello");
}
```

### Wait for Ready

```csharp
async UniTask WaitUntilReady()
{
    while (agent.Status != AgentStatus.Ready)
    {
        await UniTask.Delay(100);
    }
}
```

## Status-Based UI

```csharp
void UpdateUI(AgentStatus status)
{
    switch (status)
    {
        case AgentStatus.None:
        case AgentStatus.Initializing:
            ShowLoadingScreen("Initializing...");
            DisableInput();
            break;
            
        case AgentStatus.InitializationFailed:
            HideLoadingScreen();
            ShowError("Failed to initialize");
            DisableInput();
            break;
            
        case AgentStatus.LoadingChat:
            ShowLoadingScreen("Loading conversation...");
            DisableInput();
            break;
            
        case AgentStatus.Ready:
            HideLoadingScreen();
            EnableInput();
            statusText.text = "Ready";
            break;
            
        case AgentStatus.Processing:
            ShowLoadingIndicator();
            DisableInput();
            statusText.text = "Thinking...";
            break;
            
        case AgentStatus.WaitingForToolOutput:
            ShowToolOutputDialog();
            statusText.text = "Waiting for tool...";
            break;
            
        case AgentStatus.Cancelling:
            statusText.text = "Cancelling...";
            break;
            
        case AgentStatus.Terminating:
            ShowLoadingScreen("Shutting down...");
            DisableInput();
            break;
    }
}
```

## Status Validation

### Before Operations

```csharp
public async void SendMessage(string text)
{
    // Check if agent is ready
    if (agent.Status != AgentStatus.Ready)
    {
        Debug.LogWarning($"Cannot send: Agent status is {agent.Status}");
        return;
    }
    
    await agent.SendAsync(text);
}
```

### Safe Operation Helper

```csharp
async UniTask<bool> WaitForReadyState(float timeout = 10f)
{
    float elapsed = 0f;
    
    while (agent.Status != AgentStatus.Ready && elapsed < timeout)
    {
        await UniTask.Delay(100);
        elapsed += 0.1f;
    }
    
    return agent.Status == AgentStatus.Ready;
}

// Usage
if (await WaitForReadyState())
{
    await agent.SendAsync("Hello");
}
else
{
    Debug.LogError("Agent not ready after timeout");
}
```

## Status Timeouts

### Initialization Timeout

```csharp
async UniTask InitializeWithTimeout(float timeoutSeconds = 30f)
{
    var cts = new CancellationTokenSource(TimeSpan.FromSeconds(timeoutSeconds));
    
    try
    {
        await agent.InitializeAsync(cts.Token);
    }
    catch (OperationCanceledException)
    {
        Debug.LogError("Initialization timeout");
    }
}
```

### Processing Timeout

```csharp
async UniTask<Response> SendWithTimeout(string message, float timeoutSeconds = 60f)
{
    var cts = new CancellationTokenSource(TimeSpan.FromSeconds(timeoutSeconds));
    
    try
    {
        return await agent.SendAsync(message, cts.Token);
    }
    catch (OperationCanceledException)
    {
        Debug.LogError("Request timeout");
        return null;
    }
}
```

## Status Persistence

### Save Status

```csharp
void SaveAgentState()
{
    PlayerPrefs.SetString("AgentStatus", agent.Status.ToString());
    PlayerPrefs.SetString("ConversationId", agent.ConversationId);
    PlayerPrefs.Save();
}
```

### Restore Status

```csharp
async UniTask RestoreAgentState()
{
    string savedStatus = PlayerPrefs.GetString("AgentStatus", "None");
    string conversationId = PlayerPrefs.GetString("ConversationId", null);
    
    if (savedStatus == "Ready" || savedStatus == "WaitingForToolOutput")
    {
        // Restore conversation
        agent.ConversationId = conversationId;
        await agent.InitializeAsync();
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using UnityEngine.UI;
using Glitch9.AIDevKit.Agents;

public class StatusManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Text statusText;
    [SerializeField] private Image statusIndicator;
    [SerializeField] private Button sendButton;
    
    void Start()
    {
        agent.onStatusChanged.AddListener(OnStatusChanged);
        UpdateStatusUI(agent.Status);
    }
    
    void OnStatusChanged(AgentStatus status)
    {
        Debug.Log($"Status: {status}");
        UpdateStatusUI(status);
    }
    
    void UpdateStatusUI(AgentStatus status)
    {
        // Update text
        statusText.text = GetStatusDisplayText(status);
        
        // Update indicator color
        statusIndicator.color = GetStatusColor(status);
        
        // Update interactivity
        sendButton.interactable = CanSendMessage(status);
    }
    
    string GetStatusDisplayText(AgentStatus status)
    {
        return status switch
        {
            AgentStatus.None => "Not initialized",
            AgentStatus.Initializing => "Starting up...",
            AgentStatus.InitializationFailed => "Failed to start",
            AgentStatus.LoadingChat => "Loading history...",
            AgentStatus.Ready => "Ready",
            AgentStatus.Processing => "Thinking...",
            AgentStatus.WaitingForToolOutput => "Waiting...",
            AgentStatus.Cancelling => "Stopping...",
            AgentStatus.Terminating => "Shutting down...",
            _ => "Unknown"
        };
    }
    
    Color GetStatusColor(AgentStatus status)
    {
        return status switch
        {
            AgentStatus.Ready => Color.green,
            AgentStatus.Processing => Color.yellow,
            AgentStatus.InitializationFailed => Color.red,
            AgentStatus.WaitingForToolOutput => new Color(1f, 0.5f, 0f), // Orange
            _ => Color.gray
        };
    }
    
    bool CanSendMessage(AgentStatus status)
    {
        return status == AgentStatus.Ready;
    }
}
```

## Next Steps

- [Initialization](initialization.md) - Initialize agent
- [Disposal](disposal.md) - Proper cleanup
- [Error Handling](../events/error-handling.md) - Handle errors
