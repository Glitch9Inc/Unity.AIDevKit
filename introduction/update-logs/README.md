---
icon: calendar-lines-pen
---

# Update Logs

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

* Minor fixes (“dumb mistakes”)

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
* Added fallback logic if Default Resources aren’t generated

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

#### v.4.0.6 (Sep 21, 2025) – *Preview*

* Implemented **Responses API**
* Implemented **MCP Tools**
* Implemented **Local Shell Tools**

#### v.4.0.5 (Sep 2025) – *Preview*

* Added **Voice Changer Window** (new Generator Window)
* Upgraded **Generator Window UIs**
* Fixed **Assistant Profile inspector** issues

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
