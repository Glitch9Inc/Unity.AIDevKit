# Text Delta Events

Handle streaming text updates in real-time.

## Overview

Text Delta Events enable:

- Real-time text streaming
- Character-by-character updates
- Typewriter effects
- Progress tracking
- Partial content handling

## Basic Setup

### Subscribe to Text Delta

```csharp
public class TextDeltaListener : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text responseText;
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onTextCompleted.AddListener(OnTextCompleted);
    }
    
    void OnTextDelta(string delta)
    {
        responseText.text += delta;
    }
    
    void OnTextCompleted(string fullText)
    {
        Debug.Log($"✓ Text completed: {fullText.Length} characters");
    }
}
```

## Streaming Display

### Typewriter Effect

```csharp
public class TypewriterEffect : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text textDisplay;
    [SerializeField] private float charactersPerSecond = 50f;
    
    private Queue<char> characterQueue = new();
    private bool isTyping;
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onTextCompleted.AddListener(OnTextCompleted);
    }
    
    void OnTextDelta(string delta)
    {
        // Queue characters
        foreach (char c in delta)
        {
            characterQueue.Enqueue(c);
        }
        
        // Start typing if not already
        if (!isTyping)
        {
            StartTyping().Forget();
        }
    }
    
    async UniTaskVoid StartTyping()
    {
        isTyping = true;
        
        float delay = 1f / charactersPerSecond;
        
        while (characterQueue.Count > 0)
        {
            char c = characterQueue.Dequeue();
            textDisplay.text += c;
            
            await UniTask.Delay(TimeSpan.FromSeconds(delay));
        }
        
        isTyping = false;
    }
    
    void OnTextCompleted(string fullText)
    {
        // Ensure all text is displayed
        if (characterQueue.Count > 0)
        {
            string remaining = string.Join("", characterQueue);
            textDisplay.text += remaining;
            characterQueue.Clear();
        }
        
        isTyping = false;
        
        Debug.Log("✓ Typewriter completed");
    }
}
```

### Smooth Streaming

```csharp
public class SmoothStreaming : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text textDisplay;
    
    private StringBuilder currentText = new();
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onStreamingStarted.AddListener(OnStreamingStarted);
        agent.onStreamingCompleted.AddListener(OnStreamingCompleted);
    }
    
    void OnStreamingStarted()
    {
        currentText.Clear();
        textDisplay.text = "";
        
        Debug.Log("📡 Streaming started");
    }
    
    void OnTextDelta(string delta)
    {
        currentText.Append(delta);
        textDisplay.text = currentText.ToString();
    }
    
    void OnStreamingCompleted()
    {
        Debug.Log($"✓ Streaming completed: {currentText.Length} characters");
    }
}
```

## Progress Tracking

### Track Streaming Progress

```csharp
public class StreamingProgress : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private Slider progressBar;
    [SerializeField] private TMP_Text progressText;
    
    private int estimatedLength;
    private int currentLength;
    
    void Start()
    {
        agent.onStreamingStarted.AddListener(OnStreamingStarted);
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onStreamingCompleted.AddListener(OnStreamingCompleted);
    }
    
    void OnStreamingStarted()
    {
        // Estimate based on previous responses
        estimatedLength = 500;
        currentLength = 0;
        
        progressBar.value = 0;
        progressText.text = "0%";
    }
    
    void OnTextDelta(string delta)
    {
        currentLength += delta.Length;
        
        float progress = Mathf.Min((float)currentLength / estimatedLength, 1f);
        progressBar.value = progress;
        progressText.text = $"{progress:P0}";
    }
    
    void OnStreamingCompleted()
    {
        progressBar.value = 1f;
        progressText.text = "100%";
        
        // Update estimate for next time
        estimatedLength = currentLength;
    }
}
```

## Word-by-Word Processing

### Process Words as They Arrive

```csharp
public class WordProcessor : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    
    private StringBuilder wordBuffer = new();
    private List<string> processedWords = new();
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onTextCompleted.AddListener(OnTextCompleted);
    }
    
    void OnTextDelta(string delta)
    {
        foreach (char c in delta)
        {
            if (char.IsWhiteSpace(c))
            {
                // Complete word
                if (wordBuffer.Length > 0)
                {
                    string word = wordBuffer.ToString();
                    ProcessWord(word);
                    wordBuffer.Clear();
                }
            }
            else
            {
                wordBuffer.Append(c);
            }
        }
    }
    
    void ProcessWord(string word)
    {
        processedWords.Add(word);
        
        Debug.Log($"Word #{processedWords.Count}: {word}");
        
        // Analyze word
        AnalyzeWord(word);
    }
    
    void AnalyzeWord(string word)
    {
        // Keyword detection
        if (IsImportantKeyword(word))
        {
            HighlightKeyword(word);
        }
        
        // Language detection
        // Sentiment analysis
        // etc.
    }
    
    bool IsImportantKeyword(string word)
    {
        string[] keywords = { "error", "warning", "success", "important" };
        return keywords.Contains(word.ToLower());
    }
    
    void HighlightKeyword(string word)
    {
        Debug.Log($"⭐ Important keyword detected: {word}");
    }
    
    void OnTextCompleted(string fullText)
    {
        // Process remaining buffer
        if (wordBuffer.Length > 0)
        {
            ProcessWord(wordBuffer.ToString());
            wordBuffer.Clear();
        }
        
        Debug.Log($"✓ Processed {processedWords.Count} words");
    }
}
```

## Markdown Rendering

### Stream Markdown Content

```csharp
public class MarkdownStreaming : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text textDisplay;
    
    private StringBuilder markdownBuffer = new();
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onTextCompleted.AddListener(OnTextCompleted);
    }
    
    void OnTextDelta(string delta)
    {
        markdownBuffer.Append(delta);
        
        // Render markdown
        string rendered = RenderMarkdown(markdownBuffer.ToString());
        textDisplay.text = rendered;
    }
    
    void OnTextCompleted(string fullText)
    {
        // Final render
        string rendered = RenderMarkdown(fullText);
        textDisplay.text = rendered;
    }
    
    string RenderMarkdown(string markdown)
    {
        // Simple markdown to TextMeshPro formatting
        string result = markdown;
        
        // Bold
        result = System.Text.RegularExpressions.Regex.Replace(
            result, @"\*\*(.+?)\*\*", "<b>$1</b>");
        
        // Italic
        result = System.Text.RegularExpressions.Regex.Replace(
            result, @"\*(.+?)\*", "<i>$1</i>");
        
        // Code
        result = System.Text.RegularExpressions.Regex.Replace(
            result, @"`(.+?)`", "<color=#00ff00>$1</color>");
        
        return result;
    }
}
```

## Code Block Detection

### Detect and Format Code

```csharp
public class CodeBlockDetector : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text textDisplay;
    [SerializeField] private TMP_Text codeDisplay;
    
    private StringBuilder textBuffer = new();
    private StringBuilder codeBuffer = new();
    private bool inCodeBlock;
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
    }
    
    void OnTextDelta(string delta)
    {
        foreach (char c in delta)
        {
            if (CheckForCodeBlockToggle(c))
            {
                inCodeBlock = !inCodeBlock;
                continue;
            }
            
            if (inCodeBlock)
            {
                codeBuffer.Append(c);
                UpdateCodeDisplay();
            }
            else
            {
                textBuffer.Append(c);
                UpdateTextDisplay();
            }
        }
    }
    
    bool CheckForCodeBlockToggle(char c)
    {
        // Simplified: check for ``` pattern
        // In production, use proper state machine
        return c == '`';
    }
    
    void UpdateTextDisplay()
    {
        textDisplay.text = textBuffer.ToString();
    }
    
    void UpdateCodeDisplay()
    {
        codeDisplay.text = codeBuffer.ToString();
    }
}
```

## Buffering Strategies

### Debounced Updates

```csharp
public class DebouncedStreaming : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text textDisplay;
    [SerializeField] private float debounceDelay = 0.1f;
    
    private StringBuilder buffer = new();
    private bool updatePending;
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
    }
    
    void OnTextDelta(string delta)
    {
        buffer.Append(delta);
        
        if (!updatePending)
        {
            ScheduleUpdate().Forget();
        }
    }
    
    async UniTaskVoid ScheduleUpdate()
    {
        updatePending = true;
        
        await UniTask.Delay(TimeSpan.FromSeconds(debounceDelay));
        
        textDisplay.text = buffer.ToString();
        updatePending = false;
    }
}
```

### Batched Updates

```csharp
public class BatchedStreaming : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text textDisplay;
    [SerializeField] private int batchSize = 10;
    
    private StringBuilder buffer = new();
    private StringBuilder displayBuffer = new();
    
    void Start()
    {
        agent.onTextDelta.AddListener(OnTextDelta);
    }
    
    void OnTextDelta(string delta)
    {
        buffer.Append(delta);
        
        // Update display when batch size reached
        if (buffer.Length >= batchSize)
        {
            FlushBuffer();
        }
    }
    
    void FlushBuffer()
    {
        displayBuffer.Append(buffer.ToString());
        textDisplay.text = displayBuffer.ToString();
        buffer.Clear();
    }
}
```

## Animation Effects

### Fade In Text

```csharp
public class FadeInText : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text textDisplay;
    [SerializeField] private float fadeSpeed = 2f;
    
    void Start()
    {
        agent.onStreamingStarted.AddListener(OnStreamingStarted);
        agent.onTextDelta.AddListener(OnTextDelta);
    }
    
    void OnStreamingStarted()
    {
        textDisplay.alpha = 0;
        FadeIn().Forget();
    }
    
    async UniTaskVoid FadeIn()
    {
        while (textDisplay.alpha < 1f)
        {
            textDisplay.alpha += fadeSpeed * Time.deltaTime;
            await UniTask.Yield();
        }
        
        textDisplay.alpha = 1f;
    }
    
    void OnTextDelta(string delta)
    {
        textDisplay.text += delta;
    }
}
```

## Error Handling

### Handle Streaming Errors

```csharp
void Start()
{
    agent.onStreamingError.AddListener(OnStreamingError);
}

void OnStreamingError(string error)
{
    Debug.LogError($"Streaming error: {error}");
    
    // Show partial content
    textDisplay.text += "\n\n<color=red>[Streaming interrupted]</color>";
    
    // Offer retry
    ShowRetryButton(true);
}
```

## Complete Example

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;
using Cysharp.Threading.Tasks;

public class StreamingTextManager : MonoBehaviour
{
    [SerializeField] private AgentBehaviour agent;
    [SerializeField] private TMP_Text responseText;
    [SerializeField] private Slider progressBar;
    [SerializeField] private TMP_Text statusText;
    
    [Header("Settings")]
    [SerializeField] private float typewriterSpeed = 50f;
    [SerializeField] private bool enableTypewriter = true;
    
    private StringBuilder fullText = new();
    private Queue<char> characterQueue = new();
    private bool isTyping;
    private int estimatedLength = 500;
    
    void Start()
    {
        RegisterEvents();
    }
    
    void RegisterEvents()
    {
        agent.onStreamingStarted.AddListener(OnStreamingStarted);
        agent.onTextDelta.AddListener(OnTextDelta);
        agent.onStreamingCompleted.AddListener(OnStreamingCompleted);
        agent.onStreamingError.AddListener(OnStreamingError);
    }
    
    void OnStreamingStarted()
    {
        fullText.Clear();
        characterQueue.Clear();
        responseText.text = "";
        progressBar.value = 0;
        statusText.text = "Streaming...";
        
        Debug.Log("📡 Streaming started");
    }
    
    void OnTextDelta(string delta)
    {
        fullText.Append(delta);
        
        if (enableTypewriter)
        {
            // Queue for typewriter
            foreach (char c in delta)
            {
                characterQueue.Enqueue(c);
            }
            
            if (!isTyping)
            {
                StartTypewriter().Forget();
            }
        }
        else
        {
            // Direct update
            responseText.text = fullText.ToString();
        }
        
        // Update progress
        float progress = Mathf.Min((float)fullText.Length / estimatedLength, 1f);
        progressBar.value = progress;
    }
    
    async UniTaskVoid StartTypewriter()
    {
        isTyping = true;
        float delay = 1f / typewriterSpeed;
        
        while (characterQueue.Count > 0)
        {
            char c = characterQueue.Dequeue();
            responseText.text += c;
            
            await UniTask.Delay(TimeSpan.FromSeconds(delay));
        }
        
        isTyping = false;
    }
    
    void OnStreamingCompleted()
    {
        // Flush remaining characters
        if (enableTypewriter && characterQueue.Count > 0)
        {
            string remaining = string.Join("", characterQueue);
            responseText.text += remaining;
            characterQueue.Clear();
        }
        
        isTyping = false;
        progressBar.value = 1f;
        statusText.text = "Complete";
        
        // Update estimate
        estimatedLength = fullText.Length;
        
        Debug.Log($"✓ Streaming completed: {fullText.Length} characters");
    }
    
    void OnStreamingError(string error)
    {
        Debug.LogError($"Streaming error: {error}");
        
        statusText.text = "<color=red>Streaming interrupted</color>";
        responseText.text += "\n\n<color=red>[Stream interrupted]</color>";
        
        isTyping = false;
        characterQueue.Clear();
    }
    
    void OnDestroy()
    {
        if (agent != null)
        {
            agent.onStreamingStarted.RemoveListener(OnStreamingStarted);
            agent.onTextDelta.RemoveListener(OnTextDelta);
            agent.onStreamingCompleted.RemoveListener(OnStreamingCompleted);
            agent.onStreamingError.RemoveListener(OnStreamingError);
        }
    }
}
```

## Next Steps

- [Agent Lifecycle Hooks](agent-hooks.md)
- [Tool Call Events](tool-calls.md)
- [Audio Events](audio.md)
- [Status Events](status.md)
- [Error Handling Events](error-handling.md)
