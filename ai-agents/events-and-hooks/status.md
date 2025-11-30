# Status Events

Monitor agent status changes and state transitions.

## Overview

Status Events provide:

- Agent state monitoring
- Connection status tracking
- Activity indicators
- Health checks
- Performance metrics

## Basic Setup

### Subscribe to Status Events

```csharp
public class StatusEventListener : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.onStatusChanged.AddListener(OnStatusChanged);
        agent.onConnectionStatusChanged.AddListener(OnConnectionStatusChanged);
        agent.onActivityChanged.AddListener(OnActivityChanged);
    }
    
    void OnStatusChanged(AgentStatus status)
    {
        Debug.Log($"Status changed: {status}");
    }
    
    void OnConnectionStatusChanged(ConnectionStatus status)
    {
        Debug.Log($"Connection: {status}");
    }
    
    void OnActivityChanged(AgentActivity activity)
    {
        Debug.Log($"Activity: {activity}");
    }
}
```

## Agent Status

### Monitor Agent State

```csharp
public class AgentStatusMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text statusText;
    [SerializeField] private Image statusIndicator;
    
    void Start()
    {
        agent.onStatusChanged.AddListener(OnStatusChanged);
    }
    
    void OnStatusChanged(AgentStatus status)
    {
        statusText.text = status.ToString();
        
        switch (status)
        {
            case AgentStatus.Uninitialized:
                statusIndicator.color = Color.gray;
                Debug.Log("⚪ Agent uninitialized");
                break;
                
            case AgentStatus.Initializing:
                statusIndicator.color = Color.yellow;
                Debug.Log("🟡 Agent initializing...");
                break;
                
            case AgentStatus.Ready:
                statusIndicator.color = Color.green;
                Debug.Log("🟢 Agent ready");
                break;
                
            case AgentStatus.Busy:
                statusIndicator.color = Color.orange;
                Debug.Log("🟠 Agent busy");
                break;
                
            case AgentStatus.Error:
                statusIndicator.color = Color.red;
                Debug.Log("🔴 Agent error");
                break;
                
            case AgentStatus.Stopped:
                statusIndicator.color = Color.gray;
                Debug.Log("⚪ Agent stopped");
                break;
        }
    }
}

public enum AgentStatus
{
    Uninitialized,
    Initializing,
    Ready,
    Busy,
    Error,
    Stopped
}
```

## Connection Status

### Track API Connection

```csharp
public class ConnectionMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Image connectionIndicator;
    [SerializeField] private TMP_Text connectionText;
    
    void Start()
    {
        agent.onConnectionStatusChanged.AddListener(OnConnectionStatusChanged);
        agent.onPingCompleted.AddListener(OnPingCompleted);
    }
    
    void OnConnectionStatusChanged(ConnectionStatus status)
    {
        connectionText.text = status.ToString();
        
        switch (status)
        {
            case ConnectionStatus.Disconnected:
                connectionIndicator.color = Color.gray;
                Debug.Log("⚪ Disconnected");
                break;
                
            case ConnectionStatus.Connecting:
                connectionIndicator.color = Color.yellow;
                Debug.Log("🟡 Connecting...");
                StartConnectingAnimation();
                break;
                
            case ConnectionStatus.Connected:
                connectionIndicator.color = Color.green;
                Debug.Log("🟢 Connected");
                StopConnectingAnimation();
                break;
                
            case ConnectionStatus.Reconnecting:
                connectionIndicator.color = Color.orange;
                Debug.Log("🟠 Reconnecting...");
                StartConnectingAnimation();
                break;
                
            case ConnectionStatus.Error:
                connectionIndicator.color = Color.red;
                Debug.Log("🔴 Connection error");
                StopConnectingAnimation();
                break;
        }
    }
    
    void OnPingCompleted(float latency)
    {
        Debug.Log($"Ping: {latency}ms");
        
        // Show latency indicator
        if (latency < 100)
        {
            connectionIndicator.color = Color.green;
        }
        else if (latency < 300)
        {
            connectionIndicator.color = Color.yellow;
        }
        else
        {
            connectionIndicator.color = Color.orange;
        }
    }
    
    void StartConnectingAnimation()
    {
        // Animate connection indicator
    }
    
    void StopConnectingAnimation()
    {
        // Stop animation
    }
}

public enum ConnectionStatus
{
    Disconnected,
    Connecting,
    Connected,
    Reconnecting,
    Error
}
```

## Activity Tracking

### Monitor Agent Activity

```csharp
public class ActivityMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text activityText;
    [SerializeField] private GameObject activityIndicator;
    
    void Start()
    {
        agent.onActivityChanged.AddListener(OnActivityChanged);
    }
    
    void OnActivityChanged(AgentActivity activity)
    {
        activityText.text = GetActivityDescription(activity);
        activityIndicator.SetActive(activity != AgentActivity.Idle);
        
        Debug.Log($"Activity: {activity}");
    }
    
    string GetActivityDescription(AgentActivity activity)
    {
        return activity switch
        {
            AgentActivity.Idle => "Idle",
            AgentActivity.Thinking => "🤔 Thinking...",
            AgentActivity.Streaming => "📡 Streaming response...",
            AgentActivity.ExecutingTool => "🔧 Using tool...",
            AgentActivity.Recording => "🎤 Recording...",
            AgentActivity.Transcribing => "📝 Transcribing...",
            AgentActivity.GeneratingSpeech => "🔊 Generating speech...",
            AgentActivity.PlayingAudio => "▶️ Playing audio...",
            _ => "Unknown"
        };
    }
}

public enum AgentActivity
{
    Idle,
    Thinking,
    Streaming,
    ExecutingTool,
    Recording,
    Transcribing,
    GeneratingSpeech,
    PlayingAudio
}
```

## Health Monitoring

### Track Agent Health

```csharp
public class HealthMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private float healthCheckInterval = 60f;
    
    private int consecutiveErrors;
    private float lastHealthCheck;
    
    void Start()
    {
        agent.onHealthCheckCompleted.AddListener(OnHealthCheckCompleted);
        agent.onRequestError.AddListener(OnRequestError);
        agent.onRequestCompleted.AddListener(OnRequestCompleted);
    }
    
    void Update()
    {
        if (Time.time - lastHealthCheck >= healthCheckInterval)
        {
            PerformHealthCheck().Forget();
        }
    }
    
    async UniTaskVoid PerformHealthCheck()
    {
        lastHealthCheck = Time.time;
        
        Debug.Log("🏥 Performing health check...");
        
        bool isHealthy = await agent.CheckHealthAsync();
        
        if (isHealthy)
        {
            Debug.Log("✓ Agent healthy");
            consecutiveErrors = 0;
        }
        else
        {
            Debug.LogWarning("⚠️ Agent health check failed");
            
            if (consecutiveErrors >= 3)
            {
                Debug.LogError("❌ Agent unhealthy - attempting recovery");
                AttemptRecovery();
            }
        }
    }
    
    void OnHealthCheckCompleted(bool isHealthy)
    {
        if (isHealthy)
        {
            Debug.Log("✓ Health check passed");
        }
        else
        {
            Debug.LogWarning("⚠️ Health check failed");
        }
    }
    
    void OnRequestError(string error)
    {
        consecutiveErrors++;
        
        if (consecutiveErrors >= 3)
        {
            Debug.LogError("❌ Multiple consecutive errors detected");
            AttemptRecovery();
        }
    }
    
    void OnRequestCompleted()
    {
        consecutiveErrors = 0;
    }
    
    void AttemptRecovery()
    {
        Debug.Log("🔄 Attempting recovery...");
        
        agent.Stop();
        agent.Initialize();
        agent.Start();
        
        consecutiveErrors = 0;
    }
}
```

## Performance Metrics

### Track Performance Stats

```csharp
public class PerformanceMetrics : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text metricsText;
    
    private int totalRequests;
    private int successfulRequests;
    private int failedRequests;
    private float totalResponseTime;
    private int totalTokens;
    
    void Start()
    {
        agent.onAfterRequest.AddListener(OnAfterRequest);
        agent.onRequestError.AddListener(OnRequestError);
        
        InvokeRepeating(nameof(UpdateMetricsDisplay), 1f, 1f);
    }
    
    void OnAfterRequest(AgentResponse response)
    {
        totalRequests++;
        successfulRequests++;
        totalResponseTime += response.ResponseTime;
        totalTokens += response.UsageInfo.TotalTokens;
    }
    
    void OnRequestError(string error)
    {
        totalRequests++;
        failedRequests++;
    }
    
    void UpdateMetricsDisplay()
    {
        float avgResponseTime = successfulRequests > 0 ? 
            totalResponseTime / successfulRequests : 0;
        
        float avgTokens = successfulRequests > 0 ? 
            (float)totalTokens / successfulRequests : 0;
        
        float successRate = totalRequests > 0 ? 
            (float)successfulRequests / totalRequests * 100 : 0;
        
        metricsText.text = $@"
📊 Performance Metrics
Total Requests: {totalRequests}
Success Rate: {successRate:F1}%
Avg Response Time: {avgResponseTime:F0}ms
Avg Tokens: {avgTokens:F0}
Total Tokens: {totalTokens}
        ";
    }
}
```

## Rate Limit Tracking

### Monitor Rate Limits

```csharp
public class RateLimitMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text rateLimitText;
    [SerializeField] private Slider rateLimitBar;
    
    void Start()
    {
        agent.onRateLimitInfoReceived.AddListener(OnRateLimitInfoReceived);
        agent.onRateLimitApproaching.AddListener(OnRateLimitApproaching);
        agent.onRateLimitReached.AddListener(OnRateLimitReached);
    }
    
    void OnRateLimitInfoReceived(RateLimitInfo info)
    {
        float usage = (float)info.Used / info.Limit;
        
        rateLimitBar.value = usage;
        rateLimitText.text = $"{info.Used}/{info.Limit} requests";
        
        // Color code based on usage
        if (usage < 0.7f)
        {
            rateLimitBar.fillRect.GetComponent<Image>().color = Color.green;
        }
        else if (usage < 0.9f)
        {
            rateLimitBar.fillRect.GetComponent<Image>().color = Color.yellow;
        }
        else
        {
            rateLimitBar.fillRect.GetComponent<Image>().color = Color.red;
        }
    }
    
    void OnRateLimitApproaching(RateLimitInfo info)
    {
        Debug.LogWarning($"⚠️ Rate limit approaching: {info.Used}/{info.Limit}");
        ShowWarning("Rate limit approaching. Requests may slow down.");
    }
    
    void OnRateLimitReached(RateLimitInfo info)
    {
        Debug.LogError($"❌ Rate limit reached: {info.Used}/{info.Limit}");
        Debug.Log($"Resets in: {info.ResetTime}");
        
        ShowError($"Rate limit reached. Resets in {info.ResetTime}.");
    }
    
    void ShowWarning(string message)
    {
        // Show warning UI
    }
    
    void ShowError(string message)
    {
        // Show error UI
    }
}

public class RateLimitInfo
{
    public int Used { get; set; }
    public int Limit { get; set; }
    public TimeSpan ResetTime { get; set; }
}
```

## Token Usage Tracking

### Monitor Token Consumption

```csharp
public class TokenUsageMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text usageText;
    [SerializeField] private int budgetLimit = 10000;
    
    private int sessionTokens;
    private int totalTokens;
    
    void Start()
    {
        agent.onAfterRequest.AddListener(OnAfterRequest);
        
        totalTokens = PlayerPrefs.GetInt("TotalTokens", 0);
    }
    
    void OnAfterRequest(AgentResponse response)
    {
        int tokensUsed = response.UsageInfo.TotalTokens;
        
        sessionTokens += tokensUsed;
        totalTokens += tokensUsed;
        
        Debug.Log($"🎫 Tokens used: {tokensUsed} (Session: {sessionTokens}, Total: {totalTokens})");
        
        UpdateDisplay();
        CheckBudget();
        
        PlayerPrefs.SetInt("TotalTokens", totalTokens);
    }
    
    void UpdateDisplay()
    {
        float budgetUsage = (float)sessionTokens / budgetLimit * 100;
        
        usageText.text = $@"
🎫 Token Usage
Session: {sessionTokens:N0}
Total: {totalTokens:N0}
Budget: {sessionTokens:N0}/{budgetLimit:N0} ({budgetUsage:F1}%)
        ";
    }
    
    void CheckBudget()
    {
        float budgetUsage = (float)sessionTokens / budgetLimit;
        
        if (budgetUsage >= 1.0f)
        {
            Debug.LogWarning("❌ Budget limit reached!");
            ShowBudgetWarning("Budget limit reached. Consider upgrading.");
        }
        else if (budgetUsage >= 0.9f)
        {
            Debug.LogWarning("⚠️ 90% of budget used");
            ShowBudgetWarning("90% of budget used.");
        }
    }
    
    void ShowBudgetWarning(string message)
    {
        // Show warning UI
    }
}
```

## Network Status

### Monitor Network Conditions

```csharp
public class NetworkMonitor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text networkText;
    
    void Start()
    {
        agent.onNetworkStatusChanged.AddListener(OnNetworkStatusChanged);
        
        InvokeRepeating(nameof(CheckNetwork), 1f, 5f);
    }
    
    void CheckNetwork()
    {
        NetworkReachability reachability = Application.internetReachability;
        
        OnNetworkStatusChanged(reachability);
    }
    
    void OnNetworkStatusChanged(NetworkReachability status)
    {
        switch (status)
        {
            case NetworkReachability.NotReachable:
                networkText.text = "🔴 No Network";
                networkText.color = Color.red;
                Debug.LogWarning("No network connection");
                break;
                
            case NetworkReachability.ReachableViaCarrierDataNetwork:
                networkText.text = "🟡 Cellular";
                networkText.color = Color.yellow;
                Debug.Log("Connected via cellular");
                break;
                
            case NetworkReachability.ReachableViaLocalAreaNetwork:
                networkText.text = "🟢 WiFi";
                networkText.color = Color.green;
                Debug.Log("Connected via WiFi");
                break;
        }
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class StatusEventManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("Status UI")]
    [SerializeField] private TMP_Text statusText;
    [SerializeField] private Image statusIndicator;
    [SerializeField] private TMP_Text activityText;
    [SerializeField] private Image connectionIndicator;
    
    [Header("Metrics UI")]
    [SerializeField] private TMP_Text metricsText;
    [SerializeField] private Slider rateLimitBar;
    [SerializeField] private TMP_Text tokenUsageText;
    
    [Header("Settings")]
    [SerializeField] private float healthCheckInterval = 60f;
    
    private int totalRequests;
    private int successfulRequests;
    private float totalResponseTime;
    private int totalTokens;
    
    void Start()
    {
        RegisterAllEvents();
        StartHealthChecks();
    }
    
    void RegisterAllEvents()
    {
        // Status events
        agent.onStatusChanged.AddListener(OnStatusChanged);
        agent.onConnectionStatusChanged.AddListener(OnConnectionStatusChanged);
        agent.onActivityChanged.AddListener(OnActivityChanged);
        
        // Performance events
        agent.onAfterRequest.AddListener(OnAfterRequest);
        agent.onRequestError.AddListener(OnRequestError);
        
        // Rate limit events
        agent.onRateLimitInfoReceived.AddListener(OnRateLimitInfoReceived);
        
        Debug.Log("✓ Status events registered");
    }
    
    void StartHealthChecks()
    {
        InvokeRepeating(nameof(PerformHealthCheck), healthCheckInterval, healthCheckInterval);
    }
    
    // Status Events
    
    void OnStatusChanged(AgentStatus status)
    {
        statusText.text = status.ToString();
        
        Color indicatorColor = status switch
        {
            AgentStatus.Uninitialized => Color.gray,
            AgentStatus.Initializing => Color.yellow,
            AgentStatus.Ready => Color.green,
            AgentStatus.Busy => new Color(1f, 0.5f, 0f),
            AgentStatus.Error => Color.red,
            AgentStatus.Stopped => Color.gray,
            _ => Color.white
        };
        
        statusIndicator.color = indicatorColor;
        
        Debug.Log($"Status: {status}");
    }
    
    void OnConnectionStatusChanged(ConnectionStatus status)
    {
        Color indicatorColor = status switch
        {
            ConnectionStatus.Disconnected => Color.gray,
            ConnectionStatus.Connecting => Color.yellow,
            ConnectionStatus.Connected => Color.green,
            ConnectionStatus.Reconnecting => new Color(1f, 0.5f, 0f),
            ConnectionStatus.Error => Color.red,
            _ => Color.white
        };
        
        connectionIndicator.color = indicatorColor;
        
        Debug.Log($"Connection: {status}");
    }
    
    void OnActivityChanged(AgentActivity activity)
    {
        activityText.text = GetActivityDescription(activity);
        
        Debug.Log($"Activity: {activity}");
    }
    
    string GetActivityDescription(AgentActivity activity)
    {
        return activity switch
        {
            AgentActivity.Idle => "Idle",
            AgentActivity.Thinking => "🤔 Thinking...",
            AgentActivity.Streaming => "📡 Streaming...",
            AgentActivity.ExecutingTool => "🔧 Using tool...",
            AgentActivity.Recording => "🎤 Recording...",
            AgentActivity.Transcribing => "📝 Transcribing...",
            AgentActivity.GeneratingSpeech => "🔊 Generating speech...",
            AgentActivity.PlayingAudio => "▶️ Playing...",
            _ => "Unknown"
        };
    }
    
    // Performance Events
    
    void OnAfterRequest(AgentResponse response)
    {
        totalRequests++;
        successfulRequests++;
        totalResponseTime += response.ResponseTime;
        totalTokens += response.UsageInfo.TotalTokens;
        
        UpdateMetricsDisplay();
        UpdateTokenUsageDisplay();
    }
    
    void OnRequestError(string error)
    {
        totalRequests++;
        UpdateMetricsDisplay();
    }
    
    void OnRateLimitInfoReceived(RateLimitInfo info)
    {
        float usage = (float)info.Used / info.Limit;
        rateLimitBar.value = usage;
        
        Color barColor = usage < 0.7f ? Color.green :
                        usage < 0.9f ? Color.yellow :
                        Color.red;
        
        rateLimitBar.fillRect.GetComponent<Image>().color = barColor;
    }
    
    // Display Updates
    
    void UpdateMetricsDisplay()
    {
        float avgResponseTime = successfulRequests > 0 ? 
            totalResponseTime / successfulRequests : 0;
        
        float successRate = totalRequests > 0 ? 
            (float)successfulRequests / totalRequests * 100 : 0;
        
        metricsText.text = $@"
📊 Metrics
Requests: {totalRequests}
Success: {successRate:F1}%
Avg Time: {avgResponseTime:F0}ms
        ";
    }
    
    void UpdateTokenUsageDisplay()
    {
        float avgTokens = successfulRequests > 0 ? 
            (float)totalTokens / successfulRequests : 0;
        
        tokenUsageText.text = $@"
🎫 Tokens
Total: {totalTokens:N0}
Avg: {avgTokens:F0}
        ";
    }
    
    // Health Check
    
    async void PerformHealthCheck()
    {
        Debug.Log("🏥 Health check...");
        
        bool isHealthy = await agent.CheckHealthAsync();
        
        if (isHealthy)
        {
            Debug.Log("✓ Agent healthy");
        }
        else
        {
            Debug.LogWarning("⚠️ Health check failed");
        }
    }
    
    void OnDestroy()
    {
        if (agent != null)
        {
            agent.onStatusChanged.RemoveListener(OnStatusChanged);
            agent.onConnectionStatusChanged.RemoveListener(OnConnectionStatusChanged);
            agent.onActivityChanged.RemoveListener(OnActivityChanged);
            agent.onAfterRequest.RemoveListener(OnAfterRequest);
            agent.onRequestError.RemoveListener(OnRequestError);
            agent.onRateLimitInfoReceived.RemoveListener(OnRateLimitInfoReceived);
        }
    }
}
```

## Next Steps

- [Agent Lifecycle Hooks](agent-hooks.md)
- [Text Delta Events](text-delta.md)
- [Tool Call Events](tool-calls.md)
- [Audio Events](audio.md)
- [Error Handling Events](error-handling.md)
