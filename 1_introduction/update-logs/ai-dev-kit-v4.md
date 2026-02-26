# AI Dev Kit v4

### 4.8.0 (2025-11-30)

**Fixed:**

* Model and Voice assets path normalization issues resolved

---

### 4.7.29 (2025-11-29)

**Fixed:**

* Unity 2022 backward compatibility restored

---

### 4.7.28 (2025-11-28)

**Added:**

* DeepSeek API support (Pro package)

**Fixed:**

* Runtime path normalization issues
* Editor path normalization issues
* Transcriber component issues

---

### 4.7.24-4.7.27 (2025-11-26 ~ 2025-11-28)

**Added:**

* GroqCloud file support in Data Manager
* `GetCredits()` request - returns prepaid balance (used/total)
* FileMetadata to file classes

**Fixed:**

* Multiple filtered popup issues (#2, #3, #4)
* Path issues resolved

**Removed:**

* Removed unused 'RESEARCH_LAB' script define

---

### 4.7.23 (2025-11-23 ~ 2025-11-24)

**Added:**

* OpenRouter Files API (Data Manager)
* Voice `GetVoice()` extension method
* GENSpeech `SetVoice()` extension method
* GENSpeech `SetModel()` extension method (non-generic)
* Model `GetModel()` extension method
* Embedding Request `SetModel()` extension method
* Get Voice API (Google Gemini, ElevenLabs)
* File upload / list / delete requests (OpenRouter)

**Fixed:**

* Voice Metadata Generator / Model Metadata Generator Window (error popup fixed)
* Streaming handler resolved
* Streaming Audio Player fixed
* Auto-resolve PlaybackFormat from request format
* Filtered Popup search function fixed
* Generated Voice `OnPronounce()` callback fixed

**Changed:**

* Dev Chat tool type changed from Coding Action -> Text Chat

---

### 4.7.22 (2025-11-21)

**Added:**

* Perplexity API support (+OpenRouter)
* xAI API support (+OpenRouter)
* Cohere API support (+OpenRouter)
* Mistral API support (+OpenRouter)
* OpenRouter API support (OpenRouter)
* OpenRouter Files API support
* New model metadata system
* Voice metadata system
* Research Lab tier (Pro users exclusive, preview feature)

**Implemented:**

* Chat request with tool calls (Cohere, Mistral, OpenRouter, Perplexity, xAI)
* Chat request without tool calls (Cohere, Mistral, OpenRouter, Perplexity, xAI)
* Tool calling (Cohere, Mistral, OpenRouter, Perplexity, xAI)
* Tool conversion (Cohere, Mistral, OpenRouter, Perplexity, xAI)
* Streaming response (Cohere, Mistral, OpenRouter, Perplexity, xAI)
* Streaming tool calls (Cohere, Mistral, OpenRouter, Perplexity, xAI)
* Input audio (Cohere, Mistral, OpenRouter, Perplexity, xAI)
* Multiple system messages (Cohere, Mistral, OpenRouter, Perplexity, xAI)

**Changed:**

* Provider naming: ElevenLabs Audio API -> ElevenLabs
* Provider naming: OpenRouter (Text Only) -> OpenRouter

**Fixed:**

* OpenAI Realtime Metadata (event id, item id, etc.)
* Tool Events now properly pushed to Debug Window

---

### 4.7.20 (2025-11-19)

**Added:**

* Duplicate Model / Voice feature in Model Library & Voice Library tools
* Model / Voice preset management system
* Voice preset management UI

**Changed:**

* Data Manager: Files are now represented as Unity assets instead of pure C# classes
* FileManager class changed to FileCatalogue
* File assets changed to class File<T> where T can be AudioClip, Texture2D, or VideoClip

---

### 4.7.18-4.7.19 (2025-11-16 ~ 2025-11-18)

**Added:**

* Multiple Tool result events
* Input audio format handling
* Output audio format handling
* Anthropic Message Batches API
* OpenRouter File API (untested)
* Workspace window for navigating editor tools

**Fixed:**

* File API issues
* GenerateSpeech issues

---

### 4.7.16 (2025-11-11 ~ 2025-11-13)

**Added:**

* File manager tool (File API management tool)
* DLL missing warning window at Editor start
* Anthropic Prompt Caching API
* OpenAI GPT-4o Audio Preview API
* Google Gemini Context Caching API

**Fixed:**

* Fixed input delay for speech transcriber
* Fixed Speech Generator double slash issue
* Improved Google Gemini client error message

---

### 4.7.10-4.7.11 (2025-11-09)

**Added:**

* Tool error messages to Debug/Log window
* Tool failure events
* Utility class to get model capabilities at runtime
* OpenAI Responses API Draft (preview, limited testing)

**Changed:**

* Tool output now logged to Debug/Log window (tool name shown for better readability)

---

### 4.7.7-4.7.9 (2025-11-07 ~ 2025-11-08)

**Added:**

* JSON Schema tool description support
* Google Gemini grounding feature with Google Search
* Google Gemini grounding feature Code Execution
* Local Shell Tool support (Windows/Mac/Linux command execution)
* Google Gemini audio generation API

**Changed:**

* Simplified GEN Tasks

**Fixed:**

* OpenAI Realtime API audio output issue
* Multiple Tool-related errors

---

### 4.7.7 (2025-11-05)

**Added:**

* Tool Definition with Google Gemini support
* Tool calls from AI Responses with Google Gemini
* Server => Client tool confirmation (supported on OpenAI, Google Gemini)
* Server => Client request data (supported on OpenAI)
* MCP Server support (stable version)
* Google Gemini audio input support
* Google Gemini video input support
* Google MakerSuite files API (untested)

**Removed:**

* Obsolete Anthropic client APIs

**Fixed:**

* Various minor issues

---

### 4.7.6 (2025-11-03)

**Fixed:**

* Speech Generator audio caching
* Fixed Voice Changer issue

---

### 4.7.5 (2025-10-29)

**Added:**

* OpenAI Realtime API – text/audio streaming support
* OpenAI Realtime API – client => server function calls
* OpenAI Realtime API – server => client function requests
* OpenAI Realtime API – audio response streaming (input/output formats selection)
* OpenAI Realtime API – assistant voice customization
* OpenAI Realtime API – audio input/output event hooks
* OpenAI Realtime API – conversation item serialization
* Assistants API – thread run support
* Assistants API – streaming support
* Assistants API – tool output submission

**Fixed:**

* Model Library issues
* Multiple demo scenes issues

---

### 4.7.4 (2025-10-27)

**Fixed:**

* Model / Voice Library changes not reflecting in prefabs/scenes

---

### 4.7.2 (2025-10-26)

**Fixed:**

* ScriptableObject Toolkit initialization issues

---

### 4.7.1 (2025-10-24)

**Added:**

* Auto-conversion to .wav format if format not supported
* Editor tooltips for model library fields

**Fixed:**

* GENTask errors
* Voice metadata generator
* Model metadata generator

---

### 4.7.0 (2025-10-23)

**Added:**

* Lyria Music Player component
* Lyria Music Track (ScriptableObject)
* Google tools: Google Search, Code Executor, URL Context
* OpenAI Streaming Image API
* TextEmbeddingTask (`.GENTextEmbedding()`)

#### v.4.6.5 (Sep 21, 2025) – *Preview*

* Implemented **Responses API**
* Implemented **MCP Tools**
* Implemented **Local Shell Tools**

#### v.4.6.2 (Sep 2025)

* Internal unitypackage build (`40602`) / packaging updates

#### v.4.5.32 (Sep 03, 2025) – *Pre-Release*

* Added **Voice Changer Window** (new Generator Window)
* Upgraded **Generator Window UIs**
* Fixed **Assistant Profile inspector** issues

#### v.4.5.27 (Aug 30, 2025)

* Fixed missing marker issues
* Fixed initial setup issues
* Cleaned up initial setup process

#### v.4.5.13 (Aug 22, 2025) – *Pre-Release*

* Fixed **assistant chatbot** demo scene
* Fixed **realtime chatbot** demo scene
* Upgraded **MonoBehaviour** and **ScriptableObject** custom editors

#### v.4.5.11 (Aug 22, 2025)

* Fixed broken sample scenes
* Upgraded model asset inspector

#### v.4.5.09 (Aug 22, 2025) – *Hotfix*

* Generator Window hotfixes

#### v.4.5.06 (Aug 22, 2025) – *Pre-Release*

* Merged **Pro assembly** into **Core** (simpler setup)
* Added **Audio Clip–based Prompt Generator** window
* Fixed streaming chat not pushing response messages
* Added Editor session info formatter to Anthropic chat requests
* Polished & stabilized GenAI Windows
* Verified Unity 2022 compatibility

#### v.4.2.22 (Aug 10, 2025) – *Preview*

* Voice Library GUI updates
* Voice Metadata system upgrades
* New **Locale** management system (replaces `SystemLanguage`, stays compatible)

#### v.4.2.21 (Aug 10, 2025) – *Preview*

* Test implementation of **Microsoft Azure TTS/STT** (experimental)

#### v.4.2.20 (Aug 08, 2025) – *Preview*

* Applied Chatbot/Assistant naming rules

  * Components: `Chatbot` (e.g., Realtime Assistant → Realtime Chatbot)
  * Profiles: Assistant Profiles
* Removed icons from foldout headers
* Various editor GUI refinements

#### v.4.2.19 (Aug 07, 2025) – *Hotfix*

* Cleaned up inspectors
* Fixed model ID issue in usage tracking (ChatSession)

#### v.4.2.18 (Aug 07, 2025) – *Preview*

* Introduced **Generative Profile System**
* Added ScriptableObject assets:

  * Local Chatbot Profile
  * Assistants API Chatbot Profile
  * Realtime Assistant Profile
* Implemented custom inspectors for new profiles
* Renamed: `VoiceGender` → `Gender`, `Chatbot` → `LocalChatbot`, `ChatbotBase` → `Chatbot`, `Moderator` → `ContentModerator`
* Enhanced ScriptableObject debugging tools; removed redundant editor code

#### v.4.2.17 (Aug 06, 2025) – *Hotfix*

* Added fallback codes for **ProjectContext** (ScriptableObject)

#### v.4.2.16 (Aug 05, 2025) – *Preview*

* Added **Artificial Analysis API** (internal)
* Refactored model metadata resolvers

#### v.4.2.15 (Aug 06, 2025) – *Hotfix*

* Fixed chat profile not applying to Chatbot

#### v.4.2.14 (Aug 05, 2025) – *Hotfix*

* Added fallback codes to Chatbot, ChatSession, ChatSessionController

#### v.4.2.13 (Aug 05, 2025) – *Hotfix*

* Fixed prompt checks

#### v.4.2.12 (Aug 04, 2025) – *Preview*

* Added **Realtime Assistant Profile**
* Refactored Realtime Assistant
* Refactored Chatbot (Assistants API)
* Removed legacy Assistant lifecycle events
* Added new Assistant status event
* Added Conversation Management support
* Redesigned FileManager editor UI
* Added Assistant Thread TreeView editor window

#### v.4.2.11 (Aug 01, 2025) – *Preview*

* Refactored **UnityEvents** & custom inspector handling
* Improved UnityEvent serialization & GUI rendering
* Added editor icons
* Renamed collections: `ReferencedDictionary` → `SerializableDictionary`, `Metadata` → `SerializableMetadata`, `Database` → `SerializableDatabase`

#### v.4.2.10 (Jul 29, 2025) – *Preview*

* Continuing Pro sample fixes
* Removed unused editor icons
* Organized editor icons & GUI labels

#### v.4.2.9 (Jul 29, 2025) – *Preview*

* Finished new inspector designs
* Fixed WebGL build issues (not fully tested)
* Added/replaced editor icons

#### v.4.2.8 (Jul 28, 2025) – *Hotfix*

* Minor fixes ("dumb mistakes")

#### v.4.2.7 (Jul 28, 2025) – *Preview*

* New inspector designs (almost done)

#### v.4.2.6 (Jul 28, 2025) – *Preview*

* Working on new inspector designs (not finished)
* Separated **Speech Recorder** from AI components that use recording

#### v.4.2.5 (Jul 28, 2025) – *Preview*

* WIP: new **Inspector designs** (not finished)
* **Speech Recorder** separated from AI components using recording feature

#### v.4.2.4 (Jul 27, 2025) – *Preview*

* Refactored **Chatbot & ChatSession** architecture

  * `ChatSession` now holds **mutable data only**
  * Introduced **ChatbotProfile** (ScriptableObject) for immutable data (`Create > AIDevKit > Chatbot`)
  * Chatbot requires both `ChatbotProfile` + `ChatSession`
* Added **Custom Inspectors** for Voice & Model assets
* Added fallback logic if Default Resources aren't generated

#### v.4.2.3 (Jul 25, 2025) – *Preview*

* Fixed new chat issue in EditorChat
* Moved Styles into **Gizmos** folder
* Improved **Generator Window UI**
* Improved **Hub Window UI**
* Tested & debugged **GroqCloud TTS**

#### v.4.2.2 (Jul 24, 2025) – *Preview*

* Added **GroqCloud integration** (Chat tested; STT/TTS untested)
* Improved model & voice metadata management
* Redesigned **GENTask option system** → `.SetOptions(TOption options)`

  * Example: `.SetOptions(new OpenAICompletionOptions())`
* Added `temperature`, `seed`, `stopSequences` to **GENResponse** & ChatSession
* Removed `ReasoningOptions`, `WebSearchOptions`, `AdvancedModelSettings`, `SpeechOutputOptions` → migrated into provider-specific option classes

#### v.4.2.1 (Jul 23, 2025) – *Preview*

* `ApiSettings` are no longer singletons → access via clients

  * Example: `OpenAI.DefaultInstance.Settings` (instead of `OpenAISettings.Instance`)
* New **IApiUser** interface for in-game users

  * Provides `Id` + `GetApiKey(Api api)` override
* Improved UI for **AI Dev Kit Hub (Explorer)**
* Refactored **ApiSettings architecture**
* Added dedicated custom editor for `.asset` files

#### v.4.2.0 (Jul 22, 2025) – *Preview*

⚠️ *Super Warning*: Remove previous version completely (many GUID breaks).

* Complete cleanup of project + strict assembly management
* Added **GENPixelArt** to fluent API
* API-specific enums renamed (e.g., `OpenAI.ImageDetail`, `GoogleTypes.PersonGeneration`)
* Added `CancelLastRequest()`, `CancelAllRequests()` to `AIComponent`
* **Project Context settings** separated from `AIDevKitSettings` → standalone Editor Window

#### v.4.1.3 (Jul 18, 2025) – *Preview*

* New **UnityEvent Wrappers & Improvements**

  * `StreamingUnityEvent<>` wraps 3 UnityEvents: `OnStartStreaming`, `OnReceiveData`, `OnFinishStreaming`
* Improved **CustomPropertyDrawer** (new collapsed texture)
* Renamed Components:

  * `AIComponentBase` → `AIComponent`
  * `GeneratorComponentBase` → `AIContentGenerator`
  * `AudioRecordingComponentBase` → `SpeechToContentGenerator`

#### v.4.1.2 (Jul 17, 2025) – *Preview*

* Added `Api` parameter to **GENTasks**
* Added stricter **Api checks**
* Added new **UnityEvent system** (replace chat event receivers)
* Restored demo scenes (not tested)
* Added storage location selection to **Chat Session**
* Added automatic Chat Session creation on Chatbot start

#### v.4.1.1 (Jul 16, 2025) – *Preview*

* Component reworks completed
* ExTreeView reworks completed
* Added attachment transfer in **GENStructTask**
* Fixed broken demo scenes

#### v.4.1.0 (Jul 15, 2025) – *Preview*

* Complete rework of **component architectures** & custom editors
* Model/Voice Library abstractions (unstable)
* Removed UniTask async methods with return values from components
* Renamed Components:

  * `TextToSpeech` → `SpeechGenerator`
  * `SpeechToText` → `SpeechTranscriber`
* Renamed Classes:

  * `Content` → `ChatContent`
  * `ContentPart` → `ChatContentPart`
  * `RequestType` → `RequestTypes`
* Removed `IChatbot`

#### v.4.0.9 (Jul 14, 2025) – *Preview*

* Chatbot inspector rework (in progress)
* Added **task queue handler** to all components (not tested)
* Added **Shader Generator** windows

#### v.4.0.8 (Jul 13, 2025) – *Preview*

* Prompt history saved as **JSON** (instead of ScriptableObject)

  * Now trackable on devices and at runtime
  * More stable than ScriptableObject (slightly slower)
* Minor Generator Windows fixes

#### v.4.0.7 (Jul 13, 2025) – *Preview*

* Fixed File management system
* Added Demo: **Chatbot + Speech-to-Text**

#### v.4.0.6 (Jul 11, 2025) – *Preview*

* Refactored **ChatSession** for performance
* Added `autoStopRecordingOnSilence` option to **AudioRecordingAIComponent**
* Updated **RealtimeAudioRecorder**
* Fixed modality flag dropdown in Model Library
* Renamed Classes:

  * `ChatCompletionStreamHandler` → `StreamingChatHandler`
  * `ChatCompletionChunk` → `StreamingChatChunk`
  * `AIServerSentEventHandler` → `ServerSentEventHandler`
  * `AIServerSentEvent<T>` → `ServerSentEvent<T>`
  * `AIServerSentEventBuffer<T>` → `ServerSentEventBuffer<T>`

#### v.4.0.5 (Sep 21, 2025) – *Preview*

* Implemented **Responses API**
* Implemented **MCP Tools**
* Implemented **Local Shell Tools**

#### v.4.0.4 (Aug 30, 2025) – *Preview*

* Fixed missing marker issues
* Fixed initial setup issues
* Cleaned up initial setup process

#### v.4.0.3 (Aug 22, 2025) – *Pre-Release*

* Fixed **assistant chatbot demo** scene
* Fixed **realtime chatbot demo** scene
* Upgraded **MonoBehaviour components custom editors**
* Upgraded **ScriptableObject assets custom editors**

#### v.4.0.2 (Aug 22, 2025) – *Pre-Release*

* Merged **Pro assembly** into **Core** to simplify setup
* Added **Audio Clip–based Prompt Generator** window
* Fixed **streaming chat not pushing response messages**
* Added **Editor session info formatter** to Anthropic chat requests
* Polished & stabilized **GenAI Windows**
* Verified **Unity 2022 compatibility**

#### v.4.0.1 (Aug 10, 2025) – *Preview*

* Test implementation of **Microsoft Azure TTS, STT**
* Voice Library GUI updates
* Voice Metadata system upgrades
* New **Locale management system** (replaces SystemLanguage, backward compatible)

#### v.4.0.0 (Aug 7, 2025) – *Preview*

* Introduced **Generative Profile System**
* Added **Local Chatbot Profile**, **Assistants API Chatbot Profile**, **Realtime Assistant Profile**
* Implemented custom inspectors for new generative/chatbot profiles
* Renamed:

  * `VoiceGender` → `Gender`
  * `Chatbot` → `LocalChatbot`
  * `ChatbotBase` → `Chatbot`
  * `Moderator` → `ContentModerator`
* Enhanced **ScriptableObject debugging tools**
* Removed redundant editor code
