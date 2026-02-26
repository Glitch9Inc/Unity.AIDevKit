---
icon: cube
---

# AI Dev Kit 4.7.0

<figure><img src=".gitbook/assets/Cover Image.png" alt=""><figcaption></figcaption></figure>

## The Ultimate AI Suite for Unity

**AI Dev Kit** empowers beginner developers to effortlessly integrate advanced AI functionalities directly into Unity, dramatically simplifying your game development workflow.

With just a few clicks and simple text prompts, you can create code, generate images, produce sound effects, and even synthesize voices. **AI Dev Kit** offers broad API integrations, rich editor tools, extensive voice synthesis options, and unique audio generation capabilities.

***

### From Prompt to Output in One Line

AI Dev Kit gives you instant access to text, image, audio, and code generation — with zero boilerplate.

```csharp
// Write AI-powered NPC backstories
string response = await "Describe a stoic robot farmer on Mars."
    .GENResponse()
    .ExecuteAsync();

// Generate stylized images instantly
Texture2D image = "A rusty sci-fi door with bullet holes"
    .GENImage()
    .ExecuteAsync();

// Create voiceover from text
AudioClip voice = "Welcome back, Commander."
    .GENSpeech()
    .SetVoice(ElevenLabsVoice.Rachel)
    .ExecuteAsync();
```

Just call `.GEN*()` on your Unity objects and chain your desired behavior — it's fast, readable, and production-ready.

Works out of the box. Fully extensible. No extra SDKs required.

***

### Provider Supports

<table><thead><tr><th width="228.7147216796875">Provider</th><th data-type="checkbox">Studio</th><th data-type="checkbox">Pro</th><th data-type="checkbox">Research</th><th data-type="checkbox">Pixel </th></tr></thead><tbody><tr><td>OpenAI</td><td>true</td><td>true</td><td>true</td><td>false</td></tr><tr><td>ElevenLabs</td><td>true</td><td>true</td><td>true</td><td>false</td></tr><tr><td>Google Gemini</td><td>true</td><td>true</td><td>true</td><td>false</td></tr><tr><td>Ollama (Local Server)</td><td>false</td><td>true</td><td>true</td><td>false</td></tr><tr><td>OpenRouter</td><td>false</td><td>true</td><td>true</td><td>false</td></tr><tr><td>DeepSeek</td><td>false</td><td>true</td><td>true</td><td>false</td></tr><tr><td>Anthropic</td><td>false</td><td>true</td><td>true</td><td>false</td></tr><tr><td>GroqCloud</td><td>false</td><td>false</td><td>true</td><td>false</td></tr><tr><td>Perplexity</td><td>false</td><td>false</td><td>true</td><td>false</td></tr><tr><td>Microsoft Azure</td><td>false</td><td>false</td><td>true</td><td>false</td></tr><tr><td>xAI (Grok)</td><td>false</td><td>false</td><td>true</td><td>false</td></tr><tr><td>Cohere</td><td>false</td><td>false</td><td>true</td><td>false</td></tr><tr><td>Mistral</td><td>false</td><td>false</td><td>true</td><td>false</td></tr><tr><td>AI21 Labs</td><td>false</td><td>false</td><td>true</td><td>false</td></tr><tr><td>Amazon Bedrock</td><td>false</td><td>false</td><td>true</td><td>false</td></tr><tr><td>Amazon Polly</td><td>false</td><td>false</td><td>true</td><td>false</td></tr><tr><td>Amazon Transcribe</td><td>false</td><td>false</td><td>true</td><td>false</td></tr><tr><td>PixelLab</td><td>false</td><td>false</td><td>false</td><td>true</td></tr></tbody></table>

***

### AI Agents

**AI Agents** lets you use chat APIs (e.g., **Chat Completions**) in Unity via a drop-in **AIAgent** component.\
Attach it to a GameObject, pick a provider/model, and handle requests/responses through events—no custom wiring.

It also makes **tool calls** trivial: register tools, listen to **Tool Events** (status & output), and optionally pipe results back to the model. Hosted tools (web search, code interpreter, MCP, etc.) are supported with simple event hooks.

<table><thead><tr><th width="284.1431884765625">AI Agent</th><th data-type="checkbox">Studio</th><th data-type="checkbox">Pro</th><th data-type="checkbox">Research</th></tr></thead><tbody><tr><td>Chatbot (Chat Completions API)</td><td>false</td><td>true</td><td>true</td></tr><tr><td>Assistant Agent (Assistants API)</td><td>false</td><td>true</td><td>true</td></tr><tr><td>Voice Agent (Realtime API)</td><td>false</td><td>true</td><td>true</td></tr><tr><td><mark style="color:yellow;"><strong>Response Agent (Responses API)</strong></mark></td><td>false</td><td>false</td><td>true</td></tr></tbody></table>

***

### AI Generators

**AI Generators** are Unity components that make it easy to use Generative AI—such as **image generation**, **voice synthesis (TTS)**, and **speech recognition**—directly in Unity. They’re implemented as drop-in components, so you can add them to a GameObject and use them with simple calls or events.

<table><thead><tr><th width="284.1431884765625">AI Generator</th><th data-type="checkbox">Studio</th><th data-type="checkbox">Pro</th><th data-type="checkbox">Research</th></tr></thead><tbody><tr><td>Image Generator</td><td>false</td><td>true</td><td>true</td></tr><tr><td>Speech Generator (TTS)</td><td>false</td><td>true</td><td>true</td></tr><tr><td>Speech Transcriber (STT)</td><td>false</td><td>true</td><td>true</td></tr><tr><td>Content Moderator</td><td>false</td><td>true</td><td>true</td></tr><tr><td>Voice Changer</td><td>false</td><td>true</td><td>true</td></tr><tr><td>Lyria Music Player</td><td>false</td><td>false</td><td>true</td></tr></tbody></table>

***

### **Supported Platforms**

<table><thead><tr><th width="100">Supported</th><th>Platform</th><th data-hidden>All OpenAI APIs (2025.05.18)</th></tr></thead><tbody><tr><td><mark style="color:green;">Fully</mark></td><td>Windows, OSX, Linux</td><td><span data-gb-custom-inline data-tag="emoji" data-code="1f4ac">💬</span> Chat Completions, Completions (Legacy)</td></tr><tr><td><mark style="color:yellow;">Partially</mark></td><td>Unity WebGL</td><td></td></tr><tr><td><mark style="color:green;">Fully</mark></td><td>Android, iOS, Windows Phone/Store</td><td></td></tr><tr><td><mark style="color:green;">Fully</mark></td><td>PlayStation, Xbox, PS Vita/PSM, Switch</td><td></td></tr></tbody></table>
