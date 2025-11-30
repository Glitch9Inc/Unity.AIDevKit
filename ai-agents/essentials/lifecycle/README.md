# Agent Lifecycle

Understanding the agent's lifecycle is essential for proper initialization, usage, and cleanup.

## Lifecycle States

The agent progresses through these states during its lifetime:

```
None → Initializing → LoadingChat → Ready → Processing → Ready → Terminating
                ↓                              ↓
        InitializationFailed            WaitingForToolOutput
```

### AgentStatus Enum

| Status | Description |
|--------|-------------|
| **None** | Uninitialized state before setup |
| **Initializing** | Performing one-time initialization |
| **InitializationFailed** | Initialization error occurred |
| **LoadingChat** | Loading conversation history |
| **RetrievingAccessTokens** | Fetching OAuth/API tokens |
| **Ready** | Idle and ready for input |
| **Processing** | Actively processing request |
| **WaitingForToolOutput** | Waiting for manual tool output |
| **Cancelling** | Cancellation in progress |
| **Terminating** | Shutting down |

## Initialization

### Automatic Initialization (AgentBehaviour)

```csharp
public class MyScript : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        // Agent auto-initializes if AutoInit = true
        agent.onStatusChanged.AddListener(OnStatusChanged);
    }
    
    void OnStatusChanged(AgentStatus status)
    {
        if (status == AgentStatus.Ready)
        {
            Debug.Log("Agent ready!");
        }
    }
}
```

### Manual Initialization (Agent)

```csharp
Agent agent = new Agent(settings, behaviour, hooks);

try
{
    await agent.InitializeAsync();
    Debug.Log($"Agent initialized. Status: {agent.Status}");
}
catch (Exception ex)
{
    Debug.LogError($"Initialization failed: {ex.Message}");
}
```

### Initialization Steps

1. **Validate Configuration** - Check settings and dependencies
2. **Initialize Services** - Set up chat, audio, and tool services
3. **Load Conversation** - Restore conversation history (if any)
4. **Retrieve Tokens** - Fetch API/OAuth tokens if needed
5. **Transition to Ready** - Agent is ready to accept input

## Status Monitoring

```csharp
// Subscribe to status changes
agent.onStatusChanged.AddListener(status =>
{
    switch (status)
    {
        case AgentStatus.Ready:
            EnableUI();
            break;
        case AgentStatus.Processing:
            ShowLoadingIndicator();
            break;
        case AgentStatus.InitializationFailed:
            ShowError("Agent initialization failed");
            break;
    }
});
```

## Disposal

### Automatic Disposal (AgentBehaviour)

```csharp
// Happens automatically in OnDestroy
void OnDestroy()
{
    // AgentBehaviour handles cleanup
}
```

### Manual Disposal (Agent)

```csharp
void OnApplicationQuit()
{
    agent?.Dispose();
}
```

### Cleanup Process

1. Cancel any active requests
2. Save conversation (if applicable)
3. Dispose services and controllers
4. Release resources
5. Transition to Terminating

## Reinitialization

```csharp
// Reset after error
if (agent.Status == AgentStatus.InitializationFailed)
{
    agent.Dispose();
    agent = new Agent(settings, behaviour, hooks);
    await agent.InitializeAsync();
}
```

## Best Practices

### Check Status Before Operations

```csharp
public async void SendMessage(string text)
{
    if (agent.Status != AgentStatus.Ready)
    {
        Debug.LogWarning("Agent not ready");
        return;
    }
    
    await agent.SendAsync(text);
}
```

### Handle Initialization Errors

```csharp
try
{
    await agent.InitializeAsync();
}
catch (ArgumentException ex)
{
    Debug.LogError($"Configuration error: {ex.Message}");
}
catch (UnauthorizedAccessException ex)
{
    Debug.LogError($"Authentication error: {ex.Message}");
}
catch (Exception ex)
{
    Debug.LogError($"Unexpected error: {ex.Message}");
}
```

### Graceful Shutdown

```csharp
async void OnApplicationQuit()
{
    if (agent != null && agent.Status == AgentStatus.Processing)
    {
        await agent.CancelAsync();
    }
    
    agent?.Dispose();
}
```

## Next Steps

- [Initialization](initialization.md) - Detailed initialization guide
- [Agent Status](agent-status.md) - All status states explained
- [Disposal](disposal.md) - Proper cleanup procedures
