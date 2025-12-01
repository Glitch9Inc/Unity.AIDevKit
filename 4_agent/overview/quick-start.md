# Quick Start

Get up and running with your first AI Agent in minutes.

## Prerequisites

Before creating an agent, make sure you have:

1. **API Key configured** - See [API Key Setup](../../quick-start/api-key-setup/README.md)
2. **Model added** - See [Adding Models & Voices](../../quick-start/adding-models-and-voices/README.md)
3. **AI Dev Kit imported** - Version 4.0 or later

## Step 1: Create Agent Settings

Agent Settings are ScriptableObjects that define your agent's configuration.

1. Right-click in Project window
2. Select `Create > AI Dev Kit > Agent Settings`
3. Name it (e.g., "MyAssistantSettings")

### Configure Basic Settings

In the Inspector:

```
Agent Settings
├─ Chat Service API: Chat Completions
├─ Model: gpt-4o
├─ Instructions: "You are a helpful assistant."
├─ Temperature: 0.7
└─ Max Tokens: 1000
```

## Step 2: Add AgentBehaviour to Scene

### Option A: Using Component Menu

1. Create new GameObject (or select existing)
2. Add Component > AI Dev Kit > Agent Behaviour
3. Assign Agent Settings to the component

### Option B: Using Prefab

```csharp
// Create from code
GameObject agentObj = new GameObject("AI Agent");
AgentBehaviour agent = agentObj.AddComponent<AgentBehaviour>();
agent.Settings = myAgentSettings;
```

## Step 3: Send Your First Message

### Using AgentBehaviour in Unity

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class SimpleChat : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    async void Start()
    {
        // Wait for agent to initialize
        await agent.InitializeAsync();
        
        // Send a message
        Response response = await agent.SendAsync("Hello! Who are you?");
        
        Debug.Log($"Agent: {response.Text}");
    }
}
```

### Using Agent Class Directly

```csharp
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;
using UnityEngine;

public class DirectAgentExample : MonoBehaviour
{
    [SerializeField] private AgentSettings settings;
    
    private Agent agent;
    
    async void Start()
    {
        // Create agent
        agent = new Agent(
            settings: settings,
            behaviour: CreateBehaviour(),
            hooks: CreateHooks()
        );
        
        // Initialize
        await agent.InitializeAsync();
        
        // Send message
        Response response = await agent.SendAsync("Tell me a joke!");
        Debug.Log(response.Text);
    }
    
    private IAgentBehaviour CreateBehaviour()
    {
        return new AgentBehaviourConfig
        {
            AutoInit = true,
            ConversationStoreType = ConversationStoreType.Memory,
            Stream = false,
            LogLevel = System.Diagnostics.TraceLevel.Info
        };
    }
    
    private AgentHooks CreateHooks()
    {
        return new AgentHooks
        {
            TextDelta = (delta) => Debug.Log($"Delta: {delta}"),
            StatusChanged = (status) => Debug.Log($"Status: {status}")
        };
    }
    
    private void OnDestroy()
    {
        agent?.Dispose();
    }
}
```

## Step 4: Handle Streaming Responses

Enable real-time text streaming for better UX:

```csharp
using UnityEngine;
using UnityEngine.UI;
using Glitch9.AIDevKit.Agents;

public class StreamingChat : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Text chatDisplay;
    [SerializeField] private InputField inputField;
    
    void Start()
    {
        // Enable streaming in AgentBehaviour inspector
        agent.Stream = true;
        
        // Subscribe to text deltas
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
    }
    
    public async void OnSendButtonClicked()
    {
        string userMessage = inputField.text;
        inputField.text = "";
        
        // Display user message
        chatDisplay.text += $"\n<b>You:</b> {userMessage}\n<b>Agent:</b> ";
        
        // Send to agent (response will stream via onTextDelta)
        await agent.SendAsync(userMessage);
    }
    
    private void OnTextDelta(string delta)
    {
        // Append each piece of text as it arrives
        chatDisplay.text += delta;
    }
    
    private void OnResponseCompleted(Response response)
    {
        chatDisplay.text += "\n";
    }
}
```

## Step 5: Add Voice Input/Output (Optional)

### Enable Audio

1. In Agent Settings:
   - Check "Enable Input Audio"
   - Check "Enable Output Audio"
   - Set Speech Model (e.g., "tts-1")
   - Set Voice (e.g., "alloy")

2. Add Audio Components:

```csharp
[SerializeField] private AgentBehaviour agent;
[SerializeField] private InputAudioRecorder audioRecorder;
[SerializeField] private OutputAudioPlayer audioPlayer;

void Start()
{
    agent.InputAudioRecorder = audioRecorder;
    agent.OutputAudioPlayer = audioPlayer;
}

public async void OnMicButtonPressed()
{
    // Send recorded audio
    AudioClip recordedClip = await audioRecorder.RecordAsync();
    await agent.SendAsync(recordedClip);
}
```

## Complete Example: Simple Chat UI

```csharp
using UnityEngine;
using UnityEngine.UI;
using Glitch9.AIDevKit.Agents;
using System.Text;

public class ChatUI : MonoBehaviour
{
    [Header("Components")]
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private InputField inputField;
    [SerializeField] private Text chatDisplay;
    [SerializeField] private Button sendButton;
    [SerializeField] private Button clearButton;
    
    private StringBuilder currentResponse = new StringBuilder();
    
    void Start()
    {
        // Setup buttons
        sendButton.onClick.AddListener(OnSendClicked);
        clearButton.onClick.AddListener(OnClearClicked);
        
        // Setup agent events
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onResponseStarted.AddListener(OnResponseStarted);
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
        agent.onError.AddListener(OnError);
        
        // Enable interactivity when ready
        agent.onStatusChanged.AddListener(OnStatusChanged);
    }
    
    private async void OnSendClicked()
    {
        string message = inputField.text.Trim();
        if (string.IsNullOrEmpty(message)) return;
        
        inputField.text = "";
        sendButton.interactable = false;
        
        // Display user message
        AppendToChat($"<color=#4A90E2><b>You:</b></color> {message}\n");
        
        // Send to agent
        await agent.SendAsync(message);
        
        sendButton.interactable = true;
    }
    
    private void OnClearClicked()
    {
        chatDisplay.text = "";
        agent.ClearConversation();
    }
    
    private void OnResponseStarted(Response response)
    {
        currentResponse.Clear();
        AppendToChat("<color=#2ECC71><b>Agent:</b></color> ");
    }
    
    private void OnTextDelta(string delta)
    {
        currentResponse.Append(delta);
        chatDisplay.text += delta;
    }
    
    private void OnResponseCompleted(Response response)
    {
        AppendToChat("\n\n");
    }
    
    private void OnError(string error)
    {
        AppendToChat($"<color=red>Error: {error}</color>\n\n");
    }
    
    private void OnStatusChanged(AgentStatus status)
    {
        sendButton.interactable = status == AgentStatus.Ready;
    }
    
    private void AppendToChat(string text)
    {
        chatDisplay.text += text;
    }
}
```

## Next Steps

Now that you have a working agent:

- **[Agent Configuration](../configuration/README.md)** - Customize agent behavior
- **[Conversation Management](../conversation/README.md)** - Save and load conversations
- **[Tools](../tools/README.md)** - Add function calling capabilities
- **[Audio](../audio/README.md)** - Implement voice interactions
- **[Events & Hooks](../events/README.md)** - Advanced event handling

## Troubleshooting

### Agent not initializing

- Check API key is configured
- Verify model name is correct
- Check console for error messages

### No response received

- Ensure `await` is used on `SendAsync()`
- Check internet connection
- Verify model has sufficient quota

### Streaming not working

- Enable `Stream = true` in AgentBehaviour
- Subscribe to `onTextDelta` event before sending

### Audio not working

- Verify audio components are assigned
- Check "Enable Input/Output Audio" in settings
- Ensure microphone permissions are granted
