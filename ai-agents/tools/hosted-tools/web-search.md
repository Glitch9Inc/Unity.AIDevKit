# Web Search (Hosted Tool)

Search the web and retrieve real-time information using Google Search.

## Overview

Web Search allows agents to:

- Search Google for current information
- Access real-time data
- Verify facts
- Find URLs and resources
- Get latest news and updates

## Basic Setup

### Enable Web Search

```csharp
// Add web search tool (Google AI only)
agent.AddTool(ToolType.WebSearch);

// Or for Google Gemini
agent.AddTool(ToolType.GoogleSearch);
```

## Simple Searches

### Ask Questions

```csharp
await agent.SendAsync("What's the current weather in Tokyo?");
await agent.SendAsync("What are the latest Unity releases?");
await agent.SendAsync("Who won the game awards 2024?");
```

### Get Current Information

```csharp
await agent.SendAsync("What's the current price of Bitcoin?");
await agent.SendAsync("What are today's trending topics on Twitter?");
await agent.SendAsync("Find the latest news about AI in gaming");
```

## Search Results

### Access Search Results

```csharp
agent.onResponseCompleted.AddListener(response =>
{
    if (response.SearchResults != null)
    {
        Debug.Log($"Found {response.SearchResults.Count} search results");
        
        foreach (var result in response.SearchResults)
        {
            Debug.Log($"Title: {result.Title}");
            Debug.Log($"URL: {result.Url}");
            Debug.Log($"Snippet: {result.Snippet}");
        }
    }
});
```

### Display Search Results

```csharp
public class SearchResultsDisplay : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject resultPrefab;
    [SerializeField] private Transform resultsContainer;
    
    void Start()
    {
        agent.onResponseCompleted.AddListener(DisplayResults);
    }
    
    void DisplayResults(Response response)
    {
        // Clear previous results
        foreach (Transform child in resultsContainer)
        {
            Destroy(child.gameObject);
        }
        
        // Display new results
        if (response.SearchResults != null)
        {
            foreach (var result in response.SearchResults)
            {
                var resultObj = Instantiate(resultPrefab, resultsContainer);
                var display = resultObj.GetComponent<SearchResultItem>();
                display.SetResult(result);
            }
        }
    }
}

public class SearchResultItem : MonoBehaviour
{
    [SerializeField] private TMP_Text titleText;
    [SerializeField] private TMP_Text urlText;
    [SerializeField] private TMP_Text snippetText;
    [SerializeField] private Button openButton;
    
    private SearchResult result;
    
    public void SetResult(SearchResult result)
    {
        this.result = result;
        
        titleText.text = result.Title;
        urlText.text = result.Url;
        snippetText.text = result.Snippet;
        
        openButton.onClick.AddListener(() => Application.OpenURL(result.Url));
    }
}
```

## Use Cases

### News and Updates

```csharp
public class NewsAssistant : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool(ToolType.WebSearch);
    }
    
    public async void GetLatestNews(string topic)
    {
        await agent.SendAsync($"What's the latest news about {topic}?");
    }
    
    public async void CheckUpdates(string software)
    {
        await agent.SendAsync($"What's the latest version of {software} and what's new?");
    }
}
```

### Fact Checking

```csharp
public async void VerifyFact(string claim)
{
    await agent.SendAsync($"Is this true: {claim}? Please verify with current information.");
}
```

### Research Assistant

```csharp
public class ResearchAssistant : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    void Start()
    {
        agent.AddTool(ToolType.WebSearch);
    }
    
    public async void Research(string topic)
    {
        await agent.SendAsync($@"
Research {topic} and provide:
1. Key facts and statistics
2. Recent developments
3. Expert opinions
4. Relevant resources and links
");
    }
    
    public async void CompareOptions(string option1, string option2)
    {
        await agent.SendAsync($@"
Compare {option1} vs {option2}:
1. Pros and cons of each
2. Current market prices
3. User reviews and ratings
4. Which is better for what use case
");
    }
}
```

## Game Development Use Cases

### Check Asset Store

```csharp
public async void FindAssets(string assetType)
{
    await agent.SendAsync($@"
Find Unity Asset Store packages for {assetType}:
1. Top rated options
2. Price range
3. Key features
4. User reviews
Provide direct links.
");
}
```

### Look Up Documentation

```csharp
public async void FindDocumentation(string topic)
{
    await agent.SendAsync($@"
Find documentation for {topic}:
1. Official Unity docs link
2. Tutorial links
3. Community resources
4. Example projects
");
}
```

### Check Compatibility

```csharp
public async void CheckCompatibility(string plugin, string unityVersion)
{
    await agent.SendAsync($@"
Is {plugin} compatible with Unity {unityVersion}?
Check the official page and recent reports.
");
}
```

## Advanced Usage

### Custom Search Queries

```csharp
public class AdvancedSearchAgent : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    public async void SearchWithFilters(string query, SearchFilters filters)
    {
        string searchQuery = BuildSearchQuery(query, filters);
        await agent.SendAsync($"Search for: {searchQuery}");
    }
    
    string BuildSearchQuery(string query, SearchFilters filters)
    {
        var parts = new List<string> { query };
        
        if (filters.Site != null)
            parts.Add($"site:{filters.Site}");
        
        if (filters.FileType != null)
            parts.Add($"filetype:{filters.FileType}");
        
        if (filters.DateRange != null)
            parts.Add($"after:{filters.DateRange}");
        
        return string.Join(" ", parts);
    }
}

[System.Serializable]
public class SearchFilters
{
    public string Site;      // e.g., "unity3d.com"
    public string FileType;  // e.g., "pdf"
    public string DateRange; // e.g., "2024-01-01"
}
```

### Search Multiple Sources

```csharp
public async void MultiSourceSearch(string topic)
{
    await agent.SendAsync($@"
Search for information about {topic} from:
1. Official documentation
2. Stack Overflow discussions
3. Reddit community posts
4. Recent blog articles
5. Video tutorials

Summarize findings from each source.
");
}
```

## Rate Limiting

### Handle Rate Limits

```csharp
public class SearchRateLimiter : MonoBehaviour
{
    [SerializeField] private int maxSearchesPerMinute = 10;
    
    private Queue<DateTime> searchTimes = new();
    
    public bool CanSearch()
    {
        // Remove old entries
        var cutoff = DateTime.Now.AddMinutes(-1);
        while (searchTimes.Count > 0 && searchTimes.Peek() < cutoff)
        {
            searchTimes.Dequeue();
        }
        
        return searchTimes.Count < maxSearchesPerMinute;
    }
    
    public void RecordSearch()
    {
        searchTimes.Enqueue(DateTime.Now);
    }
    
    public async UniTask<bool> SearchWithRateLimit(string query)
    {
        if (!CanSearch())
        {
            Debug.LogWarning("Search rate limit reached");
            return false;
        }
        
        RecordSearch();
        await agent.SendAsync(query);
        return true;
    }
}
```

## Caching

### Cache Search Results

```csharp
public class SearchCache : MonoBehaviour
{
    private Dictionary<string, CachedResult> cache = new();
    private TimeSpan cacheExpiry = TimeSpan.FromHours(1);
    
    public async UniTask<Response> SearchWithCache(string query)
    {
        // Check cache
        if (cache.TryGetValue(query, out var cached))
        {
            if (DateTime.Now - cached.Timestamp < cacheExpiry)
            {
                Debug.Log("📦 Using cached result");
                return cached.Response;
            }
        }
        
        // Perform search
        var response = await agent.SendAsync(query);
        
        // Cache result
        cache[query] = new CachedResult
        {
            Response = response,
            Timestamp = DateTime.Now
        };
        
        return response;
    }
    
    class CachedResult
    {
        public Response Response;
        public DateTime Timestamp;
    }
}
```

## Error Handling

### Handle Search Errors

```csharp
agent.onToolCallError.AddListener((toolCall, error) =>
{
    if (toolCall.Function.Name.Contains("search"))
    {
        Debug.LogError($"Search failed: {error}");
        
        if (error.Contains("rate limit"))
        {
            ShowMessage("Too many searches. Please wait a moment.");
        }
        else if (error.Contains("timeout"))
        {
            ShowMessage("Search timed out. Please try again.");
        }
        else
        {
            ShowMessage("Search temporarily unavailable.");
        }
    }
});
```

## Best Practices

### 1. Be Specific

```csharp
// ❌ Vague
await agent.SendAsync("Find Unity stuff");

// ✅ Specific
await agent.SendAsync("Find Unity 2023 LTS features and changes");
```

### 2. Request Sources

```csharp
await agent.SendAsync(@"
What's the latest on Unity 6?
Please provide official sources and links.
");
```

### 3. Set Context

```csharp
await agent.SendAsync(@"
I'm developing a mobile game with Unity.
Find the best practices for mobile optimization in 2024.
Include official Unity recommendations.
");
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class WebSearchManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private int maxSearchesPerMinute = 10;
    
    private Queue<DateTime> searchHistory = new();
    private Dictionary<string, CachedResult> cache = new();
    
    void Start()
    {
        SetupWebSearch();
    }
    
    void SetupWebSearch()
    {
        agent.AddTool(ToolType.WebSearch);
        agent.onResponseCompleted.AddListener(OnSearchCompleted);
        agent.onToolCallError.AddListener(OnSearchError);
        
        Debug.Log("✓ Web search ready");
    }
    
    public async void Search(string query)
    {
        Debug.Log($"🔍 Searching: {query}");
        
        // Check rate limit
        if (!CanSearch())
        {
            ShowMessage("Too many searches. Please wait.");
            return;
        }
        
        // Check cache
        if (TryGetCached(query, out var cached))
        {
            Debug.Log("📦 Using cached result");
            DisplayResults(cached);
            return;
        }
        
        try
        {
            // Perform search
            RecordSearch();
            await agent.SendAsync(query);
        }
        catch (Exception ex)
        {
            Debug.LogError($"Search failed: {ex.Message}");
            ShowMessage("Search failed. Please try again.");
        }
    }
    
    bool CanSearch()
    {
        var cutoff = DateTime.Now.AddMinutes(-1);
        
        while (searchHistory.Count > 0 && searchHistory.Peek() < cutoff)
        {
            searchHistory.Dequeue();
        }
        
        return searchHistory.Count < maxSearchesPerMinute;
    }
    
    void RecordSearch()
    {
        searchHistory.Enqueue(DateTime.Now);
    }
    
    bool TryGetCached(string query, out Response cached)
    {
        if (cache.TryGetValue(query, out var result))
        {
            if (DateTime.Now - result.Timestamp < TimeSpan.FromHours(1))
            {
                cached = result.Response;
                return true;
            }
        }
        
        cached = null;
        return false;
    }
    
    void OnSearchCompleted(Response response)
    {
        Debug.Log("✓ Search completed");
        
        // Cache result
        string query = GetLastQuery();
        if (!string.IsNullOrEmpty(query))
        {
            cache[query] = new CachedResult
            {
                Response = response,
                Timestamp = DateTime.Now
            };
        }
        
        // Display results
        DisplayResults(response);
    }
    
    void DisplayResults(Response response)
    {
        if (response.SearchResults != null)
        {
            Debug.Log($"📋 {response.SearchResults.Count} result(s):");
            
            foreach (var result in response.SearchResults)
            {
                Debug.Log($"  • {result.Title}");
                Debug.Log($"    {result.Url}");
            }
        }
    }
    
    void OnSearchError(ToolCall toolCall, string error)
    {
        if (toolCall.Function.Name.Contains("search"))
        {
            Debug.LogError($"Search error: {error}");
            
            if (error.Contains("rate limit"))
            {
                ShowMessage("Rate limit exceeded. Please wait.");
            }
            else
            {
                ShowMessage("Search failed. Please try again.");
            }
        }
    }
    
    string GetLastQuery()
    {
        // Get last user message
        var messages = agent.Conversation.Messages;
        var lastMessage = messages.LastOrDefault(m => m.Role == "user");
        return lastMessage?.Content ?? "";
    }
    
    void ShowMessage(string message)
    {
        Debug.Log($"ℹ️ {message}");
        // Show UI notification
    }
    
    class CachedResult
    {
        public Response Response;
        public DateTime Timestamp;
    }
}
```

## Limitations

### What's Available

- ✅ Current information (news, prices, etc.)
- ✅ Public websites and pages
- ✅ Recent updates and changes
- ✅ Factual information
- ❌ Content behind paywalls
- ❌ Private or restricted sites
- ❌ Real-time chat/social media streams

### Rate Limits

```csharp
// Google Search has rate limits
// - Implement request throttling
// - Cache results when possible
// - Handle 429 errors gracefully
```

## Next Steps

- [File Search](file-search.md)
- [Code Interpreter](code-interpreter.md)
- [Google Tools](../google-tools/)
