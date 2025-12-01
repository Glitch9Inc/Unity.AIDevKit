# File Search (Hosted Tool)

Search and retrieve information from uploaded files using OpenAI's File Search tool.

## Overview

File Search allows agents to:

- Search through uploaded documents
- Answer questions about file contents
- Extract specific information
- Summarize documents
- Find relevant passages

## Basic Setup

### Enable File Search

```csharp
// Add file search tool
agent.AddTool(ToolType.FileSearch);
```

### Upload Files

```csharp
// Upload files for searching
var fileIds = await agent.UploadFilesAsync(new[]
{
    "path/to/document.pdf",
    "path/to/manual.txt",
    "path/to/data.json"
});

// Associate with vector store
await agent.CreateVectorStoreAsync(fileIds);
```

## File Upload

### Upload Single File

```csharp
string fileId = await agent.UploadFileAsync("documentation.pdf");
Debug.Log($"File uploaded: {fileId}");
```

### Upload Multiple Files

```csharp
string[] filePaths = {
    "manual.pdf",
    "guide.txt",
    "reference.md"
};

var fileIds = await agent.UploadFilesAsync(filePaths);
Debug.Log($"Uploaded {fileIds.Length} files");
```

### Supported Formats

```csharp
// Supported file types:
// - PDF (.pdf)
// - Text (.txt, .md)
// - Word (.doc, .docx)
// - PowerPoint (.ppt, .pptx)
// - HTML (.html)
// - JSON (.json)
// - CSV (.csv)

bool IsSupported(string filePath)
{
    string extension = Path.GetExtension(filePath).ToLower();
    string[] supported = { ".pdf", ".txt", ".md", ".doc", ".docx", ".json", ".csv" };
    return supported.Contains(extension);
}
```

## Vector Store

### Create Vector Store

```csharp
var vectorStore = await agent.CreateVectorStoreAsync(
    name: "Game Documentation",
    fileIds: fileIds
);

Debug.Log($"Vector store created: {vectorStore.Id}");
```

### Update Vector Store

```csharp
// Add more files
await agent.AddFilesToVectorStoreAsync(
    vectorStore.Id,
    newFileIds
);

// Remove files
await agent.RemoveFilesFromVectorStoreAsync(
    vectorStore.Id,
    oldFileIds
);
```

### List Vector Stores

```csharp
var stores = await agent.ListVectorStoresAsync();

foreach (var store in stores)
{
    Debug.Log($"Store: {store.Name}");
    Debug.Log($"  Files: {store.FileCount}");
    Debug.Log($"  Created: {store.CreatedAt}");
}
```

## Searching Files

### Ask Questions

```csharp
// Agent can now search files automatically
await agent.SendAsync("What are the system requirements?");

// Response includes citations from files
```

### Manual Search

```csharp
var results = await agent.SearchFilesAsync(
    query: "player health mechanics",
    maxResults: 5
);

foreach (var result in results)
{
    Debug.Log($"Found in: {result.FileName}");
    Debug.Log($"Content: {result.Content}");
    Debug.Log($"Relevance: {result.Score}");
}
```

## Citations

### Access Citations

```csharp
agent.onResponseCompleted.AddListener(response =>
{
    if (response.Citations != null && response.Citations.Count > 0)
    {
        Debug.Log($"Found {response.Citations.Count} citations");
        
        foreach (var citation in response.Citations)
        {
            Debug.Log($"Source: {citation.FileName}");
            Debug.Log($"Quote: {citation.Quote}");
            Debug.Log($"Page: {citation.PageNumber}");
        }
    }
});
```

### Display Citations UI

```csharp
public class CitationDisplay : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private GameObject citationPrefab;
    [SerializeField] private Transform citationsContainer;
    
    void Start()
    {
        agent.onResponseCompleted.AddListener(OnResponseCompleted);
    }
    
    void OnResponseCompleted(Response response)
    {
        // Clear previous citations
        foreach (Transform child in citationsContainer)
        {
            Destroy(child.gameObject);
        }
        
        // Display new citations
        if (response.Citations != null)
        {
            foreach (var citation in response.Citations)
            {
                var citationObj = Instantiate(citationPrefab, citationsContainer);
                var display = citationObj.GetComponent<CitationItem>();
                display.SetCitation(citation);
            }
        }
    }
}

public class CitationItem : MonoBehaviour
{
    [SerializeField] private TMP_Text fileNameText;
    [SerializeField] private TMP_Text quoteText;
    [SerializeField] private Button viewButton;
    
    private Citation citation;
    
    public void SetCitation(Citation citation)
    {
        this.citation = citation;
        
        fileNameText.text = citation.FileName;
        quoteText.text = citation.Quote;
        
        viewButton.onClick.AddListener(OnViewClicked);
    }
    
    void OnViewClicked()
    {
        // Open file at specific page/location
        FileViewer.OpenFile(citation.FileName, citation.PageNumber);
    }
}
```

## File Management

### List Files

```csharp
var files = await agent.ListFilesAsync();

foreach (var file in files)
{
    Debug.Log($"File: {file.FileName}");
    Debug.Log($"  Size: {file.Bytes} bytes");
    Debug.Log($"  Status: {file.Status}");
}
```

### Delete Files

```csharp
// Delete single file
await agent.DeleteFileAsync(fileId);

// Delete multiple files
await agent.DeleteFilesAsync(fileIds);
```

### File Status

```csharp
var status = await agent.GetFileStatusAsync(fileId);

Debug.Log($"Status: {status.Status}"); // processing, completed, failed
Debug.Log($"Progress: {status.Progress}%");

if (status.Status == "failed")
{
    Debug.LogError($"Error: {status.Error}");
}
```

## Advanced Usage

### Search Configuration

```csharp
agent.Settings.FileSearch = new FileSearchSettings
{
    MaxResults = 10,
    MinRelevanceScore = 0.7f,
    IncludeCitations = true,
    ChunkSize = 1024,
    ChunkOverlap = 128
};
```

### Custom Chunking

```csharp
// Upload with custom chunking strategy
await agent.UploadFileAsync(
    filePath: "large_document.pdf",
    chunkingStrategy: new ChunkingStrategy
    {
        Type = "static",
        MaxChunkSizeTokens = 800,
        ChunkOverlapTokens = 100
    }
);
```

### Metadata Filtering

```csharp
// Upload with metadata
await agent.UploadFileAsync(
    filePath: "guide.pdf",
    metadata: new Dictionary<string, string>
    {
        { "category", "tutorial" },
        { "version", "1.0" },
        { "language", "en" }
    }
);

// Search with metadata filter
var results = await agent.SearchFilesAsync(
    query: "getting started",
    filter: new { category = "tutorial" }
);
```

## Use Cases

### Documentation Q&A

```csharp
public class DocumentationAssistant : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    async void Start()
    {
        // Upload documentation files
        var files = Directory.GetFiles("Documentation", "*.md");
        var fileIds = await agent.UploadFilesAsync(files);
        
        await agent.CreateVectorStoreAsync("Game Docs", fileIds);
        agent.AddTool(ToolType.FileSearch);
        
        Debug.Log("📚 Documentation assistant ready");
    }
    
    public async void AskQuestion(string question)
    {
        await agent.SendAsync(question);
        // Agent will search docs and provide answer with citations
    }
}
```

### Code Reference

```csharp
async void SetupCodeReference()
{
    // Upload code documentation
    var codeFiles = new[]
    {
        "API_Reference.md",
        "Code_Examples.txt",
        "Best_Practices.pdf"
    };
    
    var fileIds = await agent.UploadFilesAsync(codeFiles);
    await agent.CreateVectorStoreAsync("Code Reference", fileIds);
    
    agent.AddTool(ToolType.FileSearch);
}

// Ask code-related questions
await agent.SendAsync("How do I implement player movement?");
await agent.SendAsync("Show me examples of inventory systems");
```

### Game Data Analysis

```csharp
async void AnalyzeGameData()
{
    // Upload game data files
    var dataFiles = new[]
    {
        "player_stats.json",
        "item_database.csv",
        "level_data.txt"
    };
    
    var fileIds = await agent.UploadFilesAsync(dataFiles);
    await agent.CreateVectorStoreAsync("Game Data", fileIds);
    
    agent.AddTool(ToolType.FileSearch);
    
    // Query game data
    await agent.SendAsync("What are the most powerful weapons?");
    await agent.SendAsync("Which levels have the highest difficulty?");
}
```

## Performance Optimization

### Batch Upload

```csharp
public async UniTask BatchUploadFiles(string[] filePaths, int batchSize = 5)
{
    var allFileIds = new List<string>();
    
    for (int i = 0; i < filePaths.Length; i += batchSize)
    {
        var batch = filePaths.Skip(i).Take(batchSize).ToArray();
        
        Debug.Log($"Uploading batch {i / batchSize + 1}...");
        var fileIds = await agent.UploadFilesAsync(batch);
        allFileIds.AddRange(fileIds);
        
        await UniTask.Delay(1000); // Rate limiting
    }
    
    return allFileIds.ToArray();
}
```

### Cache Vector Stores

```csharp
public class VectorStoreCache : MonoBehaviour
{
    private Dictionary<string, string> storeCache = new();
    
    public async UniTask<string> GetOrCreateStore(string name, string[] fileIds)
    {
        // Check cache
        if (storeCache.TryGetValue(name, out string storeId))
        {
            return storeId;
        }
        
        // Create new store
        var store = await agent.CreateVectorStoreAsync(name, fileIds);
        storeCache[name] = store.Id;
        
        // Save to PlayerPrefs
        PlayerPrefs.SetString($"VectorStore_{name}", store.Id);
        
        return store.Id;
    }
}
```

## Error Handling

### Upload Errors

```csharp
async UniTask<string> SafeUploadFile(string filePath)
{
    try
    {
        // Check file exists
        if (!File.Exists(filePath))
        {
            throw new FileNotFoundException($"File not found: {filePath}");
        }
        
        // Check file size (max 512MB)
        var fileInfo = new FileInfo(filePath);
        if (fileInfo.Length > 512 * 1024 * 1024)
        {
            throw new Exception("File too large (max 512MB)");
        }
        
        // Check format
        if (!IsSupported(filePath))
        {
            throw new Exception($"Unsupported file format: {Path.GetExtension(filePath)}");
        }
        
        // Upload
        return await agent.UploadFileAsync(filePath);
    }
    catch (Exception ex)
    {
        Debug.LogError($"Upload failed: {ex.Message}");
        return null;
    }
}
```

### Search Errors

```csharp
agent.onToolCallError.AddListener((toolCall, error) =>
{
    if (toolCall.Function.Name == "file_search")
    {
        Debug.LogError($"File search failed: {error}");
        
        // Show user message
        ShowMessage("Search temporarily unavailable. Please try again.");
    }
});
```

## Best Practices

### 1. Organize Files

```csharp
// Create separate vector stores by category
await CreateVectorStore("Tutorials", tutorialFiles);
await CreateVectorStore("API Reference", apiFiles);
await CreateVectorStore("Game Data", dataFiles);
```

### 2. Update Regularly

```csharp
async void UpdateDocumentation()
{
    // Remove old files
    var oldFiles = await agent.ListFilesAsync();
    await agent.DeleteFilesAsync(oldFiles.Select(f => f.Id).ToArray());
    
    // Upload new versions
    var newFiles = Directory.GetFiles("Documentation");
    var fileIds = await agent.UploadFilesAsync(newFiles);
    
    // Update vector store
    await agent.CreateVectorStoreAsync("Docs", fileIds);
    
    Debug.Log("✓ Documentation updated");
}
```

### 3. Monitor Usage

```csharp
void TrackFileSearchUsage()
{
    agent.onToolCallCompleted.AddListener((toolCall, output) =>
    {
        if (toolCall.Function.Name == "file_search")
        {
            int searchCount = PlayerPrefs.GetInt("FileSearchCount", 0);
            PlayerPrefs.SetInt("FileSearchCount", searchCount + 1);
            
            Debug.Log($"File searches: {searchCount + 1}");
        }
    });
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class FileSearchManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private string documentationPath = "Documentation";
    
    private string vectorStoreId;
    
    async void Start()
    {
        await SetupFileSearch();
    }
    
    async UniTask SetupFileSearch()
    {
        Debug.Log("📚 Setting up file search...");
        
        try
        {
            // Get all documentation files
            var files = GetDocumentationFiles();
            Debug.Log($"Found {files.Length} files");
            
            // Upload files
            var fileIds = await UploadFiles(files);
            Debug.Log($"Uploaded {fileIds.Length} files");
            
            // Create vector store
            var store = await agent.CreateVectorStoreAsync(
                "Game Documentation",
                fileIds
            );
            vectorStoreId = store.Id;
            
            // Enable file search
            agent.AddTool(ToolType.FileSearch);
            
            // Listen for citations
            agent.onResponseCompleted.AddListener(DisplayCitations);
            
            Debug.Log("✓ File search ready");
        }
        catch (Exception ex)
        {
            Debug.LogError($"Setup failed: {ex.Message}");
        }
    }
    
    string[] GetDocumentationFiles()
    {
        if (!Directory.Exists(documentationPath))
        {
            Debug.LogWarning($"Directory not found: {documentationPath}");
            return new string[0];
        }
        
        return Directory.GetFiles(documentationPath, "*.*", SearchOption.AllDirectories)
            .Where(f => IsSupported(f))
            .ToArray();
    }
    
    bool IsSupported(string filePath)
    {
        string ext = Path.GetExtension(filePath).ToLower();
        string[] supported = { ".pdf", ".txt", ".md", ".json", ".csv" };
        return supported.Contains(ext);
    }
    
    async UniTask<string[]> UploadFiles(string[] filePaths)
    {
        var fileIds = new List<string>();
        
        foreach (var filePath in filePaths)
        {
            try
            {
                string fileId = await agent.UploadFileAsync(filePath);
                fileIds.Add(fileId);
                Debug.Log($"✓ Uploaded: {Path.GetFileName(filePath)}");
            }
            catch (Exception ex)
            {
                Debug.LogError($"Failed to upload {filePath}: {ex.Message}");
            }
        }
        
        return fileIds.ToArray();
    }
    
    void DisplayCitations(Response response)
    {
        if (response.Citations != null && response.Citations.Count > 0)
        {
            Debug.Log($"📖 {response.Citations.Count} citation(s):");
            
            foreach (var citation in response.Citations)
            {
                Debug.Log($"  • {citation.FileName}");
                Debug.Log($"    \"{citation.Quote}\"");
            }
        }
    }
    
    public async void AskQuestion(string question)
    {
        Debug.Log($"❓ Question: {question}");
        await agent.SendAsync(question);
    }
    
    public async void UpdateDocumentation()
    {
        Debug.Log("🔄 Updating documentation...");
        
        // Delete old vector store
        if (!string.IsNullOrEmpty(vectorStoreId))
        {
            await agent.DeleteVectorStoreAsync(vectorStoreId);
        }
        
        // Re-setup
        await SetupFileSearch();
    }
}
```

## Next Steps

- [Code Interpreter](code-interpreter.md)
- [Web Search](web-search.md)
- [MCP](mcp.md)
