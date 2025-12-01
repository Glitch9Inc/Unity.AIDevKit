---
icon: store
---

# Overview

## What is an Agent?

The **Agent** is the core class in AI Dev Kit that manages AI-powered conversations, tool execution, and multi-modal interactions. It acts as a unified façade that wraps:

* **Language models** for text generation
* **Conversation management** for context and history
* **Tool execution** for function calling and integrations
* **Audio I/O** for speech-to-text and text-to-speech
* **Image generation** for visual content

## Key Design Principles

Previously, AI Dev Kit had multiple specialized agent types (ChatAgent, VoiceAgent, AssistantsAgent). The current architecture consolidates all functionality into a single `Agent` class where:

* **Core behavior** lives in one unified Agent class
* **Features** are split by interfaces and internal services
* **Specialization** is driven by configuration (AgentSettings/AgentBehaviour) rather than subclasses

This design improves maintainability and makes it easier to combine features without creating a complex inheritance hierarchy.

## Agent vs AgentBehaviour

There are two main classes you'll work with:

### Agent

The core runtime class that handles all AI logic:

* Pure C# class with no Unity dependencies
* Manages API calls, responses, and state
* Can be used in non-Unity contexts

### AgentBehaviour

A Unity MonoBehaviour wrapper for Agent:

* Attaches to GameObjects in your scene
* Provides inspector-friendly configuration
* Handles Unity-specific lifecycle (Start, OnDestroy)
* Exposes simplified API for UI components

**When to use which:**

* Use `Agent` directly for backend logic, testing, or non-Unity projects
* Use `AgentBehaviour` for Unity scenes with UI, audio, and GameObjects

## Main Features

### 🗨️ Conversation Management

* Create, load, and save conversations
* Support for multiple storage types (memory, local file, cloud)
* Context assembly with system instructions and memory

### 🤖 Multiple Chat Service APIs

* **Chat Completions API** - Standard request/response
* **Assistants API** - OpenAI's stateful assistant threads
* **Realtime API** - Low-latency voice conversations
* **Responses API** - OpenAI's new unified API

### 🔧 Tool Execution

* Function calling
* Local shell commands
* Computer use (Anthropic)
* Hosted tools (file search, code interpreter, web search)
* Local tools (image generation, speech generation)
* Google tools (search, URL context, code execution)
* MCP (Model Context Protocol) integration

### 🎙️ Multi-Modal Support

* **Audio Input** - Record and transcribe speech
* **Audio Output** - Generate speech from text
* **Image Generation** - Create images via DALL-E, Stable Diffusion, etc.
* **File Attachments** - Send images, documents, and other files

### ⚙️ Advanced Configuration

* Dynamic model switching
* Temperature, max tokens, and other parameters
* Streaming vs non-streaming responses
* Tool choice strategies
* Memory and context management

## Quick Example

```csharp
using Glitch9.AIDevKit.Agents;

public class SimpleAgentExample : MonoBehaviour
{
    [SerializeField] private AgentSettings settings;
    
    private Agent agent;
    
    async void Start()
    {
        // Create agent
        agent = new Agent(
            settings: settings,
            behaviour: this, // or create a custom IAgentBehaviour
            hooks: new AgentHooks
            {
                TextDelta = OnTextDelta
            }
        );
        
        // Initialize
        await agent.InitializeAsync();
        
        // Send message
        await agent.SendAsync("Hello! How can you help me?");
    }
    
    private void OnTextDelta(string delta)
    {
        Debug.Log($"Received: {delta}");
    }
    
    private void OnDestroy()
    {
        agent?.Dispose();
    }
}
```

## Next Steps

* [What is AgentBehaviour?](what-is-agentbehaviour.md) - Learn about the Unity wrapper
* [Agent vs AgentBehaviour](agent-vs-agentbehaviour.md) - Understand the differences
* [Quick Start](quick-start.md) - Get started with your first agent
* [Agent Configuration](../configuration/) - Configure agent settings
* [Generating Responses](../responses/) - Send messages and handle responses
