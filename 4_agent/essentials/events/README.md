---
icon: bell
---

# Events & Hooks

Subscribe to agent events and hooks for real-time updates and custom behavior.

## Overview

AI Dev Kit provides two event systems:

1. **UnityEvents** - For AgentBehaviour (Inspector-bindable)
2. **AgentHooks** - For Agent class (code-based)

## AgentBehaviour Events (Unity)

### Text Events

```csharp
[SerializeField] private AgentBehaviour agent;

void Start()
{
    agent.onTextDelta.AddListener(OnTextDelta);
    agent.onResponseStarted.AddListener(OnResponseStarted);
    agent.onResponseCompleted.AddListener(OnResponseCompleted);
}

void OnTextDelta(string delta)
{
    // Append streaming text
    chatDisplay.text += delta;
}

void OnResponseStarted(Response response)
{
    Debug.Log("Response started");
}

void OnResponseCompleted(Response response)
{
    Debug.Log($"Completed. Tokens: {response.Usage.TotalTokens}");
}
```

### Tool Events

```csharp
agent.onToolCallStarted.AddListener(OnToolCallStarted);
agent.onToolCallCompleted.AddListener(OnToolCallCompleted);

void OnToolCallStarted(ToolCall toolCall)
{
    Debug.Log($"Calling: {toolCall.Function.Name}");
}

void OnToolCallCompleted(ToolCall toolCall, string result)
{
    Debug.Log($"Result: {result}");
}
```

### Status Events

```csharp
agent.onStatusChanged.AddListener(OnStatusChanged);

void OnStatusChanged(AgentStatus status)
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
            ShowError();
            break;
    }
}
```

### Audio Events

```csharp
agent.onInputAudioStarted.AddListener(OnInputAudioStarted);
agent.onInputAudioCompleted.AddListener(OnInputAudioCompleted);
agent.onOutputAudioStarted.AddListener(OnOutputAudioStarted);
agent.onOutputAudioCompleted.AddListener(OnOutputAudioCompleted);

void OnInputAudioStarted()
{
    ShowRecordingIndicator();
}

void OnInputAudioCompleted(AudioClip clip)
{
    Debug.Log($"Recorded {clip.length} seconds");
}
```

### Error Events

```csharp
agent.onError.AddListener(OnError);

void OnError(string error)
{
    Debug.LogError($"Agent error: {error}");
    ShowErrorDialog(error);
}
```

## AgentHooks (Code-Based)

### Creating Hooks

```csharp
var hooks = new AgentHooks
{
    // Text callbacks
    TextDelta = OnTextDelta,
    ResponseStarted = OnResponseStarted,
    ResponseCompleted = OnResponseCompleted,
    
    // Tool callbacks
    ToolCall = new ToolCallHooks
    {
        Started = OnToolCallStarted,
        Completed = OnToolCallCompleted
    },
    
    // Status callback
    StatusChanged = OnStatusChanged,
    
    // Error callback
    Error = OnError,
    
    // Audio callbacks
    InputAudioRecorder = customRecorder,
    OutputAudioPlayer = customPlayer,
    
    // Conversation callback
    Conversation = new ConversationHooks
    {
        ItemAdded = OnItemAdded,
        ItemUpdated = OnItemUpdated
    }
};

// Create agent with hooks
var agent = new Agent(settings, behaviour, hooks);
```

### Hook Methods

```csharp
void OnTextDelta(string delta)
{
    Debug.Log($"Delta: {delta}");
}

void OnResponseStarted(Response response)
{
    Debug.Log("Started");
}

void OnResponseCompleted(Response response)
{
    Debug.Log($"Completed: {response.Text}");
}

void OnToolCallStarted(ToolCall toolCall)
{
    Debug.Log($"Tool: {toolCall.Function.Name}");
}

void OnToolCallCompleted(ToolCall toolCall, string result)
{
    Debug.Log($"Result: {result}");
}

void OnStatusChanged(AgentStatus status)
{
    Debug.Log($"Status: {status}");
}

void OnError(Exception ex)
{
    Debug.LogError($"Error: {ex.Message}");
}
```

## Complete Example

```csharp
using UnityEngine;
using UnityEngine.UI;
using Glitch9.AIDevKit.Agents;

public class ChatUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Text chatDisplay;
    [SerializeField] private Text statusText;
    [SerializeField] private GameObject loadingIndicator;
    
    void Start()
    {
        // Subscribe to all relevant events
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onResponseStarted.AddListener(OnResponseStarted);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
        agent.onStatusChanged.AddListener(OnStatusChanged);
        agent.onToolCallStarted.AddListener(OnToolCallStarted);
        agent.onToolCallCompleted.AddListener(OnToolCallCompleted);
        agent.onError.AddListener(OnError);
    }
    
    void OnTextDelta(string delta)
    {
        chatDisplay.text += delta;
    }
    
    void OnResponseStarted(Response response)
    {
        chatDisplay.text += "\n<color=#2ECC71><b>Agent:</b></color> ";
    }
    
    void OnResponseCompleted(Response response)
    {
        chatDisplay.text += "\n";
        statusText.text = $"Tokens: {response.Usage.TotalTokens}";
    }
    
    void OnStatusChanged(AgentStatus status)
    {
        loadingIndicator.SetActive(status == AgentStatus.Processing);
        statusText.text = status.ToString();
    }
    
    void OnToolCallStarted(ToolCall toolCall)
    {
        statusText.text = $"Calling: {toolCall.Function.Name}";
    }
    
    void OnToolCallCompleted(ToolCall toolCall, string result)
    {
        statusText.text = $"Tool completed: {toolCall.Function.Name}";
    }
    
    void OnError(string error)
    {
        chatDisplay.text += $"\n<color=red>Error: {error}</color>\n";
    }
}
```

## Event Binding in Inspector

You can also bind events directly in the Unity Inspector:

1. Select GameObject with AgentBehaviour
2. Find event (e.g., `onTextDelta`)
3. Click `+` to add listener
4. Drag GameObject/Component
5. Select method to call

## Next Steps

- [Agent Hooks](agent-hooks.md) - All available hooks
- [Text Delta Events](text-delta.md) - Streaming text events
- [Tool Call Events](tool-calls.md) - Tool execution events
- [Audio Events](audio.md) - Audio I/O events
- [Status Events](status.md) - Lifecycle status events
- [Error Handling](error-handling.md) - Error management
