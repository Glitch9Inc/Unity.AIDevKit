# Available Events

Complete reference of all available agent events.

## Agent Status Events

### AgentStatusChanged

Fired when the agent's operational status changes.

```csharp
public readonly struct AgentStatusChanged : IEvent
{
    public AgentStatus NewStatus { get; }
    public string Args { get; }
}
```

**Properties:**
- `NewStatus` - The new agent status
- `Args` - Optional context-specific arguments

**Status Values:**
- `None` - Initial state
- `Initializing` - Agent is initializing
- `Ready` - Agent is ready for requests
- `GeneratingResponse` - Agent is generating a response
- `Thinking` - Agent is processing reasoning
- `Transcribing` - Agent is transcribing audio input
- `ExecutingTool` - Agent is executing a tool
- `WaitingForApproval` - Agent is waiting for MCP approval
- `Error` - Agent encountered an error
- `Disposed` - Agent has been disposed

**Example:**
```csharp
agent.RegisterEvent<AgentStatusChanged>(evt => 
{
    Debug.Log($"Agent status: {evt.NewStatus}");
    
    if (!string.IsNullOrEmpty(evt.Args))
    {
        Debug.Log($"Context: {evt.Args}");
    }
    
    switch (evt.NewStatus)
    {
        case AgentStatus.Ready:
            Debug.Log("Agent is ready!");
            break;
            
        case AgentStatus.ExecutingTool:
            Debug.Log($"Executing tool: {evt.Args}");
            break;
            
        case AgentStatus.Error:
            Debug.LogError($"Agent error: {evt.Args}");
            break;
    }
});
```

---

## Conversation Events

All conversation events inherit from `ConversationEvent`.

### ConversationCreated

Fired when a new conversation is created.

```csharp
public class ConversationCreated : ConversationEvent
{
    public Conversation Conversation { get; }
}
```

**Example:**
```csharp
agent.RegisterEvent<ConversationCreated>(evt => 
{
    Debug.Log($"New conversation created: {evt.Conversation.Id}");
    Debug.Log($"Agent ID: {evt.Conversation.AgentId}");
    
    // Save conversation reference
    SaveConversation(evt.Conversation);
});
```

### ConversationLoaded

Fired when an existing conversation is loaded from storage.

```csharp
public class ConversationLoaded : ConversationEvent
{
    public Conversation Conversation { get; }
}
```

**Example:**
```csharp
agent.RegisterEvent<ConversationLoaded>(evt => 
{
    Debug.Log($"Conversation loaded: {evt.Conversation.Id}");
    Debug.Log($"Items: {evt.Conversation.Items?.Length ?? 0}");
});
```

### ConversationDeleted

Fired when a conversation is deleted.

```csharp
public class ConversationDeleted : ConversationEvent
{
    public string ConversationId { get; }
    public bool Success { get; }
}
```

**Example:**
```csharp
agent.RegisterEvent<ConversationDeleted>(evt => 
{
    if (evt.Success)
        Debug.Log($"Conversation deleted: {evt.ConversationId}");
    else
        Debug.LogWarning($"Failed to delete: {evt.ConversationId}");
});
```

### ConversationTitleUpdated

Fired when a conversation's title is updated.

```csharp
public class ConversationTitleUpdated : ConversationEvent
{
    public string ConversationId { get; }
    public string NewTitle { get; }
}
```

**Example:**
```csharp
agent.RegisterEvent<ConversationTitleUpdated>(evt => 
{
    Debug.Log($"Title updated for {evt.ConversationId}: {evt.NewTitle}");
    UpdateUI(evt.ConversationId, evt.NewTitle);
});
```

### ConversationSummaryUpdated

Fired when a conversation's summary is updated.

```csharp
public class ConversationSummaryUpdated : ConversationEvent
{
    public string ConversationId { get; }
    public string NewSummary { get; }
}
```

### ConversationItemsLoaded

Fired when conversation items (messages) are loaded.

```csharp
public class ConversationItemsLoaded : ConversationEvent
{
    public string ConversationId { get; }
    public ConversationItem[] Items { get; }
}
```

**Example:**
```csharp
agent.RegisterEvent<ConversationItemsLoaded>(evt => 
{
    Debug.Log($"Loaded {evt.Items.Length} items for {evt.ConversationId}");
    
    foreach (var item in evt.Items)
    {
        Debug.Log($"- {item.Role}: {item.GetText()}");
    }
});
```

### ConversationListLoaded

Fired when a list of conversations is retrieved.

```csharp
public class ConversationListLoaded : ConversationEvent
{
    public Conversation[] Conversations { get; }
}
```

**Example:**
```csharp
agent.RegisterEvent<ConversationListLoaded>(evt => 
{
    Debug.Log($"Loaded {evt.Conversations.Length} conversations");
    
    foreach (var conv in evt.Conversations)
    {
        Debug.Log($"- {conv.Id}: {conv.Title}");
    }
});
```

### Handle All Conversation Events

You can subscribe to the base type to handle all conversation events:

```csharp
agent.RegisterEvent<ConversationEvent>(evt => 
{
    switch (evt)
    {
        case ConversationCreated created:
            Debug.Log($"Created: {created.Conversation.Id}");
            break;
            
        case ConversationLoaded loaded:
            Debug.Log($"Loaded: {loaded.Conversation.Id}");
            break;
            
        case ConversationDeleted deleted:
            Debug.Log($"Deleted: {deleted.ConversationId}");
            break;
            
        case ConversationTitleUpdated titleUpdated:
            Debug.Log($"Title: {titleUpdated.NewTitle}");
            break;
            
        case ConversationSummaryUpdated summaryUpdated:
            Debug.Log($"Summary: {summaryUpdated.NewSummary}");
            break;
            
        case ConversationItemsLoaded itemsLoaded:
            Debug.Log($"Items: {itemsLoaded.Items.Length}");
            break;
            
        case ConversationListLoaded listLoaded:
            Debug.Log($"List: {listLoaded.Conversations.Length}");
            break;
    }
});
```

---

## Tool Events

### ToolStatusEvent

Fired when a tool's execution status changes.

```csharp
public sealed class ToolStatusEvent : IEvent
{
    public ToolType Type { get; }
    public ToolStatus? ToolStatus { get; }
    public SearchStatus? SearchStatus { get; }
    public GenerationStatus? GenerationStatus { get; }
    
    public bool IsFinal() { }
}
```

**Properties:**
- `Type` - The type of tool (Search, Generation, etc.)
- `ToolStatus` - General tool execution status
- `SearchStatus` - Search-specific status
- `GenerationStatus` - Generation-specific status

**Example:**
```csharp
agent.RegisterEvent<ToolStatusEvent>(evt => 
{
    Debug.Log($"Tool status: {evt.Type}");
    
    if (evt.ToolStatus.HasValue)
        Debug.Log($"Status: {evt.ToolStatus.Value}");
    
    if (evt.SearchStatus.HasValue)
        Debug.Log($"Search: {evt.SearchStatus.Value}");
    
    if (evt.GenerationStatus.HasValue)
        Debug.Log($"Generation: {evt.GenerationStatus.Value}");
    
    if (evt.IsFinal())
        Debug.Log("Tool execution finished");
});
```

### ToolOutputEvent

Fired when a tool produces output or completes execution.

```csharp
public readonly struct ToolOutputEvent : IEvent
{
    public ToolOutputEventType Type { get; }
    public string ToolName { get; }
    public ToolOutput ToolOutput { get; }
}
```

**Event Types:**
- `Unknown` - Unknown event type
- `Submitting` - Tool output is being submitted
- `Result` - Tool execution completed with result
- `Error` - Tool execution failed

**Example:**
```csharp
agent.RegisterEvent<ToolOutputEvent>(evt => 
{
    Debug.Log($"Tool: {evt.ToolName}");
    
    switch (evt.Type)
    {
        case ToolOutputEventType.Submitting:
            Debug.Log("Submitting tool output...");
            break;
            
        case ToolOutputEventType.Result:
            Debug.Log("Tool completed successfully");
            if (evt.ToolOutput != null)
            {
                Debug.Log($"Output: {evt.ToolOutput}");
            }
            break;
            
        case ToolOutputEventType.Error:
            Debug.LogError($"Tool failed: {evt.ToolName}");
            break;
    }
});
```

---

## Streaming Events (Deltas)

### Delta\<TextData\>

Fired during text streaming, including chat responses, reasoning, and transcription.

```csharp
public class Delta<TextData>
{
    public int Index { get; }
    public TextData Value { get; }
    public string Snapshot { get; }
    public bool Done { get; }
}

public class TextData
{
    public string Text { get; }
    public TextType Type { get; }
}
```

**Text Types:**
- `Message` - Standard message text
- `Reasoning` - Reasoning/thinking text
- `ReasoningSummary` - Summary of reasoning
- `InputTranscript` - Transcribed user input

**Example:**
```csharp
agent.RegisterEvent<Delta<TextData>>(delta => 
{
    if (delta.Value?.Text != null)
    {
        // Stream the text character by character
        AppendText(delta.Value.Text);
        
        // Check text type
        switch (delta.Value.Type)
        {
            case TextType.Reasoning:
                ShowReasoningIndicator();
                break;
                
            case TextType.InputTranscript:
                DisplayTranscription(delta.Value.Text);
                break;
        }
        
        // Final text available in snapshot when done
        if (delta.Done)
        {
            Debug.Log($"Complete text: {delta.Snapshot}");
            FinalizeText();
        }
    }
});
```

### Delta\<IImagePayload\>

Fired when image data is received (streaming or complete).

```csharp
agent.RegisterEvent<Delta<IImagePayload>>(delta => 
{
    if (delta.Value != null)
    {
        // Process image data
        var imageData = delta.Value;
        
        if (imageData is ImageUrl imageUrl)
        {
            Debug.Log($"Image URL: {imageUrl.Url}");
            LoadImage(imageUrl.Url);
        }
        else if (imageData is ImageBase64 base64)
        {
            Debug.Log("Image base64 data received");
            DecodeAndDisplayImage(base64.Data);
        }
        
        if (delta.Done)
        {
            Debug.Log("Image complete");
        }
    }
});
```

### Delta\<IAudioDelta\>

Fired when audio data is received (streaming or complete).

```csharp
agent.RegisterEvent<Delta<IAudioDelta>>(delta => 
{
    if (delta.Value != null)
    {
        // Process audio chunk
        var audioData = delta.Value.GetAudioData();
        PlayAudioChunk(audioData);
        
        if (delta.Done)
        {
            Debug.Log("Audio stream complete");
            FinalizeAudio();
        }
    }
});
```

### Delta\<Annotation\>

Fired when content annotations are received (citations, references, etc.).

```csharp
agent.RegisterEvent<Delta<Annotation>>(delta => 
{
    if (delta.Value != null)
    {
        var annotation = delta.Value;
        Debug.Log($"Annotation: {annotation.Type} at [{annotation.StartIndex}-{annotation.EndIndex}]");
        
        if (annotation.Reference != null)
        {
            Debug.Log($"Reference: {annotation.Reference}");
        }
    }
});
```

---

## Audio Events

### AudioBufferStateChanged

Fired when the audio buffer state changes (for Realtime API).

```csharp
public sealed class AudioBufferStateChanged : IEvent
{
    public AudioBufferStateEvent EventType { get; }
    public string PreviousItemId { get; }
    public long AudioStartMs { get; }
    public long AudioEndMs { get; }
}
```

**Event Types:**
- `Committed` - Audio buffer committed
- `Cleared` - Audio buffer cleared
- `SpeechStarted` - Speech detection started
- `SpeechStopped` - Speech detection stopped
- `TimeoutTriggered` - VAD timeout triggered

**Example:**
```csharp
agent.RegisterEvent<AudioBufferStateChanged>(evt => 
{
    Debug.Log($"Audio buffer: {evt.EventType}");
    
    switch (evt.EventType)
    {
        case AudioBufferStateEvent.SpeechStarted:
            ShowListeningIndicator();
            break;
            
        case AudioBufferStateEvent.SpeechStopped:
            HideListeningIndicator();
            break;
            
        case AudioBufferStateEvent.Committed:
            Debug.Log($"Audio committed: {evt.AudioStartMs}ms - {evt.AudioEndMs}ms");
            break;
            
        case AudioBufferStateEvent.TimeoutTriggered:
            Debug.Log("VAD timeout triggered");
            break;
    }
});
```

### AudioRateLimitsUpdated

Fired when audio rate limits are updated (for Realtime API).

```csharp
public sealed class AudioRateLimitsUpdated : IEvent
{
    public RealtimeRateLimit[] RateLimits { get; }
}
```

**Example:**
```csharp
agent.RegisterEvent<AudioRateLimitsUpdated>(evt => 
{
    Debug.Log("Rate limits updated:");
    
    foreach (var limit in evt.RateLimits)
    {
        Debug.Log($"- {limit.Name}: {limit.Remaining}/{limit.Limit}");
    }
});
```

---

## Metadata Events

### Usage

Fired when token usage information is available.

```csharp
public class Usage
{
    public int PromptTokens { get; }
    public int CompletionTokens { get; }
    public int TotalTokens { get; }
    public TokenDetails PromptTokensDetails { get; }
    public TokenDetails CompletionTokensDetails { get; }
}
```

**Example:**
```csharp
agent.RegisterEvent<Usage>(usage => 
{
    Debug.Log($"Token usage:");
    Debug.Log($"  Prompt: {usage.PromptTokens}");
    Debug.Log($"  Completion: {usage.CompletionTokens}");
    Debug.Log($"  Total: {usage.TotalTokens}");
    
    // Track usage for billing
    TrackUsage(usage.TotalTokens);
});
```

### Exception

Fired when an error occurs during agent operations.

```csharp
agent.RegisterEvent<Exception>(ex => 
{
    Debug.LogError($"Agent error: {ex.Message}");
    Debug.LogError($"Stack trace: {ex.StackTrace}");
    
    // Handle specific exceptions
    if (ex is TimeoutException)
    {
        Debug.LogWarning("Request timed out. Retrying...");
        RetryLastRequest();
    }
    else if (ex is UnauthorizedAccessException)
    {
        Debug.LogError("Authentication failed. Check API key.");
        ShowAuthenticationError();
    }
});
```

---

## Event Interfaces

If you prefer interface-based event handling, implement `IAgentEventListener`:

```csharp
public class MyEventListener : IAgentEventListener
{
    public void OnAgentStatusChanged(AgentStatusChanged evt)
    {
        Debug.Log($"Status: {evt.NewStatus}");
    }
    
    public void OnConversationEvent(ConversationEvent evt)
    {
        Debug.Log($"Conversation event: {evt.GetType().Name}");
    }
    
    public void OnTextDelta(Delta<TextData> e)
    {
        if (e.Value?.Text != null)
            Debug.Log(e.Value.Text);
    }
    
    public void OnImageDelta(Delta<IImagePayload> e)
    {
        Debug.Log("Image data received");
    }
    
    public void OnAnnotationDelta(Delta<Annotation> e)
    {
        Debug.Log($"Annotation: {e.Value?.Type}");
    }
    
    public void OnToolOutput(ToolOutputEvent e)
    {
        Debug.Log($"Tool output: {e.ToolName}");
    }
    
    public void OnToolStatus(ToolStatusEvent e)
    {
        Debug.Log($"Tool status: {e.Type}");
    }
    
    public void OnUsage(Usage usage)
    {
        Debug.Log($"Usage: {usage.TotalTokens} tokens");
    }
    
    public void OnError(Exception exception)
    {
        Debug.LogError($"Error: {exception.Message}");
    }
    
    public void OnAgentPreferencesSaved(UnifiedApiAgentSaveData prefs)
    {
        Debug.Log("Agent preferences saved");
    }
}

// Usage
var listener = new MyEventListener();
var service = new AgentService { EventListener = listener };
var agent = new Agent(config, settings, service);
```

---

## Event Summary Table

| Event | Category | Description |
|-------|----------|-------------|
| `AgentStatusChanged` | Agent | Agent operational status changes |
| `ConversationCreated` | Conversation | New conversation created |
| `ConversationLoaded` | Conversation | Conversation loaded from storage |
| `ConversationDeleted` | Conversation | Conversation deleted |
| `ConversationTitleUpdated` | Conversation | Conversation title changed |
| `ConversationSummaryUpdated` | Conversation | Conversation summary changed |
| `ConversationItemsLoaded` | Conversation | Conversation messages loaded |
| `ConversationListLoaded` | Conversation | List of conversations retrieved |
| `ToolStatusEvent` | Tool | Tool execution status update |
| `ToolOutputEvent` | Tool | Tool execution result |
| `Delta<TextData>` | Streaming | Text streaming updates |
| `Delta<IImagePayload>` | Streaming | Image data updates |
| `Delta<IAudioDelta>` | Streaming | Audio data updates |
| `Delta<Annotation>` | Streaming | Content annotation updates |
| `AudioBufferStateChanged` | Audio | Audio buffer state changes |
| `AudioRateLimitsUpdated` | Audio | Rate limit updates |
| `Usage` | Metadata | Token usage information |
| `Exception` | Metadata | Error events |

---

## Next Steps

- [Registering Events](registering-events.md) - Event subscription patterns
- [Overview](overview.md) - Event system architecture
