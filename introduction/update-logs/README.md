---
icon: calendar-lines-pen
---

# Update Logs

#### v.3.4.19 (Apr 21, 2025)

* Added ElevenLabs prompt history builder
* Fixed `CustomEditor` not saving values
* Integrated `FunctionManager` into `ChatCompletionTool` and `ChatSessionTool`
* Fixed broken demo scenes
* Fixed OpenAI prompt history builder
* Fixed Google prompt history builder
* Refactored `PromptHistoryViewer`

#### v.3.4.18 (Apr 20, 2025)

* Changed edit icon for clarity
* Added argument support to `FunctionManager`
* Added save button to `CodeGen`
* Fixed multi-image rendering in `IconGen`
* Improved `ModelSelector` by hiding providers with no models

#### v.3.4.17 (Apr 20, 2025)

* Fixed index out of range issue in `VoiceSelectorGUI`

#### v.3.4.16 (Apr 18, 2025)

* Added `FunctionManager`
* \[Demo] Realtime API + Function integration

#### v.3.4.15 (Apr 18, 2025)

* Fixed model categorizing regex
* Added cancel request feature to `GENTask`
* Implemented `IconGenWindow`
* Cleaned up codegen files
* Added output path tracking for `GeneratedImage` and `GeneratedAudio`
* Fixed realtime API sample voice paths
* Fixed Addressables `amsdef`

#### v.3.4.14 (Apr 18, 2025)

* Removed implicit operator from `EPrefs` to prevent Mono JIT crash

#### v.3.4.13 (Apr 17, 2025)

* Added missing constraints to:
  * `Samples.Editor`
  * `MaterialDesign`
  * `MaterialDesign.Editor`
* Moved `Setup` folder outside of assembly

#### v.3.4.12 (Apr 17, 2025)

* Fixed ElevenLabs voice library errors

#### v.3.4.08 (Apr 17, 2025)

* Refactored `EditorChatToolbar` as separate component
* Implemented export chat as `.txt`
* Displayed chat list names as date instead of thread ID
* Added more options to `GENText` and ElevenLabs `GENTasks`
* Added Texture generator
* Added default model settings to each API
* Added fallback model support using default models
* Renamed `ImageModel` → `DallEModel`, `TTSModel` → `OpenAITTS`
* Fixed issue fetching Google models
* Improved model family categorization

#### v.3.4.07 (Apr 16, 2025)

* Fixed TTS file path issues
* Fixed build errors related to `StartOnInit`
* Fixed Unity hang on startup
* Fixed model update triggering every time
* Added ElevenLabs subscription feature
* Added `VoiceChanger` and `AudioIsolation` to `GENTask`
* Added more options to `GENText`
* Improved `EditorChat` GUI and stream handling
* Added drag & drop support in `EditorChat`
* Added AI-powered script menu options:
  * "Improve Readability"
  * "Refactor"

#### v.3.4.16 (Apr 18, 2025)

* Added model categorizing regex
* Added cancel request feature to `GENTask`
* New feature: `IconGenWindow`
* Added output path tracking to `GeneratedImage` and `GeneratedAudio`
* Fixed Addressables `amsdef`

#### v.3.4.2 (Apr 13, 2025)

* Broken GUIDs fixed

#### v.3.2.0 (Apr 10, 2025)

* Compile errors resolved
* Refactored tons of codes

#### v.3.1.4 (Apr 7, 2025)

* Build errors resolved

#### v.3.1.2 (Mar 31, 2025)

* Added Google image models (Gemini, Imagen) to Editor Tool
* Added new Google model families
* Refactored the project

#### v.3.0.0 (Mar 17, 2025)

* Added automatic model updates on Editor startup
* Added automatic model enum code generation
