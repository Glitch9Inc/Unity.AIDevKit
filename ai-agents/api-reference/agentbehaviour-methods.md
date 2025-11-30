# AgentBehaviour Methods

Complete reference for AgentBehaviour component methods.

## Overview

`AgentBehaviour` provides Unity-friendly methods for agent initialization, conversation management, message sending, audio recording, and file attachments.

## Initialization Methods

### InitializeAsync

```csharp
public override UniTask InitializeAsync()
```

Initializes the agent, registers tool executors, and commits parameters.

**Returns:** `UniTask` that completes when initialization finishes.

**Example:**

```csharp
[SerializeField] private AgentBehaviour agent;

async void Start()
{
    await agent.InitializeAsync();
    
    Debug.Log($"Agent ready: {agent.IsInitialized}");
}
```

## Conversation Management Methods

### CreateConversation (Sync)

```csharp
public void CreateConversation()
```

Creates a new conversation and makes it active (fire-and-forget).

**Example:**

```csharp
public void OnNewChatButtonClicked()
{
    agent.CreateConversation();
    
    // UI will update via events
    ClearMessageHistory();
}
```

### CreateConversationAsync

```csharp
public UniTask<Conversation> CreateConversationAsync(CancellationToken ct = default)
```

Asynchronously creates a new conversation.

**Parameters:**

- `ct` - Cancellation token

**Returns:** Newly created `Conversation`.

**Example:**

```csharp
public async void CreateNewChat()
{
    Conversation conversation = await agent.CreateConversationAsync();
    
    Debug.Log($"New conversation: {conversation.Id}");
    UpdateConversationUI(conversation);
}
```

### LoadConversation (Sync)

```csharp
public void LoadConversation(string conversationId, bool createIfNotFound = true)
```

Loads existing conversation by ID (fire-and-forget).

**Parameters:**

- `conversationId` - Conversation identifier
- `createIfNotFound` - Create new if not found

**Example:**

```csharp
public void OnConversationSelected(string conversationId)
{
    agent.LoadConversation(conversationId);
}
```

### LoadConversationAsync

```csharp
public UniTask<Conversation> LoadConversationAsync(
    string conversationId, 
    bool createIfNotFound = true, 
    CancellationToken ct = default)
```

Asynchronously loads conversation by ID.

**Parameters:**

- `conversationId` - Conversation identifier
- `createIfNotFound` - Create new if not found
- `ct` - Cancellation token

**Returns:** Loaded or created `Conversation`.

**Example:**

```csharp
public async void LoadChatHistory(string conversationId)
{
    Conversation conversation = await agent.LoadConversationAsync(conversationId);
    
    if (conversation != null)
    {
        DisplayMessages(conversation.Messages);
    }
}
```

### SetConversation (Sync)

```csharp
public void SetConversation(Conversation conversation)
```

Sets active conversation instance (fire-and-forget).

**Parameters:**

- `conversation` - Conversation to make active

**Example:**

```csharp
public void RestoreConversation(Conversation savedConversation)
{
    agent.SetConversation(savedConversation);
}
```

### SetConversationAsync

```csharp
public UniTask SetConversationAsync(
    Conversation conversation, 
    CancellationToken ct = default)
```

Asynchronously sets active conversation.

**Parameters:**

- `conversation` - Conversation to make active
- `ct` - Cancellation token

**Returns:** `UniTask` that completes when set.

### ListConversationsAsync

```csharp
public UniTask<Conversation[]> ListConversationsAsync(CancellationToken ct = default)
```

Lists all conversations for this agent.

**Parameters:**

- `ct` - Cancellation token

**Returns:** Array of `Conversation` objects.

**Example:**

```csharp
public async void ShowConversationList()
{
    Conversation[] conversations = await agent.ListConversationsAsync();
    
    foreach (var conv in conversations)
    {
        AddConversationToList(conv.Id, conv.Title, conv.Messages.Count);
    }
}
```

### SaveConversation (Sync)

```csharp
public void SaveConversation()
```

Saves current conversation (fire-and-forget).

**Example:**

```csharp
void OnApplicationPause(bool pause)
{
    if (pause)
    {
        agent.SaveConversation();
    }
}
```

### SaveConversationAsync

```csharp
public UniTask SaveConversationAsync(CancellationToken ct = default)
```

Asynchronously saves current conversation.

**Parameters:**

- `ct` - Cancellation token

**Returns:** `UniTask` that completes when saved.

**Example:**

```csharp
public async void OnSaveButtonClicked()
{
    await agent.SaveConversationAsync();
    
    ShowNotification("Conversation saved");
}
```

### SaveConversationItems (Sync)

```csharp
public void SaveConversationItems()
```

Saves conversation items (fire-and-forget).

### SaveConversationItemsAsync

```csharp
public UniTask SaveConversationItemsAsync(CancellationToken ct = default)
```

Asynchronously saves conversation items.

**Parameters:**

- `ct` - Cancellation token

**Returns:** `UniTask` that completes when saved.

## Message Sending Methods

### SendText

```csharp
public void SendText(string inputText)
```

Sends plain text as user message.

**Parameters:**

- `inputText` - User text to send

**Example:**

```csharp
[SerializeField] private AgentBehaviour agent;
[SerializeField] private TMP_InputField inputField;

public void OnSendButtonClicked()
{
    string text = inputField.text;
    
    if (!string.IsNullOrEmpty(text))
    {
        agent.SendText(text);
        inputField.text = "";
    }
}
```

**Note:** Cannot use name `SendMessage(string)` due to MonoBehaviour conflict.

### SendMessage

```csharp
public void SendMessage(Message inputMessage)
```

Sends structured message to agent.

**Parameters:**

- `inputMessage` - Message to send

**Example:**

```csharp
public void SendImageMessage(string text, Texture2D image)
{
    var message = new UserMessage(text);
    message.AddAttachment(image.ToFile());
    
    agent.SendMessage(message);
}
```

## Audio Recording Methods

### StartTranscription

```csharp
public void StartTranscription()
```

Starts audio recording for transcription.

**Example:**

```csharp
[SerializeField] private AgentBehaviour agent;
[SerializeField] private Button recordButton;

void Start()
{
    recordButton.onClick.AddListener(() =>
    {
        if (!agent.IsRecording)
        {
            agent.StartTranscription();
            recordButton.GetComponentInChildren<TMP_Text>().text = "Stop";
        }
        else
        {
            agent.StopTranscription();
            recordButton.GetComponentInChildren<TMP_Text>().text = "Record";
        }
    });
}
```

### StopTranscription

```csharp
public void StopTranscription()
```

Stops recording and submits for transcription.

**Example:**

```csharp
public void OnRecordButtonReleased()
{
    if (agent.IsRecording)
    {
        agent.StopTranscription();
        ShowProcessingIndicator();
    }
}
```

## Tool Registration Methods

### RegisterToolCallExecutor

```csharp
public void RegisterToolCallExecutor<TCall, TOutput>(
    IToolCallExecutor<TCall, TOutput> executor)
    where TCall : ToolCall 
    where TOutput : ToolOutput
```

Registers a tool call executor for a specific tool type.

**Type Parameters:**

- `TCall` - Tool call type (FunctionCall, LocalShellCall, ComputerUseCall, CustomToolCall)
- `TOutput` - Tool output type (FunctionOutput, LocalShellOutput, ComputerUseOutput, CustomToolOutput)

**Parameters:**

- `executor` - Executor instance that handles the tool

**Example:**

```csharp
public class WeatherExecutor : MonoBehaviour, IFunctionCallExecutor
{
    public bool CanExecute(string toolName) => toolName == "get_weather";
    
    public async UniTask<FunctionOutput> ExecuteAsync(
        FunctionCall toolCall, 
        CancellationToken ct = default)
    {
        // Implementation
        return new FunctionOutput
        {
            CallId = toolCall.CallId,
            Output = "{\"temperature\": 72}"
        };
    }
}

// Manual registration
var executor = GetComponent<WeatherExecutor>();
agent.RegisterToolCallExecutor<FunctionCall, FunctionOutput>(executor);

// Note: Executors attached as components are auto-registered during initialization
```

### UnregisterToolCallExecutor

```csharp
public bool UnregisterToolCallExecutor<TCall, TOutput>(
    IToolCallExecutor<TCall, TOutput> executor)
    where TCall : ToolCall 
    where TOutput : ToolOutput
```

Unregisters a previously registered tool call executor.

**Type Parameters:**

- `TCall` - Tool call type
- `TOutput` - Tool output type

**Parameters:**

- `executor` - Executor instance to remove

**Returns:** `true` if successfully removed, `false` otherwise.

**Example:**

```csharp
var executor = GetComponent<WeatherExecutor>();

if (agent.UnregisterToolCallExecutor<FunctionCall, FunctionOutput>(executor))
{
    Debug.Log("Executor unregistered");
}
```

## File Attachment Methods

### AddAttachments (IFile)

```csharp
public void AddAttachments(params IFile[] files)
```

Adds file attachments to pending message.

**Parameters:**

- `files` - Files to attach

**Example:**

```csharp
public void OnFileSelected(IFile file)
{
    agent.AddAttachments(file);
    
    ShowAttachment(file.Name);
}
```

### AddAttachments (Texture2D)

```csharp
public void AddAttachments(params Texture2D[] images)
```

Adds image attachments to pending message.

**Parameters:**

- `images` - Images to attach

**Example:**

```csharp
public void OnScreenshotTaken(Texture2D screenshot)
{
    agent.AddAttachments(screenshot);
    
    ShowAttachmentPreview(screenshot);
}
```

### AddAttachments (AudioClip)

```csharp
public void AddAttachments(params AudioClip[] audioClips)
```

Adds audio attachments to pending message.

**Parameters:**

- `audioClips` - Audio clips to attach

**Example:**

```csharp
public void OnAudioRecorded(AudioClip clip)
{
    agent.AddAttachments(clip);
    
    ShowAudioAttachment(clip.length);
}
```

## Complete Examples

### Basic Chat Interface

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using TMPro;
using UnityEngine.UI;

public class ChatInterface : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_InputField inputField;
    [SerializeField] private Button sendButton;
    [SerializeField] private Transform messageContainer;
    [SerializeField] private GameObject messagePrefab;
    
    async void Start()
    {
        await agent.InitializeAsync();
        
        sendButton.onClick.AddListener(SendMessage);
        inputField.onSubmit.AddListener(_ => SendMessage());
        
        // Subscribe to responses
        agent.onTextCompleted += OnResponseReceived;
    }
    
    void SendMessage()
    {
        string text = inputField.text.Trim();
        
        if (string.IsNullOrEmpty(text))
            return;
        
        // Display user message
        AddMessageToUI("You", text);
        
        // Send to agent
        agent.SendText(text);
        
        // Clear input
        inputField.text = "";
        
        // Show thinking indicator
        ShowThinkingIndicator();
    }
    
    void OnResponseReceived(string responseText)
    {
        HideThinkingIndicator();
        AddMessageToUI(agent.Name, responseText);
    }
    
    void AddMessageToUI(string sender, string message)
    {
        var messageObj = Instantiate(messagePrefab, messageContainer);
        messageObj.GetComponent<MessageBubble>().Setup(sender, message);
    }
}
```

### Voice Chat Interface

```csharp
public class VoiceChatInterface : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Button recordButton;
    [SerializeField] private TMP_Text transcriptionText;
    [SerializeField] private Image recordIndicator;
    
    void Start()
    {
        recordButton.onClick.AddListener(ToggleRecording);
        
        // Subscribe to transcription
        agent.onTranscriptionCompleted += OnTranscriptionCompleted;
    }
    
    void Update()
    {
        // Update recording indicator
        recordIndicator.enabled = agent.IsRecording;
    }
    
    void ToggleRecording()
    {
        if (!agent.IsRecording)
        {
            agent.StartTranscription();
        }
        else
        {
            agent.StopTranscription();
        }
    }
    
    void OnTranscriptionCompleted(string text)
    {
        transcriptionText.text = text;
        
        // Auto-send transcription
        agent.SendText(text);
    }
}
```

### Conversation Manager

```csharp
public class ConversationManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Dropdown conversationDropdown;
    [SerializeField] private Button newChatButton;
    [SerializeField] private Button saveButton;
    
    async void Start()
    {
        await agent.InitializeAsync();
        
        newChatButton.onClick.AddListener(CreateNewChat);
        saveButton.onClick.AddListener(SaveCurrentChat);
        conversationDropdown.onValueChanged.AddListener(LoadSelectedChat);
        
        await RefreshConversationList();
    }
    
    async void CreateNewChat()
    {
        Conversation newConversation = await agent.CreateConversationAsync();
        
        Debug.Log($"Created new chat: {newConversation.Id}");
        
        await RefreshConversationList();
    }
    
    async void LoadSelectedChat(int index)
    {
        string conversationId = conversationDropdown.options[index].text;
        
        Conversation conversation = await agent.LoadConversationAsync(conversationId);
        
        Debug.Log($"Loaded chat: {conversation.Messages.Count} messages");
        
        DisplayMessages(conversation.Messages);
    }
    
    async void SaveCurrentChat()
    {
        await agent.SaveConversationAsync();
        
        ShowNotification("Chat saved!");
    }
    
    async UniTask RefreshConversationList()
    {
        Conversation[] conversations = await agent.ListConversationsAsync();
        
        conversationDropdown.options = conversations
            .Select(c => new TMP_Dropdown.OptionData(c.Id))
            .ToList();
        
        conversationDropdown.RefreshShownValue();
    }
}
```

### Image Upload Interface

```csharp
public class ImageUploadInterface : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Button uploadButton;
    [SerializeField] private TMP_InputField promptField;
    [SerializeField] private Image previewImage;
    
    private Texture2D selectedImage;
    
    void Start()
    {
        uploadButton.onClick.AddListener(SelectImage);
    }
    
    void SelectImage()
    {
        // Open file picker (implementation depends on platform)
        string path = SelectImageFile();
        
        if (!string.IsNullOrEmpty(path))
        {
            byte[] imageData = File.ReadAllBytes(path);
            selectedImage = new Texture2D(2, 2);
            selectedImage.LoadImage(imageData);
            
            previewImage.sprite = Sprite.Create(
                selectedImage,
                new Rect(0, 0, selectedImage.width, selectedImage.height),
                Vector2.zero
            );
        }
    }
    
    public void SendImageWithPrompt()
    {
        string prompt = promptField.text;
        
        if (selectedImage != null)
        {
            // Add image attachment
            agent.AddAttachments(selectedImage);
            
            // Send with prompt
            agent.SendText(prompt);
            
            // Clear
            promptField.text = "";
            selectedImage = null;
            previewImage.sprite = null;
        }
    }
}
```

### Multi-File Upload

```csharp
public class MultiFileUpload : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Transform attachmentPanel;
    [SerializeField] private GameObject attachmentPrefab;
    
    private List<IFile> attachments = new();
    
    public void AddImageFile(Texture2D image)
    {
        var file = image.ToFile();
        attachments.Add(file);
        
        ShowAttachment(file.Name);
    }
    
    public void AddAudioFile(AudioClip clip)
    {
        var file = clip.ToFile();
        attachments.Add(file);
        
        ShowAttachment(file.Name);
    }
    
    public void SendWithAttachments(string message)
    {
        if (attachments.Count > 0)
        {
            agent.AddAttachments(attachments.ToArray());
        }
        
        agent.SendText(message);
        
        ClearAttachments();
    }
    
    void ShowAttachment(string fileName)
    {
        var attachmentObj = Instantiate(attachmentPrefab, attachmentPanel);
        attachmentObj.GetComponentInChildren<TMP_Text>().text = fileName;
    }
    
    void ClearAttachments()
    {
        attachments.Clear();
        
        foreach (Transform child in attachmentPanel)
        {
            Destroy(child.gameObject);
        }
    }
}
```

## Next Steps

- [AgentBehaviour Properties](agentbehaviour-properties.md)
- [Agent Properties](agent-properties.md)
- [Agent Methods](agent-methods.md)
- [Events & Hooks](../events-and-hooks/agent-hooks.md)
