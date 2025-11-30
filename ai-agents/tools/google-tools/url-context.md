# URL Context

Extract and use content from web URLs.

## Overview

URL Context allows agents to:

- Read webpage content
- Extract article text
- Access online documentation
- Parse GitHub files
- Process web resources

## Basic Setup

### Enable URL Context

```csharp
agent.AddGoogleTool(GoogleToolType.UrlContext);
```

## Read URLs

### Simple URL Access

```csharp
await agent.SendAsync("Read the content from https://docs.unity3d.com/Manual/index.html");
await agent.SendAsync("What does this page say: https://github.com/user/repo/README.md");
```

### Extract Specific Information

```csharp
await agent.SendAsync(@"
Read https://docs.unity3d.com/ScriptReference/Vector3.html
and explain the Vector3.Lerp method
");
```

## Use Cases

### Documentation Lookup

```csharp
public async void LookupUnityAPI(string apiName)
{
    string url = $"https://docs.unity3d.com/ScriptReference/{apiName}.html";
    
    await agent.SendAsync($@"
Read the Unity documentation at: {url}
and provide:
1. Brief description
2. Key methods
3. Code example
4. Common use cases
");
}

// Usage
LookupUnityAPI("NavMeshAgent");
LookupUnityAPI("Rigidbody");
```

### GitHub File Reading

```csharp
public async void ReadGitHubFile(string owner, string repo, string filePath)
{
    string url = $"https://raw.githubusercontent.com/{owner}/{repo}/main/{filePath}";
    
    await agent.SendAsync($@"
Read the file from: {url}
and explain what this code does
");
}

// Usage
ReadGitHubFile("Cysharp", "UniTask", "README.md");
```

### Tutorial Processing

```csharp
public async void ProcessTutorial(string tutorialUrl)
{
    await agent.SendAsync($@"
Read the tutorial at: {tutorialUrl}

Provide:
1. Summary of key concepts
2. Step-by-step breakdown
3. Code examples mentioned
4. Prerequisites
");
}

// Usage
ProcessTutorial("https://learn.unity.com/tutorial/working-with-addressables");
```

## Multiple URLs

### Compare Documentation

```csharp
public async void CompareAPIs(string url1, string url2)
{
    await agent.SendAsync($@"
Read both:
1. {url1}
2. {url2}

Compare and contrast these two APIs
");
}

// Usage
CompareAPIs(
    "https://docs.unity3d.com/ScriptReference/Coroutine.html",
    "https://github.com/Cysharp/UniTask"
);
```

### Aggregate Information

```csharp
public async void AggregateResources(string[] urls)
{
    string urlList = string.Join("\n", urls.Select((url, i) => $"{i + 1}. {url}"));
    
    await agent.SendAsync($@"
Read all these resources:
{urlList}

Provide a comprehensive summary combining information from all sources
");
}
```

## URL Parsing

### Parse Release Notes

```csharp
public async void ParseReleaseNotes(string packageName, string version)
{
    string url = $"https://github.com/{packageName}/releases/tag/v{version}";
    
    await agent.SendAsync($@"
Read release notes from: {url}

Extract:
1. New features
2. Bug fixes
3. Breaking changes
4. Migration guide
");
}

// Usage
ParseReleaseNotes("Unity-Technologies/Addressables-Sample", "1.21.2");
```

### Extract Code Examples

```csharp
public async void ExtractCodeExamples(string docUrl)
{
    await agent.SendAsync($@"
Read: {docUrl}

Extract all code examples and explain each one
");
}
```

## Content Filtering

### Extract Specific Sections

```csharp
public async void ExtractSection(string url, string sectionName)
{
    await agent.SendAsync($@"
Read: {url}

Find and extract only the '{sectionName}' section
");
}

// Usage
ExtractSection(
    "https://docs.unity3d.com/Manual/BestPracticeUnderstandingPerformanceInUnity.html",
    "Memory Management"
);
```

### Get Table of Contents

```csharp
public async void GetTableOfContents(string url)
{
    await agent.SendAsync($@"
Read: {url}

Extract the table of contents or main sections
");
}
```

## Integration with Game Features

### Dynamic Help System

```csharp
public class ContextualHelp : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private Dictionary<string, string> helpUrls = new()
    {
        { "inventory", "https://docs.unity.com/example/inventory-system" },
        { "combat", "https://docs.unity.com/example/combat-mechanics" },
        { "quests", "https://docs.unity.com/example/quest-system" }
    };
    
    public async void ShowHelp(string topic)
    {
        if (!helpUrls.TryGetValue(topic, out string url))
        {
            Debug.LogWarning($"No help URL for topic: {topic}");
            return;
        }
        
        await agent.SendAsync($@"
Read the documentation at: {url}

Provide a concise explanation suitable for in-game help
");
    }
}

// Usage
contextualHelp.ShowHelp("inventory");
```

### Asset Documentation Reader

```csharp
public async void ReadAssetDocs(string assetUrl)
{
    await agent.SendAsync($@"
Read asset documentation from: {assetUrl}

Provide:
1. Installation instructions
2. Quick start guide
3. Key features
4. API overview
");
}
```

## Error Handling

### Handle URL Errors

```csharp
agent.onUrlContextError.AddListener(error =>
{
    Debug.LogError($"URL context failed: {error}");
    
    if (error.Contains("not_found"))
    {
        ShowMessage("URL not found (404)");
    }
    else if (error.Contains("forbidden"))
    {
        ShowMessage("Access forbidden (403)");
    }
    else if (error.Contains("timeout"))
    {
        ShowMessage("Request timed out");
    }
    else
    {
        ShowMessage("Failed to read URL");
    }
});
```

## Caching

### Cache URL Content

```csharp
public class UrlContentCache : MonoBehaviour
{
    private Dictionary<string, CachedContent> cache = new();
    private TimeSpan cacheExpiry = TimeSpan.FromHours(24);
    
    public async UniTask<string> GetOrFetch(AgentBehaviour agent, string url)
    {
        if (cache.TryGetValue(url, out var cached))
        {
            if (DateTime.Now - cached.Timestamp < cacheExpiry)
            {
                Debug.Log("Using cached URL content");
                return cached.Content;
            }
        }
        
        // Fetch new content
        string content = await FetchUrl(agent, url);
        
        cache[url] = new CachedContent
        {
            Content = content,
            Timestamp = DateTime.Now
        };
        
        return content;
    }
    
    async UniTask<string> FetchUrl(AgentBehaviour agent, string url)
    {
        // Implementation
        return "";
    }
    
    struct CachedContent
    {
        public string Content;
        public DateTime Timestamp;
    }
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class UrlContextManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_InputField urlInput;
    [SerializeField] private Button fetchButton;
    [SerializeField] private TMP_Text contentDisplay;
    
    private Dictionary<string, string> cache = new();
    
    async void Start()
    {
        await SetupUrlContext();
        
        fetchButton.onClick.AddListener(OnFetchClicked);
    }
    
    async UniTask SetupUrlContext()
    {
        // Add URL Context tool
        agent.AddGoogleTool(GoogleToolType.UrlContext);
        
        // Listen for events
        agent.onUrlContextCompleted.AddListener(OnContentFetched);
        agent.onUrlContextError.AddListener(OnFetchError);
        
        Debug.Log("✓ URL Context ready");
    }
    
    async void OnFetchClicked()
    {
        string url = urlInput.text.Trim();
        
        if (string.IsNullOrEmpty(url))
        {
            ShowMessage("Please enter a URL");
            return;
        }
        
        if (!IsValidUrl(url))
        {
            ShowMessage("Invalid URL format");
            return;
        }
        
        // Check cache
        if (cache.TryGetValue(url, out var cachedContent))
        {
            Debug.Log("Using cached content");
            DisplayContent(cachedContent);
            return;
        }
        
        // Fetch content
        Debug.Log($"📄 Fetching: {url}");
        fetchButton.interactable = false;
        contentDisplay.text = "Loading...";
        
        await agent.SendAsync($"Read and summarize the content from: {url}");
    }
    
    void OnContentFetched(string content)
    {
        Debug.Log($"✓ Content fetched ({content.Length} chars)");
        
        // Cache
        string url = urlInput.text.Trim();
        cache[url] = content;
        
        // Display
        DisplayContent(content);
        
        fetchButton.interactable = true;
    }
    
    void DisplayContent(string content)
    {
        // Limit display length
        const int maxLength = 5000;
        
        if (content.Length > maxLength)
        {
            content = content.Substring(0, maxLength) + "\n\n[Content truncated...]";
        }
        
        contentDisplay.text = content;
    }
    
    void OnFetchError(string error)
    {
        Debug.LogError($"Fetch error: {error}");
        contentDisplay.text = $"<color=red>Error: {error}</color>";
        fetchButton.interactable = true;
    }
    
    bool IsValidUrl(string url)
    {
        return Uri.TryCreate(url, UriKind.Absolute, out var uri) &&
               (uri.Scheme == Uri.UriSchemeHttp || uri.Scheme == Uri.UriSchemeHttps);
    }
    
    void ShowMessage(string message)
    {
        contentDisplay.text = message;
    }
}
```

## Next Steps

- [Google Search](google-search.md)
- [Code Execution](code-execution.md)
- [Web Search (Hosted)](../hosted-tools/web-search.md)
