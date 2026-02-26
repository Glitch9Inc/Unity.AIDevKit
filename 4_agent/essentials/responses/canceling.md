---
icon: ban
---

# Canceling Requests

Cancel in-progress agent requests.

## Overview

Cancel requests when:

- User cancels the operation
- Timeout occurs
- New request needs to start
- User navigates away

## Basic Cancellation

### Using Agent

```csharp
// Start a request
var sendTask = agent.SendAsync("Write a long story...");

// Cancel it
await agent.CancelAsync();

// The sendTask will throw OperationCanceledException
try
{
    await sendTask;
}
catch (OperationCanceledException)
{
    Debug.Log("Request was cancelled");
}
```

### With CancellationToken

```csharp
CancellationTokenSource cts = new CancellationTokenSource();

// Start request with token
var task = agent.SendAsync("Long message", ct: cts.Token);

// Cancel after 5 seconds
await UniTask.Delay(5000);
cts.Cancel();

try
{
    await task;
}
catch (OperationCanceledException)
{
    Debug.Log("Timed out");
}
```

## Cancel Button

### Simple Implementation

```csharp
public class CancelableChat : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject cancelButton;
    
    private CancellationTokenSource cts;
    
    public async void SendMessage(string message)
    {
        cts = new CancellationTokenSource();
        cancelButton.SetActive(true);
        
        try
        {
            Response response = await agent.SendAsync(message, ct: cts.Token);
            Debug.Log(response.Text);
        }
        catch (OperationCanceledException)
        {
            Debug.Log("Cancelled by user");
        }
        finally
        {
            cancelButton.SetActive(false);
            cts?.Dispose();
            cts = null;
        }
    }
    
    public void CancelRequest()
    {
        cts?.Cancel();
    }
}
```

### With UI Feedback

```csharp
public class CancelWithFeedback : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Button sendButton;
    [SerializeField] private Button cancelButton;
    [SerializeField] private TMP_Text statusText;
    
    private CancellationTokenSource cts;
    
    public async void SendMessage(string message)
    {
        cts = new CancellationTokenSource();
        
        // Update UI
        sendButton.interactable = false;
        cancelButton.gameObject.SetActive(true);
        statusText.text = "Sending...";
        
        try
        {
            Response response = await agent.SendAsync(message, ct: cts.Token);
            
            statusText.text = "Complete";
            Debug.Log(response.Text);
        }
        catch (OperationCanceledException)
        {
            statusText.text = "Cancelled";
            Debug.Log("Request cancelled");
        }
        catch (Exception ex)
        {
            statusText.text = $"Error: {ex.Message}";
            Debug.LogError(ex);
        }
        finally
        {
            sendButton.interactable = true;
            cancelButton.gameObject.SetActive(false);
            
            cts?.Dispose();
            cts = null;
        }
    }
    
    public void Cancel()
    {
        cts?.Cancel();
        statusText.text = "Cancelling...";
    }
}
```

## Timeout

### Request Timeout

```csharp
public async UniTask<Response> SendWithTimeout(
    string message,
    int timeoutSeconds = 30)
{
    CancellationTokenSource cts = new CancellationTokenSource();
    cts.CancelAfter(TimeSpan.FromSeconds(timeoutSeconds));
    
    try
    {
        return await agent.SendAsync(message, ct: cts.Token);
    }
    catch (OperationCanceledException)
    {
        throw new TimeoutException($"Request timed out after {timeoutSeconds}s");
    }
    finally
    {
        cts.Dispose();
    }
}
```

### Configurable Timeout

```csharp
public class TimeoutManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private float timeoutSeconds = 30f;
    
    public async UniTask<Response> SendWithTimeout(string message)
    {
        CancellationTokenSource cts = new CancellationTokenSource();
        
        // Start timeout countdown
        var timeoutTask = UniTask.Delay(
            TimeSpan.FromSeconds(timeoutSeconds),
            cancellationToken: cts.Token
        );
        
        // Start request
        var requestTask = agent.SendAsync(message, ct: cts.Token);
        
        // Wait for either to complete
        var completedTask = await UniTask.WhenAny(requestTask, timeoutTask);
        
        if (completedTask == 1)
        {
            // Timeout occurred
            cts.Cancel();
            throw new TimeoutException("Request timed out");
        }
        
        // Request completed
        cts.Cancel(); // Cancel timeout
        return await requestTask;
    }
}
```

## Streaming Cancellation

### Cancel Streaming

```csharp
public class StreamingCancellation : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text chatText;
    
    private CancellationTokenSource cts;
    private StringBuilder currentResponse = new();
    
    void Start()
    {
        agent.Stream = true;
        agent.onTextDelta.AddListener(OnTextDelta);
    }
    
    public async void SendMessage(string message)
    {
        cts = new CancellationTokenSource();
        currentResponse.Clear();
        
        try
        {
            await agent.SendAsync(message, ct: cts.Token);
        }
        catch (OperationCanceledException)
        {
            Debug.Log("Streaming cancelled");
            chatText.text += "\n[Cancelled]";
        }
        finally
        {
            cts?.Dispose();
            cts = null;
        }
    }
    
    void OnTextDelta(string delta)
    {
        currentResponse.Append(delta);
        chatText.text = currentResponse.ToString();
    }
    
    public void Cancel()
    {
        cts?.Cancel();
    }
}
```

### Graceful Streaming Stop

```csharp
public async void StopStreaming()
{
    if (agent.Status != AgentStatus.Responding)
    {
        return;
    }
    
    // Cancel the request
    await agent.CancelAsync();
    
    // Wait for cancellation to complete
    await UniTask.WaitUntil(() => agent.Status == AgentStatus.Ready);
    
    Debug.Log("Streaming stopped gracefully");
}
```

## Multiple Requests

### Cancel Previous Request

```csharp
public class SingleRequestManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private CancellationTokenSource currentCts;
    
    public async void SendMessage(string message)
    {
        // Cancel any existing request
        currentCts?.Cancel();
        currentCts?.Dispose();
        
        // Create new token
        currentCts = new CancellationTokenSource();
        
        try
        {
            Response response = await agent.SendAsync(message, ct: currentCts.Token);
            Debug.Log(response.Text);
        }
        catch (OperationCanceledException)
        {
            Debug.Log("Previous request cancelled");
        }
        finally
        {
            if (currentCts != null && !currentCts.IsCancellationRequested)
            {
                currentCts.Dispose();
                currentCts = null;
            }
        }
    }
    
    void OnDestroy()
    {
        currentCts?.Cancel();
        currentCts?.Dispose();
    }
}
```

### Queue System

```csharp
public class RequestQueue : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Queue<string> requestQueue = new();
    private bool isProcessing = false;
    private CancellationTokenSource cts;
    
    public void EnqueueMessage(string message)
    {
        requestQueue.Enqueue(message);
        
        if (!isProcessing)
        {
            ProcessQueue().Forget();
        }
    }
    
    async UniTaskVoid ProcessQueue()
    {
        isProcessing = true;
        
        while (requestQueue.Count > 0)
        {
            string message = requestQueue.Dequeue();
            
            cts = new CancellationTokenSource();
            
            try
            {
                Response response = await agent.SendAsync(message, ct: cts.Token);
                Debug.Log($"Q: {message}");
                Debug.Log($"A: {response.Text}\n");
            }
            catch (OperationCanceledException)
            {
                Debug.Log("Request cancelled");
                break; // Stop processing queue
            }
            finally
            {
                cts?.Dispose();
                cts = null;
            }
        }
        
        isProcessing = false;
    }
    
    public void CancelQueue()
    {
        cts?.Cancel();
        requestQueue.Clear();
    }
}
```

## Cleanup

### Proper Disposal

```csharp
public class ProperCleanup : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private CancellationTokenSource cts;
    
    public async void SendMessage(string message)
    {
        cts?.Dispose(); // Dispose old token
        cts = new CancellationTokenSource();
        
        try
        {
            await agent.SendAsync(message, ct: cts.Token);
        }
        catch (OperationCanceledException)
        {
            Debug.Log("Cancelled");
        }
        finally
        {
            cts?.Dispose();
            cts = null;
        }
    }
    
    void OnDestroy()
    {
        // Clean up on destroy
        cts?.Cancel();
        cts?.Dispose();
    }
    
    void OnApplicationQuit()
    {
        // Clean up on quit
        cts?.Cancel();
        cts?.Dispose();
    }
}
```

## Scene Transitions

### Cancel on Scene Change

```csharp
public class SceneTransitionHandler : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private CancellationTokenSource cts;
    
    void Start()
    {
        // Listen for scene change
        UnityEngine.SceneManagement.SceneManager.sceneLoaded += OnSceneLoaded;
    }
    
    void OnDestroy()
    {
        UnityEngine.SceneManagement.SceneManager.sceneLoaded -= OnSceneLoaded;
    }
    
    void OnSceneLoaded(
        UnityEngine.SceneManagement.Scene scene,
        UnityEngine.SceneManagement.LoadSceneMode mode)
    {
        // Cancel any in-progress requests
        CancelCurrentRequest();
    }
    
    public async void SendMessage(string message)
    {
        cts = new CancellationTokenSource();
        
        try
        {
            await agent.SendAsync(message, ct: cts.Token);
        }
        catch (OperationCanceledException)
        {
            Debug.Log("Cancelled due to scene change");
        }
        finally
        {
            cts?.Dispose();
            cts = null;
        }
    }
    
    void CancelCurrentRequest()
    {
        cts?.Cancel();
        cts?.Dispose();
        cts = null;
    }
}
```

## Error Handling

### Distinguish Cancellation from Errors

```csharp
public async void SendWithErrorHandling(string message)
{
    CancellationTokenSource cts = new CancellationTokenSource();
    
    try
    {
        Response response = await agent.SendAsync(message, ct: cts.Token);
        Debug.Log(response.Text);
    }
    catch (OperationCanceledException)
    {
        // User cancelled - not an error
        Debug.Log("Request cancelled by user");
        ShowMessage("Cancelled");
    }
    catch (ApiException ex)
    {
        // API error
        Debug.LogError($"API error: {ex.Message}");
        ShowError($"Error: {ex.Message}");
    }
    catch (Exception ex)
    {
        // Other errors
        Debug.LogError($"Unexpected error: {ex.Message}");
        ShowError("An unexpected error occurred");
    }
    finally
    {
        cts?.Dispose();
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using UnityEngine.UI;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;
using TMPro;
using System;
using System.Threading;

public class CancellationExample : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_InputField inputField;
    [SerializeField] private Button sendButton;
    [SerializeField] private Button cancelButton;
    [SerializeField] private TMP_Text chatText;
    [SerializeField] private TMP_Text statusText;
    
    [Header("Settings")]
    [SerializeField] private float timeoutSeconds = 30f;
    
    private CancellationTokenSource cts;
    private DateTime requestStartTime;
    
    void Start()
    {
        agent.Stream = true;
        
        sendButton.onClick.AddListener(OnSendClicked);
        cancelButton.onClick.AddListener(OnCancelClicked);
        
        cancelButton.gameObject.SetActive(false);
    }
    
    void Update()
    {
        // Update status during request
        if (cts != null && !cts.IsCancellationRequested)
        {
            var elapsed = DateTime.Now - requestStartTime;
            statusText.text = $"Responding... ({elapsed.TotalSeconds:F1}s)";
        }
    }
    
    async void OnSendClicked()
    {
        string message = inputField.text.Trim();
        
        if (string.IsNullOrWhiteSpace(message))
        {
            return;
        }
        
        // Clear input
        inputField.text = "";
        
        // Setup cancellation
        cts?.Dispose();
        cts = new CancellationTokenSource();
        cts.CancelAfter(TimeSpan.FromSeconds(timeoutSeconds));
        
        // Update UI
        sendButton.interactable = false;
        cancelButton.gameObject.SetActive(true);
        requestStartTime = DateTime.Now;
        statusText.text = "Sending...";
        
        // Show user message
        AppendMessage("You", message);
        
        try
        {
            Response response = await agent.SendAsync(message, ct: cts.Token);
            
            AppendMessage("Assistant", response.Text);
            statusText.text = "Ready";
        }
        catch (OperationCanceledException)
        {
            AppendMessage("System", "[Cancelled]");
            statusText.text = "Cancelled";
        }
        catch (Exception ex)
        {
            AppendMessage("System", $"Error: {ex.Message}");
            statusText.text = "Error";
            Debug.LogError(ex);
        }
        finally
        {
            // Cleanup
            sendButton.interactable = true;
            cancelButton.gameObject.SetActive(false);
            
            cts?.Dispose();
            cts = null;
        }
    }
    
    void OnCancelClicked()
    {
        if (cts != null && !cts.IsCancellationRequested)
        {
            statusText.text = "Cancelling...";
            cts.Cancel();
        }
    }
    
    void AppendMessage(string sender, string message)
    {
        chatText.text += $"\n<b>{sender}:</b> {message}\n";
    }
    
    void OnDestroy()
    {
        cts?.Cancel();
        cts?.Dispose();
    }
}
```

## Next Steps

- [Text Messages](text-messages.md)
- [Streaming vs Non-Streaming](streaming.md)
- [Agent Status](../lifecycle/agent-status.md)
