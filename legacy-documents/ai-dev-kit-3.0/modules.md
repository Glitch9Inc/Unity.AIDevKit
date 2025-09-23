---
description: >-
  Optional modules that let your AI listen, speak, generate images, or execute
  functions.
icon: codepen
---

# Modules

These modules act as optional extensions that can be plugged into AI components like [`Chatbot`](../../unity-components/ai-agents/chatbot-chat-completion-api.md),  [`Chatbot (Assistants API)`](../../unity-components/ai-agents/assistant-agent-assistants-api.md), and other higher-level interfaces.

They allow your AI to listen, speak, generate images, or execute user-defined functions in response to AI requests.

***

#### 1. Speech to Text

Enables voice input functionality using microphone capture.

**What it does**

* Records audio using the microphone.
* Converts speech to text asynchronously.
* Returns a [`Transcript`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.Transcript.html) object.

**Typical usage**\
Call `StartRecording()`, speak, then call `StopRecording()` to get the [`Transcript`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.Transcript.html).

```csharp
stt.StartRecording();
// user speaks
Transcript transcript = await stt.StopRecording();
```

<figure><img src="../../.gitbook/assets/image (3).png" alt="" width="563"><figcaption></figcaption></figure>

***

#### Text to Speech

Provides voice output via TTS generation.

**What it does**

* Converts a string response into [`GeneratedAudio`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.GeneratedAudio.html) (`AudioClip`).
* Can play automatically if `AudioSource` is attached.

**Use with**\
[`Chatbot`](../../unity-components/ai-agents/chatbot-chat-completion-api.md) or [`Chatbot (Assistants API)`](../../unity-components/ai-agents/assistant-agent-assistants-api.md) to read responses aloud.

```csharp
GeneratedAudio audio = await tts.GenerateSpeechAsync("Hello!");
audioSource.clip = audio;
audioSource.Play();
```

<figure><img src="../../.gitbook/assets/image (4).png" alt="" width="563"><figcaption></figcaption></figure>

***

#### Voice Changer

Applies optional pitch and speed post-processing to audio output.

**What it does**

* Alters pitch and speed for stylistic or character-based transformation.
* Simulates different speaker characteristics.

**Optional**\
Enhances immersion but is not required.

<figure><img src="../../.gitbook/assets/image (9).png" alt="" width="563"><figcaption></figcaption></figure>

***

#### Image Generator

Generates images from natural language prompts.

**What it does**

* Supports models like DALL·E (OpenAI) and Imagen (Google).
* Customizable: resolution, style, aspect ratio, quality.

**Output**\
Returns a [`GeneratedImage`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.GeneratedImage.html) containing a `Texture2D`.

```csharp
var image = await imageGenerator.GenerateImageAsync("A futuristic city at night");
```

<figure><img src="../../.gitbook/assets/image (5).png" alt="" width="563"><figcaption></figcaption></figure>

***

#### Function Manager

Registers C# methods as callable functions through Function Calling.

**What it does**

* Makes Unity methods callable by the AI.
* Auto-generates parameter schema and metadata.

**How to Use**

1. Add [`FunctionManager`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.Components.FunctionManager.html) to a GameObject.
2. Assign a script with public void methods.
3. Select a method in the Inspector.
4. Optionally describe parameters for the AI.

<figure><img src="../../.gitbook/assets/image (8).png" alt="" width="563"><figcaption></figcaption></figure>

```csharp
public class GameFunctions : MonoBehaviour
{
    public void HealPlayer(int amount, string reason)
    {
        Debug.Log($"Healing player by {amount} because {reason}");
    }
}
```

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

