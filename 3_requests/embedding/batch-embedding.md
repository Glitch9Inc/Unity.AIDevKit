---
icon: layer-group
---

# Batch Embedding

Generate embeddings for multiple texts efficiently using `.GENEmbed()`.

## Basic Usage

```csharp
string[] texts = new[]
{
    "First document",
    "Second document",
    "Third document"
};

float[][] embeddings = await texts
    .GENEmbed()
    .ExecuteAsync();
```

## Configuration

```csharp
string[] texts = new[] { "Text 1", "Text 2", "Text 3" };

float[][] embeddings = await texts
    .GENEmbed()
    .SetModel(OpenAIModel.TextEmbedding3Small)
    .ExecuteAsync();
```

## Unity Integration Examples

### Example 1: Bulk Document Indexing

```csharp
public class BulkIndexer : MonoBehaviour
{
    public async UniTask IndexDocuments(List<string> documents)
    {
        // Generate embeddings in batches of 100
        int batchSize = 100;
        var allEmbeddings = new List<float[]>();
        
        for (int i = 0; i < documents.Count; i += batchSize)
        {
            var batch = documents
                .Skip(i)
                .Take(batchSize)
                .ToArray();
            
            float[][] embeddings = await batch
                .GENEmbed()
                .SetModel(OpenAIModel.TextEmbedding3Small)
                .ExecuteAsync();
            
            allEmbeddings.AddRange(embeddings);
            
            Debug.Log($"Indexed {i + batch.Length}/{documents.Count} documents");
        }
        
        SaveEmbeddings(allEmbeddings);
    }
}
```

### Example 2: FAQ Database Builder

```csharp
public class FAQDatabaseBuilder : MonoBehaviour
{
    [System.Serializable]
    public class FAQ
    {
        public string question;
        public string answer;
        public float[] embedding;
    }
    
    public async UniTask BuildFAQDatabase(List<(string q, string a)> faqData)
    {
        // Extract all questions
        string[] questions = faqData.Select(x => x.q).ToArray();
        
        // Generate embeddings for all questions at once
        float[][] embeddings = await questions
            .GENEmbed()
            .ExecuteAsync();
        
        // Build FAQ database
        var faqs = new List<FAQ>();
        for (int i = 0; i < faqData.Count; i++)
        {
            faqs.Add(new FAQ
            {
                question = faqData[i].q,
                answer = faqData[i].a,
                embedding = embeddings[i]
            });
        }
        
        SaveFAQs(faqs);
    }
}
```

### Example 3: Content Library Processor

```csharp
public class ContentLibraryProcessor : MonoBehaviour
{
    public async UniTask ProcessLibrary(List<string> contentItems)
    {
        Debug.Log($"Processing {contentItems.Count} items...");
        
        // Generate embeddings for entire library
        float[][] embeddings = await contentItems
            .ToArray()
            .GENEmbed()
            .ExecuteAsync();
        
        // Store with metadata
        for (int i = 0; i < contentItems.Count; i++)
        {
            SaveContentWithEmbedding(contentItems[i], embeddings[i]);
        }
        
        Debug.Log("Library processing complete");
    }
}
```

### Example 4: Multi-Language Indexer

```csharp
public class MultiLanguageIndexer : MonoBehaviour
{
    public async UniTask IndexMultiLanguageContent(Dictionary<string, List<string>> contentByLanguage)
    {
        foreach (var (language, contents) in contentByLanguage)
        {
            Debug.Log($"Indexing {contents.Count} {language} documents...");
            
            float[][] embeddings = await contents
                .ToArray()
                .GENEmbed()
                .ExecuteAsync();
            
            SaveLanguageIndex(language, contents, embeddings);
        }
    }
}
```

## Performance Benefits

### Sequential vs Batch

```csharp
// ❌ Slow - Sequential
async UniTask<List<float[]>> SequentialEmbedding(List<string> texts)
{
    var embeddings = new List<float[]>();
    
    foreach (var text in texts)
    {
        float[] embed = await text.GENEmbed().ExecuteAsync();
        embeddings.Add(embed);
    }
    
    return embeddings;  // Takes N * API_CALL_TIME
}

// ✅ Fast - Batch
async UniTask<float[][]> BatchEmbedding(List<string> texts)
{
    return await texts
        .ToArray()
        .GENEmbed()
        .ExecuteAsync();  // Takes ~1 * API_CALL_TIME
}
```

## Batch Size Recommendations

Different providers have different limits:

```csharp
public class BatchEmbedder
{
    private const int OPENAI_MAX_BATCH = 2048;
    private const int GOOGLE_MAX_BATCH = 100;
    
    public async UniTask<float[][]> EmbedLargeDataset(
        string[] texts, 
        int batchSize = 100)
    {
        var allEmbeddings = new List<float[]>();
        
        for (int i = 0; i < texts.Length; i += batchSize)
        {
            var batch = texts
                .Skip(i)
                .Take(batchSize)
                .ToArray();
            
            float[][] embeddings = await batch
                .GENEmbed()
                .ExecuteAsync();
            
            allEmbeddings.AddRange(embeddings);
        }
        
        return allEmbeddings.ToArray();
    }
}
```

## Best Practices

### ✅ Good Practices

```csharp
// ✅ Use batching for multiple texts
string[] texts = GetTexts();
float[][] embeddings = await texts.GENEmbed().ExecuteAsync();

// ✅ Process in reasonable batch sizes
const int BATCH_SIZE = 100;
var batches = texts.Chunk(BATCH_SIZE);

// ✅ Show progress for large batches
for (int i = 0; i < batches.Count; i++)
{
    await batches[i].GENEmbed().ExecuteAsync();
    Debug.Log($"Progress: {i+1}/{batches.Count}");
}

// ✅ Handle errors per batch
try
{
    await batch.GENEmbed().ExecuteAsync();
}
catch (Exception ex)
{
    Debug.LogError($"Batch {i} failed: {ex.Message}");
}
```

### ❌ Bad Practices

```csharp
// ❌ Don't use sequential when batch is available
foreach (var text in texts)
{
    await text.GENEmbed().ExecuteAsync();  // Slow!
}

// ❌ Don't use excessively large batches
string[] hugeBatch = new string[10000];  // Too large!
await hugeBatch.GENEmbed().ExecuteAsync();

// ❌ Don't ignore rate limits
// Process batches too quickly without delay
```

## Error Handling

```csharp
public async UniTask<float[][]> SafeBatchEmbedding(string[] texts)
{
    const int BATCH_SIZE = 100;
    var allEmbeddings = new List<float[]>();
    
    for (int i = 0; i < texts.Length; i += BATCH_SIZE)
    {
        try
        {
            var batch = texts
                .Skip(i)
                .Take(BATCH_SIZE)
                .ToArray();
            
            float[][] embeddings = await batch
                .GENEmbed()
                .ExecuteAsync();
            
            allEmbeddings.AddRange(embeddings);
        }
        catch (Exception ex)
        {
            Debug.LogError($"Batch {i} failed: {ex.Message}");
            
            // Add empty embeddings for failed batch
            for (int j = 0; j < BATCH_SIZE && i + j < texts.Length; j++)
            {
                allEmbeddings.Add(null);
            }
        }
    }
    
    return allEmbeddings.ToArray();
}
```

## Provider Limits

| Provider | Max Batch Size | Notes |
|----------|----------------|-------|
| **OpenAI** | 2,048 texts | Per request |
| **Google** | 100 texts | Per request |

## Performance Comparison

**Example: 1000 documents**

| Method | Time | API Calls |
|--------|------|-----------|
| Sequential | ~1000s | 1000 |
| Batch (100) | ~10s | 10 |
| Batch (500) | ~2s | 2 |

## Use Cases

| Use Case | Batch Size |
|----------|-----------|
| **Small FAQ** | All at once (10-50) |
| **Medium Library** | 100-200 per batch |
| **Large Dataset** | 500-1000 per batch |
| **Real-time** | Process as needed |

## Next Steps

- [Text Embedding](text-embedding.md) - Single text embeddings
