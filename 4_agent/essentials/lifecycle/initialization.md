---
icon: power-off
---

# Initialization

Learn how to properly initialize your agent for use.

## Initialization Process

Agent initialization involves several steps:

1. **Validate Configuration** - Check all settings and dependencies
2. **Initialize Services** - Set up chat, audio, and tool services
3. **Load Conversation** - Restore conversation history (if applicable)
4. **Retrieve Tokens** - Fetch API keys and OAuth tokens
5. **Transition to Ready** - Agent ready to accept input

## Automatic Initialization (AgentBehaviour)

### Auto-Init Enabled

```csharp
[SerializeField] private AgentBehaviour agent;

void Start()
{
    // Agent initializes automatically in Start() if AutoInit = true
    agent.onStatusChanged.AddListener(OnStatusChanged);
}

void OnStatusChanged(AgentStatus status)
{
    if (status == AgentStatus.Ready)
    {
        Debug.Log("Agent ready!");
        EnableUI();
    }
    else if (status == AgentStatus.InitializationFailed)
    {
        Debug.LogError("Initialization failed!");
    }
}
```

### Manual Init

```csharp
[SerializeField] private AgentBehaviour agent;

async void Start()
{
    // Disable auto-init in inspector
    // agent.AutoInit = false;
    
    try
    {
        await agent.InitializeAsync();
        Debug.Log("Agent initialized successfully");
    }
    catch (Exception ex)
    {
        Debug.LogError($"Init failed: {ex.Message}");
    }
}
```

## Manual Initialization (Agent Class)

### Basic Initialization

```csharp
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AgentManager
{
    private Agent agent;
    
    public async UniTask InitializeAsync(AgentSettings settings)
    {
        // Create agent
        agent = new Agent(
            settings: settings,
            behaviour: CreateBehaviour(),
            hooks: CreateHooks()
        );
        
        // Initialize
        await agent.InitializeAsync();
        
        Debug.Log($"Agent status: {agent.Status}");
    }
    
    private IAgentBehaviour CreateBehaviour()
    {
        return new AgentBehaviourConfig
        {
            AutoInit = false,
            ConversationStoreType = ConversationStoreType.LocalFile,
            Stream = true,
            LogLevel = System.Diagnostics.TraceLevel.Info
        };
    }
    
    private AgentHooks CreateHooks()
    {
        return new AgentHooks
        {
            StatusChanged = OnStatusChanged,
            TextDelta = OnTextDelta
        };
    }
    
    private void OnStatusChanged(AgentStatus status)
    {
        Debug.Log($"Status: {status}");
    }
    
    private void OnTextDelta(string delta)
    {
        Debug.Log($"Delta: {delta}");
    }
}
```

### Advanced Initialization

```csharp
public async UniTask InitializeWithServicesAsync()
{
    // Custom services
    var services = new AgentServices
    {
        ChatApi = new CustomChatService(),
        SpeechApi = new CustomSpeechService()
    };
    
    // Custom logger
    var logger = new CustomLogger(TraceLevel.Verbose);
    
    // MCP token service
    var tokenService = new McpAccessTokenService();
    
    // Preferences store
    var prefsStore = new PlayerPrefsStore<AgentPrefs>();
    
    // Create agent with all dependencies
    agent = new Agent(
        settings: settings,
        behaviour: behaviour,
        initialTools: GetInitialTools(),
        toolDefinitions: GetToolDefinitions(),
        hooks: hooks,
        services: services,
        tokenService: tokenService,
        prefsStore: prefsStore,
        logger: logger
    );
    
    // Initialize with cancellation token
    var cts = new CancellationTokenSource(TimeSpan.FromSeconds(30));
    
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

## Initialization with Conversation

### Load Existing Conversation

```csharp
async void Start()
{
    // Set conversation ID before initializing
    agent.ConversationId = "conv_abc123";
    
    await agent.InitializeAsync();
    
    // Conversation is loaded and ready
    Debug.Log($"Loaded {agent.Messages.Count} messages");
}
```

### Create New Conversation

```csharp
async void Start()
{
    // Initialize without conversation ID
    agent.ConversationId = null;
    await agent.InitializeAsync();
    
    // Agent creates new conversation
    Debug.Log($"New conversation: {agent.ConversationId}");
}
```

## Monitoring Initialization

### Status Events

```csharp
void Start()
{
    agent.onStatusChanged.AddListener(status =>
    {
        switch (status)
        {
            case AgentStatus.Initializing:
                ShowLoadingScreen("Initializing agent...");
                break;
                
            case AgentStatus.LoadingChat:
                ShowLoadingScreen("Loading conversation...");
                break;
                
            case AgentStatus.RetrievingAccessTokens:
                ShowLoadingScreen("Authenticating...");
                break;
                
            case AgentStatus.Ready:
                HideLoadingScreen();
                EnableChat();
                break;
                
            case AgentStatus.InitializationFailed:
                ShowError("Failed to initialize agent");
                break;
        }
    });
}
```

### Progress Tracking

```csharp
async UniTask InitializeWithProgressAsync()
{
    var progress = new Progress<float>(p =>
    {
        UpdateProgressBar(p);
    });
    
    await agent.InitializeAsync(progress: progress);
}
```

## Error Handling

### Common Initialization Errors

```csharp
try
{
    await agent.InitializeAsync();
}
catch (ArgumentException ex)
{
    // Configuration error
    Debug.LogError($"Invalid configuration: {ex.Message}");
    ShowConfigError();
}
catch (UnauthorizedAccessException ex)
{
    // API key error
    Debug.LogError($"Authentication failed: {ex.Message}");
    ShowApiKeyDialog();
}
catch (HttpRequestException ex)
{
    // Network error
    Debug.LogError($"Network error: {ex.Message}");
    ShowNetworkError();
}
catch (TimeoutException ex)
{
    // Timeout
    Debug.LogError($"Initialization timeout: {ex.Message}");
    ShowTimeoutError();
}
catch (Exception ex)
{
    // Unknown error
    Debug.LogError($"Unexpected error: {ex.Message}");
    ShowGenericError();
}
```

### Retry Logic

```csharp
async UniTask<bool> InitializeWithRetryAsync(int maxRetries = 3)
{
    for (int i = 0; i < maxRetries; i++)
    {
        try
        {
            await agent.InitializeAsync();
            return true;
        }
        catch (Exception ex)
        {
            Debug.LogWarning($"Init attempt {i + 1} failed: {ex.Message}");
            
            if (i < maxRetries - 1)
            {
                // Wait before retry (exponential backoff)
                await UniTask.Delay(TimeSpan.FromSeconds(Math.Pow(2, i)));
            }
        }
    }
    
    return false;
}
```

## Reinitialization

### After Configuration Change

```csharp
async void ChangeConfiguration()
{
    // Dispose current agent
    agent.Dispose();
    
    // Update settings
    settings.Model = "gpt-4o";
    settings.Temperature = 0.9f;
    
    // Create and initialize new agent
    agent = new Agent(settings, behaviour, hooks);
    await agent.InitializeAsync();
}
```

### After Initialization Failure

```csharp
async void RetryAfterFailure()
{
    if (agent.Status == AgentStatus.InitializationFailed)
    {
        // Dispose failed agent
        agent.Dispose();
        
        // Fix configuration based on error
        FixConfiguration();
        
        // Recreate and retry
        agent = new Agent(settings, behaviour, hooks);
        await agent.InitializeAsync();
    }
}
```

## Initialization Checklist

Before initializing, verify:

- [ ] API key is configured
- [ ] Model ID is valid
- [ ] Required services are available
- [ ] Network connection exists
- [ ] Conversation store is accessible (if loading)
- [ ] Audio components assigned (if using audio)
- [ ] Tool executors registered (if using tools)

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AgentInitializer : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject loadingScreen;
    [SerializeField] private GameObject errorDialog;
    
    async void Start()
    {
        await InitializeAgentAsync();
    }
    
    async UniTask InitializeAgentAsync()
    {
        // Show loading
        loadingScreen.SetActive(true);
        
        // Subscribe to status
        agent.onStatusChanged.AddListener(OnStatusChanged);
        
        try
        {
            // Initialize with timeout
            var cts = new CancellationTokenSource(TimeSpan.FromSeconds(30));
            await agent.InitializeAsync(cts.Token);
            
            Debug.Log("Agent ready!");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Initialization failed: {ex.Message}");
            ShowError(ex.Message);
        }
        finally
        {
            loadingScreen.SetActive(false);
        }
    }
    
    void OnStatusChanged(AgentStatus status)
    {
        Debug.Log($"Status: {status}");
        
        if (status == AgentStatus.Ready)
        {
            OnAgentReady();
        }
    }
    
    void OnAgentReady()
    {
        // Enable UI
        EnableChatUI();
        
        // Send welcome message if new conversation
        if (agent.Messages.Count == 0)
        {
            ShowWelcomeMessage();
        }
    }
    
    void ShowError(string message)
    {
        errorDialog.SetActive(true);
        // Show error details...
    }
}
```

## Next Steps

- [Agent Status](agent-status.md) - All status states
- [Disposal](disposal.md) - Proper cleanup
- [Agent Configuration](../configuration/README.md) - Configure settings
