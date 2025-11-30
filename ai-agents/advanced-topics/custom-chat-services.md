# Custom Chat Services

Create custom chat service implementations.

## Overview

Custom Chat Services enable:

- Custom API integrations
- Provider abstraction
- Request/response handling
- Streaming support
- Error management

## Base Chat Service

### IChatService Interface

```csharp
public interface IChatService
{
    Task<ChatResponse> SendAsync(ChatRequest request);
    Task<ChatResponse> SendStreamingAsync(ChatRequest request, Action<string> onDelta);
    bool SupportsStreaming { get; }
    string ProviderName { get; }
}

public class ChatRequest
{
    public string Model { get; set; }
    public List<ChatMessage> Messages { get; set; }
    public float Temperature { get; set; }
    public int MaxTokens { get; set; }
    public List<ToolDefinition> Tools { get; set; }
}

public class ChatResponse
{
    public string Content { get; set; }
    public List<ToolCall> ToolCalls { get; set; }
    public int PromptTokens { get; set; }
    public int CompletionTokens { get; set; }
    public string Model { get; set; }
    public string FinishReason { get; set; }
}

public class ChatMessage
{
    public string Role { get; set; }
    public string Content { get; set; }
}
```

## Custom Service Implementation

### OpenAI-Compatible Service

```csharp
using UnityEngine;
using UnityEngine.Networking;
using Cysharp.Threading.Tasks;

public class OpenAICompatibleService : IChatService
{
    private string apiKey;
    private string baseUrl;
    
    public bool SupportsStreaming => true;
    public string ProviderName => "OpenAI Compatible";
    
    public OpenAICompatibleService(string apiKey, string baseUrl = "https://api.openai.com/v1")
    {
        this.apiKey = apiKey;
        this.baseUrl = baseUrl;
    }
    
    public async Task<ChatResponse> SendAsync(ChatRequest request)
    {
        string url = $"{baseUrl}/chat/completions";
        
        // Build request body
        var body = new
        {
            model = request.Model,
            messages = request.Messages.Select(m => new { role = m.Role, content = m.Content }),
            temperature = request.Temperature,
            max_tokens = request.MaxTokens,
            tools = request.Tools?.Select(t => new
            {
                type = "function",
                function = new
                {
                    name = t.Name,
                    description = t.Description,
                    parameters = t.Parameters
                }
            })
        };
        
        string json = JsonUtility.ToJson(body);
        
        // Send request
        using var webRequest = UnityWebRequest.Post(url, json, "application/json");
        webRequest.SetRequestHeader("Authorization", $"Bearer {apiKey}");
        
        await webRequest.SendWebRequest();
        
        if (webRequest.result != UnityWebRequest.Result.Success)
        {
            throw new Exception($"Request failed: {webRequest.error}");
        }
        
        // Parse response
        string responseJson = webRequest.downloadHandler.text;
        return ParseResponse(responseJson);
    }
    
    public async Task<ChatResponse> SendStreamingAsync(ChatRequest request, Action<string> onDelta)
    {
        string url = $"{baseUrl}/chat/completions";
        
        var body = new
        {
            model = request.Model,
            messages = request.Messages.Select(m => new { role = m.Role, content = m.Content }),
            temperature = request.Temperature,
            max_tokens = request.MaxTokens,
            stream = true
        };
        
        string json = JsonUtility.ToJson(body);
        
        using var webRequest = UnityWebRequest.Post(url, json, "application/json");
        webRequest.SetRequestHeader("Authorization", $"Bearer {apiKey}");
        
        // Handle streaming
        var handler = new StreamingDownloadHandler(onDelta);
        webRequest.downloadHandler = handler;
        
        await webRequest.SendWebRequest();
        
        if (webRequest.result != UnityWebRequest.Result.Success)
        {
            throw new Exception($"Request failed: {webRequest.error}");
        }
        
        return handler.GetFinalResponse();
    }
    
    ChatResponse ParseResponse(string json)
    {
        // Parse JSON response
        var response = JsonUtility.FromJson<OpenAIResponse>(json);
        
        return new ChatResponse
        {
            Content = response.choices[0].message.content,
            ToolCalls = ParseToolCalls(response.choices[0].message.tool_calls),
            PromptTokens = response.usage.prompt_tokens,
            CompletionTokens = response.usage.completion_tokens,
            Model = response.model,
            FinishReason = response.choices[0].finish_reason
        };
    }
    
    List<ToolCall> ParseToolCalls(object[] toolCalls)
    {
        if (toolCalls == null)
            return null;
        
        // Parse tool calls from response
        return new List<ToolCall>();
    }
}

// Response DTOs
[System.Serializable]
public class OpenAIResponse
{
    public string id;
    public string model;
    public Choice[] choices;
    public Usage usage;
}

[System.Serializable]
public class Choice
{
    public Message message;
    public string finish_reason;
}

[System.Serializable]
public class Message
{
    public string role;
    public string content;
    public object[] tool_calls;
}

[System.Serializable]
public class Usage
{
    public int prompt_tokens;
    public int completion_tokens;
    public int total_tokens;
}
```

### Custom Streaming Handler

```csharp
public class StreamingDownloadHandler : DownloadHandlerScript
{
    private Action<string> onDelta;
    private StringBuilder fullContent = new();
    
    public StreamingDownloadHandler(Action<string> onDelta)
    {
        this.onDelta = onDelta;
    }
    
    protected override bool ReceiveData(byte[] data, int dataLength)
    {
        if (data == null || dataLength == 0)
            return false;
        
        string text = System.Text.Encoding.UTF8.GetString(data, 0, dataLength);
        
        // Parse SSE format
        var lines = text.Split('\n');
        
        foreach (var line in lines)
        {
            if (line.StartsWith("data: "))
            {
                string json = line.Substring(6);
                
                if (json == "[DONE]")
                    continue;
                
                // Parse delta
                var delta = ParseDelta(json);
                
                if (!string.IsNullOrEmpty(delta))
                {
                    fullContent.Append(delta);
                    onDelta?.Invoke(delta);
                }
            }
        }
        
        return true;
    }
    
    string ParseDelta(string json)
    {
        try
        {
            var response = JsonUtility.FromJson<StreamResponse>(json);
            return response.choices[0].delta.content;
        }
        catch
        {
            return null;
        }
    }
    
    public ChatResponse GetFinalResponse()
    {
        return new ChatResponse
        {
            Content = fullContent.ToString()
        };
    }
}

[System.Serializable]
public class StreamResponse
{
    public StreamChoice[] choices;
}

[System.Serializable]
public class StreamChoice
{
    public Delta delta;
}

[System.Serializable]
public class Delta
{
    public string content;
}
```

## Google Gemini Service

### Custom Gemini Implementation

```csharp
public class GeminiChatService : IChatService
{
    private string apiKey;
    private string baseUrl = "https://generativelanguage.googleapis.com/v1beta";
    
    public bool SupportsStreaming => true;
    public string ProviderName => "Google Gemini";
    
    public GeminiChatService(string apiKey)
    {
        this.apiKey = apiKey;
    }
    
    public async Task<ChatResponse> SendAsync(ChatRequest request)
    {
        string url = $"{baseUrl}/models/{request.Model}:generateContent?key={apiKey}";
        
        // Convert to Gemini format
        var body = new
        {
            contents = request.Messages.Select(m => new
            {
                role = ConvertRole(m.Role),
                parts = new[] { new { text = m.Content } }
            }),
            generationConfig = new
            {
                temperature = request.Temperature,
                maxOutputTokens = request.MaxTokens
            }
        };
        
        string json = JsonUtility.ToJson(body);
        
        using var webRequest = UnityWebRequest.Post(url, json, "application/json");
        
        await webRequest.SendWebRequest();
        
        if (webRequest.result != UnityWebRequest.Result.Success)
        {
            throw new Exception($"Request failed: {webRequest.error}");
        }
        
        string responseJson = webRequest.downloadHandler.text;
        return ParseGeminiResponse(responseJson);
    }
    
    public async Task<ChatResponse> SendStreamingAsync(ChatRequest request, Action<string> onDelta)
    {
        string url = $"{baseUrl}/models/{request.Model}:streamGenerateContent?key={apiKey}&alt=sse";
        
        var body = new
        {
            contents = request.Messages.Select(m => new
            {
                role = ConvertRole(m.Role),
                parts = new[] { new { text = m.Content } }
            })
        };
        
        string json = JsonUtility.ToJson(body);
        
        using var webRequest = UnityWebRequest.Post(url, json, "application/json");
        
        var handler = new GeminiStreamingHandler(onDelta);
        webRequest.downloadHandler = handler;
        
        await webRequest.SendWebRequest();
        
        if (webRequest.result != UnityWebRequest.Result.Success)
        {
            throw new Exception($"Request failed: {webRequest.error}");
        }
        
        return handler.GetFinalResponse();
    }
    
    string ConvertRole(string role)
    {
        return role.ToLower() switch
        {
            "user" => "user",
            "assistant" => "model",
            "system" => "user", // Gemini doesn't have system role
            _ => "user"
        };
    }
    
    ChatResponse ParseGeminiResponse(string json)
    {
        var response = JsonUtility.FromJson<GeminiResponse>(json);
        
        return new ChatResponse
        {
            Content = response.candidates[0].content.parts[0].text,
            PromptTokens = response.usageMetadata.promptTokenCount,
            CompletionTokens = response.usageMetadata.candidatesTokenCount,
            Model = "gemini-pro",
            FinishReason = response.candidates[0].finishReason
        };
    }
}

[System.Serializable]
public class GeminiResponse
{
    public Candidate[] candidates;
    public UsageMetadata usageMetadata;
}

[System.Serializable]
public class Candidate
{
    public Content content;
    public string finishReason;
}

[System.Serializable]
public class Content
{
    public Part[] parts;
}

[System.Serializable]
public class Part
{
    public string text;
}

[System.Serializable]
public class UsageMetadata
{
    public int promptTokenCount;
    public int candidatesTokenCount;
    public int totalTokenCount;
}

public class GeminiStreamingHandler : DownloadHandlerScript
{
    private Action<string> onDelta;
    private StringBuilder fullContent = new();
    
    public GeminiStreamingHandler(Action<string> onDelta)
    {
        this.onDelta = onDelta;
    }
    
    protected override bool ReceiveData(byte[] data, int dataLength)
    {
        string text = System.Text.Encoding.UTF8.GetString(data, 0, dataLength);
        
        // Parse Gemini streaming format
        var lines = text.Split('\n');
        
        foreach (var line in lines)
        {
            if (line.StartsWith("data: "))
            {
                string json = line.Substring(6);
                
                try
                {
                    var response = JsonUtility.FromJson<GeminiResponse>(json);
                    string delta = response.candidates[0].content.parts[0].text;
                    
                    if (!string.IsNullOrEmpty(delta))
                    {
                        fullContent.Append(delta);
                        onDelta?.Invoke(delta);
                    }
                }
                catch { }
            }
        }
        
        return true;
    }
    
    public ChatResponse GetFinalResponse()
    {
        return new ChatResponse
        {
            Content = fullContent.ToString()
        };
    }
}
```

## Service Manager

### Manage Multiple Services

```csharp
public class ChatServiceManager
{
    private Dictionary<string, IChatService> services = new();
    private string activeService;
    
    public void RegisterService(string name, IChatService service)
    {
        services[name] = service;
        Debug.Log($"✓ Chat service registered: {name}");
    }
    
    public void SetActiveService(string name)
    {
        if (!services.ContainsKey(name))
        {
            throw new InvalidOperationException($"Service not found: {name}");
        }
        
        activeService = name;
        Debug.Log($"Active service: {name}");
    }
    
    public IChatService GetActiveService()
    {
        if (string.IsNullOrEmpty(activeService))
        {
            throw new InvalidOperationException("No active service set");
        }
        
        return services[activeService];
    }
    
    public IChatService GetService(string name)
    {
        if (services.TryGetValue(name, out var service))
        {
            return service;
        }
        
        throw new InvalidOperationException($"Service not found: {name}");
    }
    
    public List<string> GetAvailableServices()
    {
        return services.Keys.ToList();
    }
}
```

## Service Integration

### Use Custom Service with Agent

```csharp
public class AgentWithCustomService : MonoBehaviour
{
    [SerializeField] private string openAIApiKey;
    [SerializeField] private string geminiApiKey;
    
    private Agent agent;
    private ChatServiceManager serviceManager;
    
    void Start()
    {
        InitializeServices();
        InitializeAgent();
    }
    
    void InitializeServices()
    {
        serviceManager = new ChatServiceManager();
        
        // Register OpenAI
        var openAIService = new OpenAICompatibleService(openAIApiKey);
        serviceManager.RegisterService("openai", openAIService);
        
        // Register Gemini
        var geminiService = new GeminiChatService(geminiApiKey);
        serviceManager.RegisterService("gemini", geminiService);
        
        // Set default
        serviceManager.SetActiveService("openai");
        
        Debug.Log("✓ Chat services initialized");
    }
    
    void InitializeAgent()
    {
        agent = new Agent();
        
        // Configure agent to use custom service
        agent.ChatService = serviceManager.GetActiveService();
        
        agent.Initialize();
        agent.Start();
    }
    
    public void SwitchToGemini()
    {
        serviceManager.SetActiveService("gemini");
        agent.ChatService = serviceManager.GetActiveService();
        
        Debug.Log("Switched to Gemini");
    }
    
    public void SwitchToOpenAI()
    {
        serviceManager.SetActiveService("openai");
        agent.ChatService = serviceManager.GetActiveService();
        
        Debug.Log("Switched to OpenAI");
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class CustomChatServiceDemo : MonoBehaviour
{
    [SerializeField] private string apiKey;
    [SerializeField] private TMP_Text responseText;
    [SerializeField] private TMP_InputField inputField;
    
    private Agent agent;
    private IChatService chatService;
    
    void Start()
    {
        InitializeCustomService();
        InitializeAgent();
    }
    
    void InitializeCustomService()
    {
        // Create custom service
        chatService = new OpenAICompatibleService(apiKey);
        
        Debug.Log($"✓ Custom service initialized: {chatService.ProviderName}");
    }
    
    void InitializeAgent()
    {
        agent = new Agent();
        agent.ChatService = chatService;
        
        agent.Initialize();
        agent.Start();
        
        Debug.Log("✓ Agent initialized with custom service");
    }
    
    public async void SendMessage()
    {
        string message = inputField.text;
        
        if (string.IsNullOrEmpty(message))
            return;
        
        responseText.text = "Thinking...";
        
        try
        {
            // Build request
            var request = new ChatRequest
            {
                Model = "gpt-4",
                Messages = new List<ChatMessage>
                {
                    new ChatMessage { Role = "user", Content = message }
                },
                Temperature = 0.7f,
                MaxTokens = 1000
            };
            
            // Send with streaming
            await chatService.SendStreamingAsync(request, OnTextDelta);
            
            Debug.Log("✓ Response completed");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Error: {ex.Message}");
            responseText.text = $"Error: {ex.Message}";
        }
    }
    
    void OnTextDelta(string delta)
    {
        if (responseText.text == "Thinking...")
        {
            responseText.text = "";
        }
        
        responseText.text += delta;
    }
}
```

## Next Steps

- [Agent Services](agent-services.md)
- [Controllers](controllers.md)
- [Event Router](event-router.md)
- [API Reference](../api-reference/agent-properties.md)
