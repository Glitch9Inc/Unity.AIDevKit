# What is AgentBehaviour?

`AgentBehaviour` is a Unity `MonoBehaviour` that wraps the core `Agent` class, making it easy to integrate AI functionality into your Unity scenes.

## Purpose

The `AgentBehaviour` component serves as a **Unity-facing wrapper** that:

1. **Exposes Agent functionality** in the Unity Inspector
2. **Manages Unity lifecycle** (initialization, updates, cleanup)
3. **Provides simplified API** for UI and gameplay code
4. **Handles serialization** of settings and preferences

## Key Features

### Inspector-Friendly Configuration

All agent settings can be configured directly in the Unity Inspector:

- Agent Settings reference
- Conversation store type
- Audio recorder/player references
- Streaming options
- Tool call behavior
- MCP approval settings

### Automatic Lifecycle Management

`AgentBehaviour` handles the Unity component lifecycle:

```csharp
// Initialization happens automatically
void Start() {
    // Agent is created and initialized
}

// Cleanup happens automatically
void OnDestroy() {
    // Agent is properly disposed
}
```

### Simplified Public API

Instead of working with internal controllers and services, you get clean methods:

```csharp
// Simple, unified API
await agentBehaviour.SendAsync("Hello!");
await agentBehaviour.SendAsync(audioClip);
await agentBehaviour.CancelAsync();

// Easy access to state
Conversation conversation = agentBehaviour.Conversation;
List<Message> messages = agentBehaviour.Messages;
AgentStatus status = agentBehaviour.Status;
```

## Component Structure

```
GameObject
  └─ AgentBehaviour (MonoBehaviour)
       └─ Agent (C# class)
            ├─ ConversationController
            ├─ AudioController
            ├─ ImageController
            ├─ ToolCallController
            ├─ McpController
            └─ ParametersController
```

## Configuration Fields

### Required Settings

- **Agent Settings** - Reference to an `AgentSettings` ScriptableObject that defines the agent's configuration

### Conversation Settings

- **Conversation Store** - Where to persist conversations (Memory, LocalFile, Remote)
- **Conversation Id** - Optional initial conversation to load
- **Preferences Store Type** - How to store user preferences (PlayerPrefs, LocalFile)

### Audio Settings

- **Input Audio Recorder** - Component for recording user speech
- **Output Audio Player** - Component for playing agent speech
- **Stream** - Enable streaming responses

### Tool Behavior

- **Unhandled Tool Call Behaviour** - How to handle tools not registered locally
  - `Ignore` - Skip unhandled tools
  - `RaiseEvent` - Fire event for external handling
  - `SubmitToolOutput` - Wait for manual tool output submission
  
- **Submit Tool Output Timeout** - Seconds to wait before timing out (when using SubmitToolOutput)
- **MCP Approval Timeout** - Seconds to wait for MCP tool approval

### Advanced Options

- **Include Obfuscation** - Obfuscate sensitive data in logs
- **Auto Init** - Initialize agent automatically on Start

## Example Setup

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class ChatUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private UnityEngine.UI.InputField inputField;
    [SerializeField] private UnityEngine.UI.Text chatDisplay;
    
    void Start()
    {
        // Agent initializes automatically
        agent.onTextDelta += OnTextReceived;
        agent.onResponseCompleted += OnResponseCompleted;
    }
    
    public async void OnSendButtonClicked()
    {
        string userMessage = inputField.text;
        inputField.text = "";
        
        chatDisplay.text += $"\nYou: {userMessage}";
        
        await agent.SendAsync(userMessage);
    }
    
    private void OnTextReceived(string delta)
    {
        chatDisplay.text += delta;
    }
    
    private void OnResponseCompleted(Response response)
    {
        chatDisplay.text += "\n";
    }
}
```

## When to Use AgentBehaviour

**Use AgentBehaviour when:**

- Building Unity UI for chat interfaces
- Integrating agents into GameObjects
- You need Inspector configuration
- Working with Unity-specific components (AudioSource, UI, etc.)

**Use Agent directly when:**

- Building backend services
- Writing unit tests
- Non-Unity C# projects
- You need full control over initialization

## Next Steps

- [Agent vs AgentBehaviour](agent-vs-agentbehaviour.md) - Detailed comparison
- [Quick Start](quick-start.md) - Build your first agent scene
- [Agent Configuration](../configuration/README.md) - Configure agent settings
