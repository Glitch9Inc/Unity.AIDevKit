# Agent Lifecycle Hooks

Monitor and respond to agent lifecycle events.

## Overview

Agent Lifecycle Hooks provide:

- Agent state monitoring
- Initialization callbacks
- Shutdown handling
- Error recovery
- Event subscriptions

## Basic Setup

### Subscribe to Events

```csharp
public class AgentEventListener : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        // Lifecycle events
        agent.onAgentInitialized.AddListener(OnAgentInitialized);
        agent.onAgentStarted.AddListener(OnAgentStarted);
        agent.onAgentStopped.AddListener(OnAgentStopped);
        agent.onAgentError.AddListener(OnAgentError);
    }
    
    void OnAgentInitialized()
    {
        Debug.Log("✓ Agent initialized");
    }
    
    void OnAgentStarted()
    {
        Debug.Log("▶️ Agent started");
    }
    
    void OnAgentStopped()
    {
        Debug.Log("⏹️ Agent stopped");
    }
    
    void OnAgentError(string error)
    {
        Debug.LogError($"❌ Agent error: {error}");
    }
}
```

## Initialization Hooks

### Pre-Initialization

```csharp
public class PreInitializationHook : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Awake()
    {
        agent.onBeforeInitialize.AddListener(OnBeforeInitialize);
    }
    
    void OnBeforeInitialize()
    {
        Debug.Log("Preparing agent initialization...");
        
        // Load saved preferences
        LoadPreferences();
        
        // Configure services
        ConfigureServices();
        
        // Validate API keys
        ValidateAPIKeys();
    }
    
    void LoadPreferences()
    {
        // Load user preferences
        string savedModel = PlayerPrefs.GetString("PreferredModel", "gpt-4o");
        agent.ParametersController.SetModel(savedModel);
    }
    
    void ConfigureServices()
    {
        // Setup custom services
    }
    
    void ValidateAPIKeys()
    {
        // Check API key validity
    }
}
```

### Post-Initialization

```csharp
void Start()
{
    agent.onAgentInitialized.AddListener(OnAgentInitialized);
}

void OnAgentInitialized()
{
    Debug.Log("✓ Agent initialized");
    
    // Load conversation history
    LoadConversationHistory();
    
    // Register custom tools
    RegisterTools();
    
    // Setup UI
    SetupUI();
}

async void LoadConversationHistory()
{
    var conversation = await agent.ConversationController.LoadAsync("last_session");
    
    if (conversation != null)
    {
        Debug.Log($"Loaded conversation: {conversation.Messages.Count} messages");
    }
}

void RegisterTools()
{
    // Register custom tool executors
    agent.ToolController.RegisterExecutor(new MyCustomToolExecutor());
}
```

## Start/Stop Hooks

### Agent Start

```csharp
public class AgentStartHook : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.onAgentStarted.AddListener(OnAgentStarted);
    }
    
    void OnAgentStarted()
    {
        Debug.Log("▶️ Agent started");
        
        // Show agent UI
        ShowAgentUI(true);
        
        // Start monitoring
        StartMonitoring();
        
        // Send welcome message
        SendWelcomeMessage();
    }
    
    void ShowAgentUI(bool show)
    {
        // Update UI state
    }
    
    void StartMonitoring()
    {
        // Start performance monitoring
    }
    
    async void SendWelcomeMessage()
    {
        await agent.SendAsync("Hello! How can I help you today?");
    }
}
```

### Agent Stop

```csharp
void Start()
{
    agent.onAgentStopped.AddListener(OnAgentStopped);
}

void OnAgentStopped()
{
    Debug.Log("⏹️ Agent stopped");
    
    // Save conversation
    SaveConversation();
    
    // Stop monitoring
    StopMonitoring();
    
    // Hide UI
    ShowAgentUI(false);
    
    // Clean up resources
    CleanupResources();
}

async void SaveConversation()
{
    var conversation = agent.ConversationController.CurrentConversation;
    
    if (conversation != null)
    {
        await conversation.SaveAsync("last_session");
        Debug.Log("💾 Conversation saved");
    }
}

void CleanupResources()
{
    // Release resources
}
```

## Request Hooks

### Before Request

```csharp
public class RequestHooks : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.onBeforeRequest.AddListener(OnBeforeRequest);
        agent.onAfterRequest.AddListener(OnAfterRequest);
    }
    
    void OnBeforeRequest(string userMessage)
    {
        Debug.Log($"📤 Sending: {userMessage}");
        
        // Show loading indicator
        ShowLoadingIndicator(true);
        
        // Log request
        LogRequest(userMessage);
        
        // Update analytics
        TrackRequest();
    }
    
    void ShowLoadingIndicator(bool show)
    {
        // Update UI
    }
    
    void LogRequest(string message)
    {
        // Log to analytics
    }
    
    void TrackRequest()
    {
        // Track request metrics
    }
}
```

### After Request

```csharp
void OnAfterRequest(AgentResponse response)
{
    Debug.Log($"📥 Received: {response.TextContent}");
    
    // Hide loading indicator
    ShowLoadingIndicator(false);
    
    // Log response
    LogResponse(response);
    
    // Update UI
    UpdateUI(response);
}

void LogResponse(AgentResponse response)
{
    Debug.Log($"Tokens used: {response.UsageInfo.TotalTokens}");
    Debug.Log($"Response time: {response.ResponseTime}ms");
}

void UpdateUI(AgentResponse response)
{
    // Update conversation UI
}
```

## Error Hooks

### Error Handling

```csharp
public class ErrorHooks : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int maxRetries = 3;
    
    private int retryCount;
    
    void Start()
    {
        agent.onAgentError.AddListener(OnAgentError);
        agent.onRequestError.AddListener(OnRequestError);
    }
    
    void OnAgentError(string error)
    {
        Debug.LogError($"❌ Agent error: {error}");
        
        // Show error message
        ShowErrorMessage(error);
        
        // Log error
        LogError(error);
        
        // Attempt recovery
        AttemptRecovery();
    }
    
    void OnRequestError(string error)
    {
        Debug.LogError($"❌ Request error: {error}");
        
        if (ShouldRetry(error))
        {
            RetryRequest();
        }
        else
        {
            ShowErrorMessage("Request failed. Please try again.");
        }
    }
    
    bool ShouldRetry(string error)
    {
        if (retryCount >= maxRetries)
            return false;
        
        // Retry on transient errors
        return error.Contains("timeout") || 
               error.Contains("rate_limit") ||
               error.Contains("network");
    }
    
    async void RetryRequest()
    {
        retryCount++;
        Debug.Log($"🔄 Retrying ({retryCount}/{maxRetries})...");
        
        // Wait before retry
        await UniTask.Delay(TimeSpan.FromSeconds(retryCount * 2));
        
        // Retry last request
        await agent.RetryLastRequestAsync();
    }
    
    void AttemptRecovery()
    {
        // Try to recover agent
        Debug.Log("Attempting recovery...");
        
        agent.Stop();
        agent.Initialize();
        agent.Start();
    }
    
    void ShowErrorMessage(string message)
    {
        // Update UI
    }
    
    void LogError(string error)
    {
        // Log to analytics
    }
}
```

## Conversation Hooks

### Message Events

```csharp
public class ConversationHooks : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.onMessageAdded.AddListener(OnMessageAdded);
        agent.onMessageRemoved.AddListener(OnMessageRemoved);
        agent.onConversationCleared.AddListener(OnConversationCleared);
    }
    
    void OnMessageAdded(ConversationMessage message)
    {
        Debug.Log($"➕ Message added: {message.Role}");
        
        // Update UI
        UpdateMessageUI(message);
        
        // Save to history
        SaveMessageToHistory(message);
    }
    
    void OnMessageRemoved(string messageId)
    {
        Debug.Log($"➖ Message removed: {messageId}");
        
        // Update UI
        RemoveMessageFromUI(messageId);
    }
    
    void OnConversationCleared()
    {
        Debug.Log("🗑️ Conversation cleared");
        
        // Clear UI
        ClearConversationUI();
    }
    
    void UpdateMessageUI(ConversationMessage message)
    {
        // Add message to chat UI
    }
    
    void SaveMessageToHistory(ConversationMessage message)
    {
        // Save to persistent storage
    }
    
    void RemoveMessageFromUI(string messageId)
    {
        // Remove from chat UI
    }
    
    void ClearConversationUI()
    {
        // Clear all messages from UI
    }
}
```

## Performance Monitoring

### Track Performance Metrics

```csharp
public class PerformanceMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private List<float> responseTimes = new();
    private int totalRequests;
    private int totalTokens;
    
    void Start()
    {
        agent.onAfterRequest.AddListener(OnAfterRequest);
    }
    
    void OnAfterRequest(AgentResponse response)
    {
        // Track metrics
        responseTimes.Add(response.ResponseTime);
        totalRequests++;
        totalTokens += response.UsageInfo.TotalTokens;
        
        // Log stats every 10 requests
        if (totalRequests % 10 == 0)
        {
            LogStatistics();
        }
    }
    
    void LogStatistics()
    {
        float avgResponseTime = responseTimes.Average();
        float avgTokensPerRequest = (float)totalTokens / totalRequests;
        
        Debug.Log($"📊 Statistics:");
        Debug.Log($"  Total requests: {totalRequests}");
        Debug.Log($"  Avg response time: {avgResponseTime:F2}ms");
        Debug.Log($"  Avg tokens per request: {avgTokensPerRequest:F0}");
        Debug.Log($"  Total tokens used: {totalTokens}");
    }
}
```

## Custom Hook Manager

### Centralized Hook Management

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class AgentHookManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("Settings")]
    [SerializeField] private bool enableLogging = true;
    [SerializeField] private bool enableAnalytics = true;
    [SerializeField] private bool autoSaveConversation = true;
    
    private int requestCount;
    private float totalResponseTime;
    
    void Start()
    {
        RegisterAllHooks();
    }
    
    void RegisterAllHooks()
    {
        // Lifecycle hooks
        agent.onBeforeInitialize.AddListener(OnBeforeInitialize);
        agent.onAgentInitialized.AddListener(OnAgentInitialized);
        agent.onAgentStarted.AddListener(OnAgentStarted);
        agent.onAgentStopped.AddListener(OnAgentStopped);
        
        // Request hooks
        agent.onBeforeRequest.AddListener(OnBeforeRequest);
        agent.onAfterRequest.AddListener(OnAfterRequest);
        
        // Error hooks
        agent.onAgentError.AddListener(OnAgentError);
        agent.onRequestError.AddListener(OnRequestError);
        
        // Conversation hooks
        agent.onMessageAdded.AddListener(OnMessageAdded);
        agent.onConversationCleared.AddListener(OnConversationCleared);
        
        Debug.Log("✓ Agent hooks registered");
    }
    
    // Lifecycle Hooks
    
    void OnBeforeInitialize()
    {
        if (enableLogging)
            Debug.Log("🔧 Initializing agent...");
        
        LoadSavedPreferences();
    }
    
    void OnAgentInitialized()
    {
        if (enableLogging)
            Debug.Log("✓ Agent initialized");
        
        if (autoSaveConversation)
            LoadLastConversation().Forget();
    }
    
    void OnAgentStarted()
    {
        if (enableLogging)
            Debug.Log("▶️ Agent started");
        
        if (enableAnalytics)
            TrackAgentStart();
    }
    
    void OnAgentStopped()
    {
        if (enableLogging)
            Debug.Log("⏹️ Agent stopped");
        
        if (autoSaveConversation)
            SaveCurrentConversation().Forget();
        
        if (enableAnalytics)
            TrackAgentStop();
    }
    
    // Request Hooks
    
    void OnBeforeRequest(string message)
    {
        if (enableLogging)
            Debug.Log($"📤 Request: {message.Substring(0, Math.Min(50, message.Length))}...");
        
        requestCount++;
        
        if (enableAnalytics)
            TrackRequest(message);
    }
    
    void OnAfterRequest(AgentResponse response)
    {
        if (enableLogging)
        {
            Debug.Log($"📥 Response: {response.TextContent.Substring(0, Math.Min(50, response.TextContent.Length))}...");
            Debug.Log($"⏱️ Response time: {response.ResponseTime}ms");
            Debug.Log($"🎫 Tokens: {response.UsageInfo.TotalTokens}");
        }
        
        totalResponseTime += response.ResponseTime;
        
        if (enableAnalytics)
            TrackResponse(response);
    }
    
    // Error Hooks
    
    void OnAgentError(string error)
    {
        Debug.LogError($"❌ Agent error: {error}");
        
        if (enableAnalytics)
            TrackError("agent", error);
        
        ShowErrorNotification("Agent error occurred");
    }
    
    void OnRequestError(string error)
    {
        Debug.LogError($"❌ Request error: {error}");
        
        if (enableAnalytics)
            TrackError("request", error);
        
        ShowErrorNotification("Request failed");
    }
    
    // Conversation Hooks
    
    void OnMessageAdded(ConversationMessage message)
    {
        if (enableLogging)
            Debug.Log($"➕ Message added: {message.Role}");
        
        if (autoSaveConversation)
            AutoSaveConversation().Forget();
    }
    
    void OnConversationCleared()
    {
        if (enableLogging)
            Debug.Log("🗑️ Conversation cleared");
    }
    
    // Helper Methods
    
    void LoadSavedPreferences()
    {
        string savedModel = PlayerPrefs.GetString("PreferredModel", "gpt-4o");
        agent.ParametersController.SetModel(savedModel);
    }
    
    async UniTaskVoid LoadLastConversation()
    {
        try
        {
            var conversation = await agent.ConversationController.LoadAsync("last_session");
            
            if (conversation != null)
            {
                Debug.Log($"📂 Loaded conversation: {conversation.Messages.Count} messages");
            }
        }
        catch (Exception ex)
        {
            Debug.LogError($"Failed to load conversation: {ex.Message}");
        }
    }
    
    async UniTaskVoid SaveCurrentConversation()
    {
        try
        {
            var conversation = agent.ConversationController.CurrentConversation;
            
            if (conversation != null && conversation.Messages.Count > 0)
            {
                await conversation.SaveAsync("last_session");
                Debug.Log($"💾 Conversation saved: {conversation.Messages.Count} messages");
            }
        }
        catch (Exception ex)
        {
            Debug.LogError($"Failed to save conversation: {ex.Message}");
        }
    }
    
    async UniTaskVoid AutoSaveConversation()
    {
        // Debounce auto-save
        await UniTask.Delay(TimeSpan.FromSeconds(5));
        await SaveCurrentConversation();
    }
    
    void TrackAgentStart()
    {
        // Send analytics event
    }
    
    void TrackAgentStop()
    {
        // Send analytics event with session stats
        float avgResponseTime = requestCount > 0 ? totalResponseTime / requestCount : 0;
        Debug.Log($"📊 Session stats - Requests: {requestCount}, Avg time: {avgResponseTime:F2}ms");
    }
    
    void TrackRequest(string message)
    {
        // Track request in analytics
    }
    
    void TrackResponse(AgentResponse response)
    {
        // Track response metrics
    }
    
    void TrackError(string type, string error)
    {
        // Track error in analytics
    }
    
    void ShowErrorNotification(string message)
    {
        // Show error UI
    }
    
    void OnDestroy()
    {
        // Cleanup
        if (agent != null)
        {
            agent.onBeforeInitialize.RemoveListener(OnBeforeInitialize);
            agent.onAgentInitialized.RemoveListener(OnAgentInitialized);
            agent.onAgentStarted.RemoveListener(OnAgentStarted);
            agent.onAgentStopped.RemoveListener(OnAgentStopped);
            agent.onBeforeRequest.RemoveListener(OnBeforeRequest);
            agent.onAfterRequest.RemoveListener(OnAfterRequest);
            agent.onAgentError.RemoveListener(OnAgentError);
            agent.onRequestError.RemoveListener(OnRequestError);
            agent.onMessageAdded.RemoveListener(OnMessageAdded);
            agent.onConversationCleared.RemoveListener(OnConversationCleared);
        }
    }
}
```

## Next Steps

- [Text Delta Events](text-delta.md)
- [Tool Call Events](tool-calls.md)
- [Audio Events](audio.md)
- [Status Events](status.md)
- [Error Handling Events](error-handling.md)
