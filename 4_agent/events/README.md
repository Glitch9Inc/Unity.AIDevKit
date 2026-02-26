---
icon: bolt
---

# Events

Subscribe to agent events for real-time updates on status changes, conversations, tools, audio, and more.

## Overview

The Agent event system provides a type-safe, generic event subscription mechanism that allows you to respond to various agent activities. Events are dispatched through a central `EventBus` managed by the `AgentControlHub`.

All events implement the `IEvent` marker interface, and you can subscribe to any event type using strongly-typed handlers.

## Event Categories

### Agent Status Events
Track agent lifecycle and status changes:
- `AgentStatusChanged` - Agent status transitions

### Conversation Events
Monitor conversation lifecycle:
- `ConversationCreated` - New conversation created
- `ConversationLoaded` - Conversation loaded from storage
- `ConversationDeleted` - Conversation deleted
- `ConversationTitleUpdated` - Conversation title changed
- `ConversationSummaryUpdated` - Conversation summary updated
- `ConversationItemsLoaded` - Conversation items loaded
- `ConversationListLoaded` - List of conversations retrieved

### Tool Events
Track tool execution lifecycle:
- `ToolCall` - Tool invocation by agent
- `ToolStatusEvent` - Tool execution status updates
- `ToolOutputEvent` - Tool execution results
- `McpApprovalRequest` - MCP tool approval request

### Audio Events
Monitor audio buffer state:
- `AudioBufferStateChanged` - Audio buffer state transitions
- `AudioRateLimitsUpdated` - Rate limit updates for realtime audio

### Delta Events (Streaming)
Real-time streaming updates:
- `Delta<ITextChunk>` - Streaming text updates
- `Delta<IImageChunk>` - Streaming image data
- `Delta<IAudioChunk>` - Streaming audio data
- `Delta<IAnnotationChunk>` - Content annotations

### Metadata Events
General agent metadata:
- `Usage` - Token usage information
- `Exception` - Error events

## Quick Start

### Basic Event Registration

```csharp
using Glitch9.AIDevKit.Agents;
using Glitch9.AIDevKit.Conversations;

Agent agent = new Agent(config, settings);

// Register a status change handler
var subscription = agent.RegisterEvent<AgentStatusChanged>(evt => 
{
    Debug.Log($"Agent status: {evt.NewStatus}");
});

// Unregister when done
subscription.Dispose();
```

### Multiple Event Handlers

```csharp
// Agent status
agent.RegisterEvent<AgentStatusChanged>(OnStatusChanged);

// Conversations
agent.RegisterEvent<ConversationCreated>(OnConversationCreated);
agent.RegisterEvent<ConversationLoaded>(OnConversationLoaded);

// Tools
agent.RegisterEvent<ToolOutputEvent>(OnToolOutput);

// Text streaming
agent.RegisterEvent<Delta<ITextChunk>>(OnTextDelta);

// Usage tracking
agent.RegisterEvent<Usage>(OnUsage);
```

## When to Use Events

Use the event system when you need:

✅ Real-time notifications of agent state changes  
✅ Monitoring conversation lifecycle  
✅ Tracking tool execution  
✅ Processing streaming content  
✅ Collecting usage metrics  
✅ Error handling and logging  

## Next Steps

- [Overview](overview.md) - Detailed event system architecture
- [Registering Events](registering-events.md) - Registration and lifecycle management
- [Available Events](available-events.md) - Complete event reference
