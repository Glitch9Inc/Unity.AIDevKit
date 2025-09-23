---
icon: book-font
---

# Prefixes

To get the most out of IntelliSense when using the AI Dev Kit, the following prefixes are designed to make autocomplete clean, fast, and intuitive.&#x20;

Try to remember and follow these naming conventions:

***

#### `GEN` — Prefix for All Task Creators

These are extension methods that can be called directly on known objects to start an AI generation task.

<table><thead><tr><th width="200.00018310546875">Prompt Type</th><th>Usage</th></tr></thead><tbody><tr><td><mark style="color:blue;">string</mark></td><td>Text/Content Generation<br>Image Generation<br>Speech Generation (TTS)<br>Sound FX Generation<br>Video Generation</td></tr><tr><td><mark style="color:green;">AudioClip</mark></td><td>Transcript Generation (STT)<br>Voice Change<br>Audio Isolation</td></tr><tr><td><mark style="color:green;">Texture2D, Sprite</mark></td><td>Text/Content Generation (Vision)<br>Image Edit (Inpaint)<br>Image Variation</td></tr><tr><td><mark style="color:purple;"><strong>ChatSession</strong></mark></td><td>Chat</td></tr></tbody></table>

<pre class="language-csharp"><code class="lang-csharp"><strong>string aiJoke = await "Tell me a joke."
</strong>    .GENText()
    .SetModel(OpenAIModel.GPT4o)
    .ExecuteAsync();

string transcript = await audioClip
    .GENTranscript()
    .ExecuteAsync();
    
Texture2D aiImage = await "A cyberpunk city at night"
    .GENImage()
    .ExecuteAsync();
</code></pre>

***

#### `Set` — Prefix for All Configuration Options

Use this to configure task parameters such as model, output count, output path, and advanced options like reasoning, web search, or voice settings.

```csharp
task
    .SetModel(OpenAIModel.GPT4o)
    .SetCount(3)
    .SetReasoningEffort(ReasoningEffort.High);
```

***

#### `On` — Prefix for All Streaming Callbacks

Use these methods to attach real-time callbacks for streamed results, such as receiving text tokens, tool calls, or error handling.

```csharp
task
    .OnStreamText(Debug.Log)
    .OnStreamError(HandleError)
    .OnStreamDone(OnGenerationComplete);
```

***

#### `Append` — Prefix for Task Sequences

Used with [`GENSequence`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.GENSequence.html) to compose a sequence of tasks or insert delays between steps.

```csharp
new GENSequence()
    .AppendText("Generate story".GENText())
    .AppendInterval(2.0f)
    .AppendTextToImage(text => text.GENImage())
    .ExecuteAsync();
```

***

These prefixes follow a consistent pattern to maximize IntelliSense usability and make task flows easy to read and construct.

{% hint style="success" %}
**Tip:** Just type `GEN`, `Set`, or `On` and let IntelliSense show you what’s possible.
{% endhint %}
