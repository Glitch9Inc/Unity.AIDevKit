# AgentBehaviour Properties

Complete reference for AgentBehaviour component properties.

## Overview

`AgentBehaviour` is a Unity MonoBehaviour component that wraps the core `Agent` class, providing inspector-friendly configuration and Unity lifecycle integration.

## Agent Reference

### Agent

```csharp
public Agent Agent { get; }
```

**Read-only.** Underlying runtime Agent instance.

**Example:**

```csharp
var agent = agentBehaviour.Agent;
await agent.GenerateResponseAsync("Hello");
```

## Identity Properties

### Id

```csharp
public string Id { get; }
```

**Read-only.** Unique agent identifier from settings.

**Example:**

```csharp
Debug.Log($"Agent ID: {agentBehaviour.Id}");
```

### Name

```csharp
public string Name { get; }
```

**Read-only.** Human-readable agent name.

**Example:**

```csharp
chatHeaderText.text = agentBehaviour.Name;
```

### Api

```csharp
public override Api Api { get; }
```

**Read-only.** API provider inferred from Model.

**Values:**

- `Api.OpenAI`
- `Api.Anthropic`
- `Api.Google`

## State Properties

### State

```csharp
public AgentStatus State { get; }
```

**Read-only.** Current agent lifecycle state.

**Values:**

- `None` - Not initialized
- `Initializing` - Initialization in progress
- `InitializationFailed` - Failed to initialize
- `Ready` - Ready for requests
- `Processing` - Handling request
- `Terminating` - Shutting down

**Example:**

```csharp
if (agentBehaviour.State == AgentStatus.Ready)
{
    sendButton.interactable = true;
}
```

### IsInitialized

```csharp
public bool IsInitialized { get; }
```

**Read-only.** True when agent is fully initialized.

**Example:**

```csharp
void Update()
{
    statusText.text = agentBehaviour.IsInitialized 
        ? "Ready" 
        : "Initializing...";
}
```

## Model Properties

### Model

```csharp
public Model Model { get; set; }
```

Active language model for text generation.

**Example:**

```csharp
agentBehaviour.Model = Model.GPT4;
agentBehaviour.Model = Model.Claude35Sonnet;
```

**Inspector:**

```csharp
[SerializeField] private AgentBehaviour agent;

void Start()
{
    // Model can be changed at runtime
    agent.Model = Model.GPT4Turbo;
}
```

### ToolChoice

```csharp
public ToolChoice ToolChoice { get; set; }
```

Tool selection strategy.

**Values:**

- `ToolChoice.Auto` - Model decides
- `ToolChoice.None` - Disable tools
- `ToolChoice.Required` - Must use tool
- `ToolChoice.Function("name")` - Specific tool

**Example:**

```csharp
// Let model decide
agentBehaviour.ToolChoice = ToolChoice.Auto;

// Force specific tool
agentBehaviour.ToolChoice = ToolChoice.Function("get_weather");
```

## Voice & Audio Properties

### Voice

```csharp
public Voice Voice { get; set; }
```

TTS voice for audio responses.

**Example:**

```csharp
agentBehaviour.Voice = Voice.Alloy;
agentBehaviour.Voice = Voice.Nova;
```

### OutputAudioVolume

```csharp
public float OutputAudioVolume { get; set; }
```

Master volume for speech output (0.0 to 1.0).

**Example:**

```csharp
volumeSlider.onValueChanged.AddListener(volume =>
{
    agentBehaviour.OutputAudioVolume = volume;
});
```

### InputAudioLanguage

```csharp
public SystemLanguage InputAudioLanguage { get; set; }
```

Language for audio input and transcription.

**Example:**

```csharp
agentBehaviour.InputAudioLanguage = SystemLanguage.English;
agentBehaviour.InputAudioLanguage = SystemLanguage.Japanese;
```

### Microphone

```csharp
public string Microphone { get; set; }
```

Name of microphone device for audio input.

**Example:**

```csharp
// List available microphones
string[] devices = UnityEngine.Microphone.devices;
microphoneDropdown.options = devices
    .Select(d => new TMP_Dropdown.OptionData(d))
    .ToList();

// Set selected microphone
microphoneDropdown.onValueChanged.AddListener(index =>
{
    agentBehaviour.Microphone = devices[index];
});
```

### IsRecording

```csharp
public bool IsRecording { get; }
```

**Read-only.** True if currently recording audio.

**Example:**

```csharp
void Update()
{
    recordButton.GetComponent<Image>().color = agentBehaviour.IsRecording 
        ? Color.red 
        : Color.white;
}
```

## Conversation Properties

### Conversation

```csharp
public Conversation Conversation { get; }
```

**Read-only.** Active conversation instance.

**Example:**

```csharp
var conversation = agentBehaviour.Conversation;
Debug.Log($"Messages: {conversation.Messages.Count}");
Debug.Log($"ID: {conversation.Id}");
```

### Items

```csharp
public List<ConversationItem> Items { get; }
```

**Read-only.** All items in conversation.

**Example:**

```csharp
foreach (var item in agentBehaviour.Items)
{
    Debug.Log($"{item.Type}: {item}");
}
```

### Messages

```csharp
public List<Message> Messages { get; }
```

**Read-only.** User and assistant messages only.

**Example:**

```csharp
foreach (var message in agentBehaviour.Messages)
{
    AddMessageToUI(message);
}
```

### LastMessage

```csharp
public Message LastMessage { get; }
```

**Read-only.** Most recent message.

**Example:**

```csharp
if (agentBehaviour.LastMessage != null)
{
    lastMessageText.text = agentBehaviour.LastMessage.Content;
}
```

### Instructions

```csharp
public string Instructions { get; }
```

**Read-only.** System instructions for agent.

**Example:**

```csharp
instructionsText.text = agentBehaviour.Instructions;
```

## Component Properties

### InputAudioRecorder

```csharp
public InputAudioRecorder InputAudioRecorder { get; }
```

**Read-only.** Recorder component for audio input.

**Example:**

```csharp
if (agentBehaviour.InputAudioRecorder != null)
{
    var level = agentBehaviour.InputAudioRecorder.GetAudioLevel();
    levelMeter.value = level;
}
```

### OutputAudioPlayer

```csharp
public OutputAudioPlayer OutputAudioPlayer { get; }
```

**Read-only.** Player component for audio output.

**Example:**

```csharp
if (agentBehaviour.OutputAudioPlayer != null)
{
    var isPlaying = agentBehaviour.OutputAudioPlayer.IsPlaying;
    speakerIcon.enabled = isPlaying;
}
```

## Capability Properties

### HasInputAudio

```csharp
public bool HasInputAudio { get; }
```

**Read-only.** True if agent accepts audio input.

**Example:**

```csharp
if (agentBehaviour.HasInputAudio)
{
    microphoneButton.gameObject.SetActive(true);
}
```

### HasOutputAudio

```csharp
public bool HasOutputAudio { get; }
```

**Read-only.** True if agent can output speech.

**Example:**

```csharp
if (agentBehaviour.HasOutputAudio)
{
    speakerButton.gameObject.SetActive(true);
}
```

### HasOutputImage

```csharp
public bool HasOutputImage { get; }
```

**Read-only.** True if agent can generate images.

**Example:**

```csharp
if (agentBehaviour.HasOutputImage)
{
    imagePanel.gameObject.SetActive(true);
}
```

## Configuration Properties

### ConversationStoreType

```csharp
public ConversationStoreType ConversationStoreType { get; }
```

**Read-only.** Storage backend for conversations.

**Serialized Field:**

```csharp
[SerializeField] private ConversationStoreType conversationStore = ConversationStoreType.LocalFile;
```

**Values:**

- `LocalFile` - Local file system
- `CloudStorage` - Cloud storage
- `Database` - Database

### InitialConversationId

```csharp
public string InitialConversationId { get; }
```

**Read-only.** Conversation ID to load on initialization.

**Serialized Field:**

```csharp
[SerializeField] private string conversationId;
```

### IncludeObfuscation

```csharp
public bool IncludeObfuscation { get; }
```

**Read-only.** Include obfuscation metadata in responses.

**Serialized Field:**

```csharp
[SerializeField] private bool includeObfuscation = false;
```

### Stream

```csharp
public bool Stream { get; set; }
```

Enable/disable streaming responses.

**Serialized Field:**

```csharp
[SerializeField] private bool stream = false;
```

**Example:**

```csharp
streamingToggle.onValueChanged.AddListener(enabled =>
{
    agentBehaviour.Stream = enabled;
});
```

## Behavior Properties

### WaitForToolCallsCompletion

```csharp
public bool WaitForToolCallsCompletion { get; }
```

**Read-only.** True if agent waits for all tool calls to complete.

### UnhandledToolCallBehaviour

```csharp
public UnhandledToolCallBehaviour UnhandledToolCallBehaviour { get; }
```

**Read-only.** Policy for unhandled tool calls.

**Serialized Field:**

```csharp
[SerializeField] private UnhandledToolCallBehaviour unhandledToolCallBehaviour;
```

**Values:**

- `Ignore` - Skip unhandled calls
- `ThrowError` - Throw exception
- `SubmitToolOutput` - Wait for external submission
- `Event` - Fire onUnhandledToolCall event

### SubmitToolOutputTimeoutSeconds

```csharp
public int SubmitToolOutputTimeoutSeconds { get; }
```

**Read-only.** Timeout for tool output submission.

**Serialized Field:**

```csharp
[Range(0, 300)]
[SerializeField] private int submitToolOutputTimeoutSeconds = 60;
```

### McpApprovalTimeoutSeconds

```csharp
public int McpApprovalTimeoutSeconds { get; }
```

**Read-only.** Timeout for MCP approval requests.

**Serialized Field:**

```csharp
[SerializeField] private int mcpApprovalTimeoutSeconds = 30;
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using TMPro;

public class AgentPropertiesUI : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    [Header("UI Elements")]
    [SerializeField] private TMP_Text nameText;
    [SerializeField] private TMP_Text statusText;
    [SerializeField] private TMP_Text modelText;
    [SerializeField] private TMP_Text messagesText;
    [SerializeField] private GameObject audioControls;
    [SerializeField] private GameObject imagePanel;
    
    void Start()
    {
        UpdateUI();
    }
    
    void Update()
    {
        UpdateStatus();
    }
    
    void UpdateUI()
    {
        // Identity
        nameText.text = agent.Name;
        
        // Model
        modelText.text = agent.Model.ToString();
        
        // Capabilities
        audioControls.SetActive(agent.HasInputAudio || agent.HasOutputAudio);
        imagePanel.SetActive(agent.HasOutputImage);
    }
    
    void UpdateStatus()
    {
        // State
        statusText.text = agent.State switch
        {
            AgentStatus.None => "Not Initialized",
            AgentStatus.Initializing => "Initializing...",
            AgentStatus.Ready => "Ready",
            AgentStatus.Processing => "Processing...",
            AgentStatus.Terminating => "Shutting Down",
            _ => "Unknown"
        };
        
        // Conversation
        if (agent.Messages != null)
        {
            messagesText.text = $"Messages: {agent.Messages.Count}";
        }
    }
    
    public void OnModelChanged(int index)
    {
        Model[] models = { Model.GPT4, Model.GPT4Turbo, Model.Claude35Sonnet };
        agent.Model = models[index];
        
        UpdateUI();
    }
    
    public void OnVolumeChanged(float volume)
    {
        agent.OutputAudioVolume = volume;
    }
}
```

## Next Steps

- [AgentBehaviour Methods](agentbehaviour-methods.md)
- [Agent Properties](agent-properties.md)
- [Agent Methods](agent-methods.md)
