---
icon: diagram-project
---

# How Agents Work

This page explains the internal architecture of an Agent and how all the components work together.

## Architecture Overview

```mermaid
graph TB
    subgraph Unity["Unity Scene"]
        GO[GameObject]
        AB[AgentBehaviour<br/>MonoBehaviour]
    end
    
    subgraph Core["Agent Core (C# Class)"]
        A[Agent]
        AS[AgentSettings]
        
        subgraph Controllers["Controllers"]
            PC[ParametersController]
            CC[ConversationController]
            AC[AudioController]
            TC[ToolCallController]
            ER[EventRouter]
        end
        
        subgraph Services["Services"]
            PS[AgentPrefsService]
            CS[ConversationService]
            CAS[ChatApiService]
        end
    end
    
    subgraph External["External"]
        API[AI Provider API<br/>OpenAI/Anthropic/etc]
        STORE[Storage<br/>Local/Remote]
    end
    
    GO -->|contains| AB
    AB -->|owns| A
    A -->|uses| AS
    
    A --> PC
    A --> CC
    A --> AC
    A --> TC
    A --> ER
    
    PC --> PS
    CC --> CS
    CC --> CAS
    
    ER -->|events| AB
    AB -->|UnityEvents| Unity
    
    CAS -->|HTTP| API
    CS -->|save/load| STORE
    PS -->|save/load| STORE
    
    style AB fill:#e1f5ff
    style A fill:#fff4e1
    style Controllers fill:#f0f0f0
    style Services fill:#f0f0f0
```

## Component Breakdown

### 1. GameObject & AgentBehaviour (MonoBehaviour)

**AgentBehaviour** is the Unity component you attach to a GameObject. It:

- Handles Unity lifecycle (Start, OnDestroy)
- Exposes UnityEvents for UI/gameplay integration
- Creates and manages the core **Agent** instance
- Bridges Unity and C# pure logic

```mermaid
graph LR
    AB[AgentBehaviour] -->|Start| INIT[Initialize Agent]
    AB -->|User Input| SEND[SendMessage]
    SEND --> A[Agent.SendAsync]
    A -->|Response| EVT[Events]
    EVT --> UE[UnityEvents]
    UE --> UI[Update UI]
```

### 2. Agent (Core C# Class)

**Agent** is a pure C# class (no MonoBehaviour) that orchestrates everything:

```mermaid
graph TB
    A[Agent]
    
    A -->|Configuration| PC[ParametersController]
    A -->|Conversations| CC[ConversationController]
    A -->|Voice I/O| AC[AudioController]
    A -->|Tool Calls| TC[ToolCallController]
    A -->|Event Distribution| ER[EventRouter]
    
    PC -->|Model/Voice/Settings| PARAMS[Parameters]
    CC -->|Messages| CONV[Conversation]
    AC -->|AudioClip| AUDIO[Audio I/O]
    TC -->|Tool Execution| TOOLS[Tools]
    ER -->|Callbacks| EVENTS[Event Handlers]
```

### 3. Controllers

Each controller handles a specific domain:

#### ParametersController

Manages model, voice, and configuration settings:

```mermaid
graph LR
    PC[ParametersController] -->|Loads| PREFS[AgentPrefs]
    PC -->|Resolves| MODEL[Model]
    PC -->|Resolves| VOICE[Voice]
    PC -->|Validates| TOOLS[Tools]
    PC -->|Creates| PARAMS[Parameters Object]
```

#### ConversationController

Handles message history and context:

```mermaid
graph LR
    CC[ConversationController] -->|Creates| CONV[Conversation]
    CC -->|Loads/Saves| STORE[ConversationStore]
    CC -->|Adds| MSG[Messages]
    CC -->|Assembles| CTX[Context for API]
```

#### AudioController

Manages voice input/output:

```mermaid
graph LR
    AC[AudioController] -->|Records| MIC[Microphone]
    AC -->|Transcribes| STT[Speech-to-Text]
    AC -->|Synthesizes| TTS[Text-to-Speech]
    AC -->|Plays| SPEAKER[AudioSource]
```

#### ToolCallController

Executes tool calls:

```mermaid
graph LR
    TC[ToolCallController] -->|Receives| CALL[ToolCall]
    TC -->|Routes| EXEC[Tool Executor]
    EXEC -->|Returns| OUTPUT[ToolOutput]
    TC -->|Submits| API[Back to API]
```

#### EventRouter

Distributes events to listeners:

```mermaid
graph LR
    ER[EventRouter] -->|Text Delta| TXT[onTextDelta]
    ER -->|Tool Status| TOOL[onToolStatus]
    ER -->|Audio| AUD[onAudio]
    ER -->|Status| STATUS[onStatusChanged]
    ER -->|Complete| DONE[onResponseComplete]
```

## Message Flow

Here's what happens when you send a message:

```mermaid
sequenceDiagram
    participant User
    participant AgentBehaviour
    participant Agent
    participant ConversationController
    participant ParametersController
    participant ChatApiService
    participant API as AI Provider
    
    User->>AgentBehaviour: SendMessage("Hello")
    AgentBehaviour->>Agent: SendAsync("Hello")
    
    Agent->>ConversationController: AddUserMessage("Hello")
    ConversationController->>ConversationController: Save to Conversation
    
    Agent->>ParametersController: Get Parameters
    ParametersController-->>Agent: Model, Voice, Settings
    
    Agent->>ConversationController: AssembleContext()
    ConversationController-->>Agent: Full message history
    
    Agent->>ChatApiService: SendRequest(context, params)
    ChatApiService->>API: HTTP POST /chat/completions
    
    API-->>ChatApiService: Streaming response
    
    loop For each token
        ChatApiService-->>Agent: Text delta
        Agent->>EventRouter: Emit onTextDelta
        EventRouter->>AgentBehaviour: Fire UnityEvent
        AgentBehaviour->>User: Update UI
    end
    
    API-->>ChatApiService: Complete
    ChatApiService-->>Agent: Final response
    
    Agent->>ConversationController: AddAssistantMessage(response)
    ConversationController->>ConversationController: Save
    
    Agent->>EventRouter: Emit onResponseComplete
    EventRouter->>AgentBehaviour: Fire UnityEvent
    AgentBehaviour->>User: Show complete message
```

## Tool Call Flow

When the AI wants to use a tool:

```mermaid
sequenceDiagram
    participant API as AI Provider
    participant Agent
    participant ToolCallController
    participant ToolExecutor
    participant User as Your Code
    
    API->>Agent: Response with tool_calls
    Agent->>ToolCallController: HandleToolCalls(calls)
    
    loop For each tool call
        ToolCallController->>ToolCallController: Find Executor
        
        alt Executor Found
            ToolCallController->>ToolExecutor: Execute(tool, args)
            ToolExecutor->>User: Your function runs
            User-->>ToolExecutor: Return result
            ToolExecutor-->>ToolCallController: ToolOutput
        else No Executor
            ToolCallController->>Agent: Emit UnhandledToolCall
            Agent->>User: onUnhandledToolCall event
        end
    end
    
    ToolCallController->>Agent: All outputs ready
    Agent->>API: Submit tool outputs
    API->>Agent: Continue with results
```

## Initialization Flow

What happens when you start an Agent:

```mermaid
graph TB
    START([GameObject.Start]) --> CHECK{Agent exists?}
    CHECK -->|No| CREATE[Create new Agent]
    CHECK -->|Yes| INIT
    
    CREATE --> INIT[Agent.InitAsync]
    
    INIT --> INIT_PARAMS[ParametersController.InitAsync]
    INIT_PARAMS --> LOAD_PREFS[Load AgentPrefs]
    LOAD_PREFS --> RESOLVE_MODEL[Resolve Model]
    RESOLVE_MODEL --> RESOLVE_VOICE[Resolve Voice]
    RESOLVE_VOICE --> VALIDATE_TOOLS[Validate Tools]
    VALIDATE_TOOLS --> CREATE_PARAMS[Create Parameters]
    
    INIT --> INIT_CONV[ConversationController.InitAsync]
    INIT_CONV --> LOAD_CONV{Load saved conversation?}
    LOAD_CONV -->|Yes| RESTORE[Restore Conversation]
    LOAD_CONV -->|No| NEW_CONV[Create new Conversation]
    
    INIT --> INIT_AUDIO[AudioController.InitAsync]
    INIT_AUDIO --> SETUP_MIC[Setup Microphone]
    SETUP_MIC --> SETUP_SPEAKER[Setup AudioSource]
    
    INIT --> INIT_TOOLS[ToolCallController.InitAsync]
    INIT_TOOLS --> REG_TOOLS[Register Tool Executors]
    
    CREATE_PARAMS --> READY
    NEW_CONV --> READY
    RESTORE --> READY
    SETUP_SPEAKER --> READY
    REG_TOOLS --> READY
    
    READY([Agent Ready]) --> EMIT_READY[Emit onReady event]
    EMIT_READY --> START_MSG{Has starting message?}
    START_MSG -->|Yes| SEND_START[Send starting message]
    START_MSG -->|No| IDLE[Idle, waiting for input]
```

## Status Lifecycle

Agent goes through different statuses:

```mermaid
stateDiagram-v2
    [*] --> Uninitialized
    Uninitialized --> Initializing : InitAsync()
    Initializing --> Ready : Init Complete
    Initializing --> Failed : Init Error
    
    Ready --> Processing : SendMessage()
    Processing --> Ready : Response Complete
    Processing --> Failed : API Error
    
    Ready --> Disposed : Dispose()
    Failed --> Disposed : Dispose()
    Processing --> Disposed : Dispose()
    
    Disposed --> [*]
    
    note right of Processing
        - Waiting for API
        - Streaming response
        - Executing tools
    end note
    
    note right of Ready
        - Can send messages
        - Can change settings
        - Can save/load
    end note
```

## Data Flow

How data moves through the system:

```mermaid
graph TB
    subgraph Input
        TXT[Text Input]
        AUD[Audio Input]
        FILE[File Input]
    end
    
    subgraph Processing
        CONV[Conversation]
        CTX[Context Assembly]
        REQ[API Request]
    end
    
    subgraph Output
        RESP[AI Response]
        TOOLS[Tool Calls]
        AUDIO[Audio Output]
    end
    
    subgraph Storage
        LOCAL[Local Files]
        REMOTE[Remote Store]
        PREFS[Preferences]
    end
    
    TXT --> CONV
    AUD -->|Transcribe| CONV
    FILE --> CONV
    
    CONV --> CTX
    CTX --> REQ
    REQ --> API[AI Provider]
    
    API --> RESP
    API --> TOOLS
    RESP -->|TTS| AUDIO
    
    CONV -.->|Save| LOCAL
    CONV -.->|Save| REMOTE
    PREFS -.->|Save| LOCAL
    
    LOCAL -.->|Load| CONV
    REMOTE -.->|Load| CONV
    LOCAL -.->|Load| PREFS
```

## Key Takeaways

1. **AgentBehaviour** = Unity wrapper (MonoBehaviour)
2. **Agent** = Core logic (pure C#, no Unity dependencies)
3. **Controllers** = Specialized handlers (Parameters, Conversation, Audio, Tools, Events)
4. **Services** = External interactions (API, Storage, Preferences)
5. **EventRouter** = Event distribution hub

This architecture allows:

- ✅ Clean separation of concerns
- ✅ Easy testing (Agent is pure C#)
- ✅ Flexible event handling
- ✅ Modular tool system
- ✅ Multiple storage options

## Next Steps

- [Agent vs Request](agent-vs-request.md) - When to use which approach
- [Your First Agent](your-first-agent.md) - Create your first agent
- [Creating an Agent](../getting-started/creating-agent.md) - Step-by-step guide
