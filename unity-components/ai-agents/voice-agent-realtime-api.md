---
description: >-
  Integrate OpenAI’s Realtime API into your Unity project to enable fully
  voice-driven conversations with millisecond-level latency.
icon: phone-flip
---

# Voice Agent (Realtime API)

### What it does

This component enables:

* Real-time two-way voice interaction using OpenAI's **Realtime API**
* Microphone streaming with live transcription via Whisper
* Low-latency voice output using OpenAI’s built-in real-time voices (e.g., Alloy, Echo)
* Dynamic playback without generating or decoding audio clips locally
* Optional tool calling via FunctionManager
* UnityEvent hooks for transcription, assistant responses, WebSocket events, and more

***

<figure><img src="../../.gitbook/assets/image (109).png" alt="" width="563"><figcaption></figcaption></figure>

### 1. General Settings

| Field          | Description                                                     |
| -------------- | --------------------------------------------------------------- |
| Realtime Model | The assistant model to stream to, e.g., GPT-4o Realtime Preview |
| Voice Actor    | One of OpenAI’s real-time voices (Alloy, Shimmer, Echo, etc.)   |
| Instructions   | System prompt to define assistant behavior                      |
| Auto Start     | Automatically starts streaming on scene start                   |

***

### 2. Input Audio Transcription

| Field                | Description                                            |
| -------------------- | ------------------------------------------------------ |
| Speech-to-Text Model | STT model used for transcription (typically Whisper 1) |
| Spoken Language      | Language spoken by the user                            |

***

### 3. Input Audio Recording

| Field                   | Description                                  |
| ----------------------- | -------------------------------------------- |
| Input Audio Format      | Audio encoding format (e.g., PCM16)          |
| Input Audio Sample Rate | Sample rate in Hz (e.g., 16000)              |
| Input Sample Duration   | Chunk size in milliseconds to send to OpenAI |
| Silence Duration        | Max silence before speech is considered done |
| Silence Threshold       | Volume threshold for silence detection       |

***

### 4. Output Audio

| Field               | Description                            |
| ------------------- | -------------------------------------- |
| Output Audio Format | Format for streamed output from OpenAI |
| Output Audio Volume | Multiplier for playback loudness (0–1) |

> Note: Unlike traditional TTS, **audio is streamed directly from OpenAI’s server in real time.** There is no `AudioClip` or local decoding step.

***

### 5. Event Managers & Receivers

| Field                        | Description                                         |
| ---------------------------- | --------------------------------------------------- |
| Function Manager             | Executes Unity-side methods triggered by tool calls |
| Realtime Event Receiver      | General connection and session-level events         |
| WebSocket Event Receiver     | Low-level socket status updates                     |
| Input Transcription Receiver | Full transcript from user's speech input            |
| Text Event Receiver          | Assistant’s partial or full text response           |
| Transcript Event Receiver    | Final transcript of entire user utterance           |
| Tool Call Receiver           | Triggered when assistant requests tool execution    |

