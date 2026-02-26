---
icon: microphone
---

# Voice Input (Transcription)

Convert speech to text using AI-powered transcription.

## Overview

Voice input enables your agent to understand spoken language:

- **Speech-to-Text** - Convert audio to text
- **Real-time Transcription** - Stream audio for immediate transcription
- **Multi-language Support** - Support for 50+ languages
- **Automatic Language Detection** - No need to specify language
- **High Accuracy** - Powered by OpenAI Whisper

## Basic Usage

### Start Listening

```csharp
using UnityEngine;
using Glitch9.AIDevKit.Agents;

public class VoiceInput : MonoBehaviour
{
    private Agent agent;
    
    async void Start()
    {
        agent = new Agent(new AgentConfiguration
        {
            EnableVoiceInput = true
        });
        
        await agent.InitializeAsync();
        
        // Listen for voice input events
        agent.OnTranscriptionCompleted += (text) => {
            Debug.Log($"You said: {text}\");
        };
    }
    
    // Call this when user presses push-to-talk button
    async void OnButtonPress()
    {
        await agent.StartListeningAsync();
    }
    
    // Call this when user releases button
    async void OnButtonRelease()
    {
        await agent.StopListeningAsync();
    }
}
```

### Send Audio File

```csharp
// Transcribe pre-recorded audio
AudioClip audioClip = GetRecordedAudio();

var transcription = await agent.TranscribeAsync(audioClip);
Debug.Log($\"Transcribed: {transcription}\");\n```\n\n### Send Audio with Message\n\n```csharp\n// Send voice input and get AI response\nAudioClip audioClip = GetRecordedAudio();\n\nvar response = await agent.SendAudioAsync(audioClip);\nDebug.Log($\"AI: {response.Content}\");\n```\n\n## Recording Audio\n\n### Push-to-Talk\n\n```csharp\npublic class PushToTalk : MonoBehaviour\n{\n    private Agent agent;\n    private bool isRecording;\n    \n    async void Start()\n    {\n        agent = new Agent(new AgentConfiguration\n        {\n            EnableVoiceInput = true\n        });\n        await agent.InitializeAsync();\n    }\n    \n    void Update()\n    {\n        // Hold space to talk\n        if (Input.GetKeyDown(KeyCode.Space))\n        {\n            StartRecording();\n        }\n        \n        if (Input.GetKeyUp(KeyCode.Space))\n        {\n            StopRecording();\n        }\n    }\n    \n    async void StartRecording()\n    {\n        if (isRecording) return;\n        \n        isRecording = true;\n        await agent.StartListeningAsync();\n        Debug.Log(\"Recording started...\");\n    }\n    \n    async void StopRecording()\n    {\n        if (!isRecording) return;\n        \n        isRecording = false;\n        await agent.StopListeningAsync();\n        Debug.Log(\"Recording stopped\");\n    }\n}\n```\n\n### Voice Activation Detection (VAD)\n\n```csharp\npublic class VoiceActivation : MonoBehaviour\n{\n    private Agent agent;\n    private float volumeThreshold = 0.01f;\n    private float silenceDuration = 1.5f;\n    \n    async void Start()\n    {\n        agent = new Agent(new AgentConfiguration\n        {\n            EnableVoiceInput = true,\n            VoiceActivationDetection = true,\n            VoiceActivationThreshold = volumeThreshold,\n            SilenceDetectionDuration = silenceDuration\n        });\n        \n        await agent.InitializeAsync();\n        \n        // Start listening continuously\n        await agent.StartContinuousListeningAsync();\n    }\n    \n    void OnDestroy()\n    {\n        agent?.StopContinuousListening();\n    }\n}\n```\n\n## Transcription Options\n\n### Language Specification\n\n```csharp\n// Auto-detect language (default)\nvar text = await agent.TranscribeAsync(\n    audioClip,\n    language: null\n);\n\n// Specify language for better accuracy\nvar text = await agent.TranscribeAsync(\n    audioClip,\n    language: \"en\"  // English\n);\n\nvar text = await agent.TranscribeAsync(\n    audioClip,\n    language: \"ko\"  // Korean\n);\n```\n\n### Prompts for Context\n\n```csharp\n// Provide context to improve accuracy\nvar text = await agent.TranscribeAsync(\n    audioClip,\n    prompt: \"This is a conversation about Unity game development.\"\n);\n\n// Specify terminology\nvar text = await agent.TranscribeAsync(\n    audioClip,\n    prompt: \"Keywords: GameObject, Transform, Rigidbody\"\n);\n```\n\n### Temperature Control\n\n```csharp\n// Lower temperature = more consistent (default: 0)\nvar text = await agent.TranscribeAsync(\n    audioClip,\n    temperature: 0.0f\n);\n\n// Higher temperature = more creative\nvar text = await agent.TranscribeAsync(\n    audioClip,\n    temperature: 0.8f\n);\n```\n\n## Audio Format Options\n\n```csharp\n// Configure audio format\nvar config = new AgentConfiguration\n{\n    InputAudioFormat = AudioFormat.Wav,\n    AudioSampleRate = 16000,  // 16kHz recommended\n    AudioChannels = 1         // Mono\n};\n\nAgent agent = new Agent(config);\n```\n\n**Supported Formats:**\n- WAV\n- MP3\n- M4A\n- WebM\n- FLAC\n\n## Real-time Streaming\n\n```csharp\npublic class StreamingTranscription : MonoBehaviour\n{\n    private Agent agent;\n    \n    async void Start()\n    {\n        agent = new Agent(new AgentConfiguration\n        {\n            EnableAudioStreaming = true\n        });\n        \n        await agent.InitializeAsync();\n        \n        // Partial transcription as it's being processed\n        agent.OnPartialTranscription += (partialText) => {\n            Debug.Log($\"Partial: {partialText}\");\n            UpdateSubtitles(partialText);\n        };\n        \n        // Final transcription\n        agent.OnTranscriptionCompleted += (finalText) => {\n            Debug.Log($\"Final: {finalText}\");\n            DisplayFinalText(finalText);\n        };\n        \n        // Start streaming\n        await agent.StartStreamingTranscriptionAsync();\n    }\n}\n```\n\n## Events\n\n```csharp\n// Recording started\nagent.OnRecordingStarted += () => {\n    Debug.Log(\"Started recording\");\n    ShowRecordingIndicator();\n};\n\n// Recording stopped\nagent.OnRecordingStopped += () => {\n    Debug.Log(\"Stopped recording\");\n    HideRecordingIndicator();\n};\n\n// Audio chunk received (streaming)\nagent.OnAudioChunkReceived += (chunk) => {\n    UpdateAudioWaveform(chunk);\n};\n\n// Transcription started\nagent.OnTranscriptionStarted += () => {\n    Debug.Log(\"Transcribing...\");\n    ShowProcessingIndicator();\n};\n\n// Partial transcription (streaming)\nagent.OnPartialTranscription += (text) => {\n    UpdateLiveSubtitles(text);\n};\n\n// Transcription completed\nagent.OnTranscriptionCompleted += (text) => {\n    Debug.Log($\"Transcribed: {text}\");\n    HideProcessingIndicator();\n    ProcessTranscribedText(text);\n};\n\n// Transcription failed\nagent.OnTranscriptionFailed += (error) => {\n    Debug.LogError($\"Transcription failed: {error.Message}\");\n    ShowErrorMessage(\"Could not understand. Please try again.\");\n};\n\n// Voice detected (VAD)\nagent.OnVoiceDetected += () => {\n    Debug.Log(\"Voice detected, starting recording\");\n};\n\n// Silence detected (VAD)\nagent.OnSilenceDetected += () => {\n    Debug.Log(\"Silence detected, stopping recording\");\n};\n```\n\n## Microphone Management\n\n### Select Microphone\n\n```csharp\n// List available microphones\nstring[] microphones = Microphone.devices;\nforeach (var mic in microphones)\n{\n    Debug.Log($\"Available mic: {mic}\");\n}\n\n// Select specific microphone\nagent.Configuration.MicrophoneDevice  = \"Built-in Microphone\";\n\n// Or use default (null)\nagent.Configuration.MicrophoneDevice = null;\n```\n\n### Check Permissions\n\n```csharp\nvoid Start()\n{\n    // Request microphone permission\n    if (!Application.HasUserAuthorization(UserAuthorization.Microphone))\n    {\n        Application.RequestUserAuthorization(UserAuthorization.Microphone);\n    }\n}\n\nasync void CheckAndStartListening()\n{\n    if (Application.HasUserAuthorization(UserAuthorization.Microphone))\n    {\n        await agent.StartListeningAsync();\n    }\n    else\n    {\n        Debug.LogError(\"Microphone permission denied\");\n    }\n}\n```\n\n## Use Cases\n\n### Voice Commands\n\n```csharp\nagent.OnTranscriptionCompleted += async (text) => {\n    // Convert to lower case for matching\n    string command = text.ToLower();\n    \n    if (command.Contains(\"jump\"))\n    {\n        player.Jump();\n    }\n    else if (command.Contains(\"attack\"))\n    {\n        player.Attack();\n    }\n    else if (command.Contains(\"help\"))\n    {\n        // Get AI response\n        var response = await agent.SendMessageAsync(text);\n        await agent.SpeakAsync(response.Content);\n    }\n};\n```\n\n### Dictation\n\n```csharp\npublic class DictationSystem : MonoBehaviour\n{\n    private Agent agent;\n    private StringBuilder dictationText = new StringBuilder();\n    \n    void Start()\n    {\n        agent.OnTranscriptionCompleted += (text) => {\n            // Append to dictation\n            dictationText.AppendLine(text);\n            UpdateTextDisplay(dictationText.ToString());\n        };\n    }\n}\n```\n\n### Voice Chat\n\n```csharp\npublic class VoiceChat : MonoBehaviour\n{\n    private Agent agent;\n    \n    async void Start()\n    {\n        agent.OnTranscriptionCompleted += async (userSpeech) => {\n            // Show user's transcribed speech\n            AddChatMessage(\"You\", userSpeech);\n            \n            // Get AI response\n            var response = await agent.SendMessageAsync(userSpeech);\n            \n            // Show AI response\n            AddChatMessage(\"Agent\", response.Content);\n            \n            // Speak response\n            await agent.SpeakAsync(response.Content);\n        };\n    }\n}\n```\n\n## Audio Quality Tips\n\n### Recording Settings\n\n```csharp\n// Optimal settings for speech\nvar config = new AgentConfiguration\n{\n    AudioSampleRate = 16000,  // 16kHz is optimal for speech\n    AudioChannels = 1,        // Mono is sufficient\n    InputAudioFormat = AudioFormat.Wav\n};\n```\n\n### Noise Reduction\n\n```csharp\n// Enable noise reduction\nagent.Configuration.EnableNoiseReduction = true;\n\n// Set noise gate threshold\nagent.Configuration.NoiseGateThreshold = -40f; // dB\n```\n\n### Audio Normalization\n\n```csharp\n// Normalize audio levels\nagent.Configuration.EnableAudioNormalization = true;\n```\n\n## Best Practices\n\n### 1. Provide Feedback\n\n```csharp\n// Visual feedback during recording\nagent.OnRecordingStarted += () => {\n    microphoneIcon.color = Color.red;\n    ShowWaveform();\n};\n\nagent.OnRecordingStopped += () => {\n    microphoneIcon.color = Color.white;\n    HideWaveform();\n};\n```\n\n### 2. Handle Errors Gracefully\n\n```csharp\nagent.OnTranscriptionFailed += (error) => {\n    if (error is NoSpeechDetectedException)\n    {\n        ShowMessage(\"No speech detected. Please try again.\");\n    }\n    else if (error is AudioTooShortException)\n    {\n        ShowMessage(\"Audio too short. Please speak longer.\");\n    }\n    else\n    {\n        ShowMessage(\"Could not understand. Please try again.\");\n    }\n};\n```\n\n### 3. Optimize for Context\n\n```csharp\n// Provide context for better accuracy\nvar text = await agent.TranscribeAsync(\n    audioClip,\n    prompt: GetConversationContext()\n);\n\nstring GetConversationContext()\n{\n    // Return recent conversation topics\n    return \"This conversation is about: \" + string.Join(\", \", topics);\n}\n```\n\n### 4. Validate Input\n\n```csharp\nagent.OnTranscriptionCompleted += async (text) => {\n    // Filter out very short inputs\n    if (text.Length < 3)\n    {\n        ShowMessage(\"Please speak more clearly\");\n        return;\n    }\n    \n    // Process valid input\n    await ProcessVoiceInput(text);\n};\n```\n\n## Limitations\n\n- **Audio Length**: Maximum 25MB file size\n- **Real-time Latency**: 1-3 seconds delay\n- **Ambient Noise**:  May affect accuracy\n- **Accents**: Accuracy varies by accent\n- **Technical Terms**: May misinterpret specialized vocabulary\n\n## Troubleshooting\n\n### No Microphone Detected\n\n```csharp\nif (Microphone.devices.Length == 0)\n{\n    Debug.LogError(\"No microphone detected\");\n    ShowMessage(\"Please connect a microphone\");\n    return;\n}\n```\n\n### Low Accuracy\n\n```csharp\n// Specify language\nlanguage: \"en\"\n\n// Provide context\nprompt: \"Technical discussion about Unity development\"\n\n// Check audio quality\nif (audioClip.frequency < 16000)\n{\n    Debug.LogWarning(\"Audio sample rate too low\");\n}\n```\n\n### Recording Not Working\n\n```csharp\n// Check permissions\nif (!Application.HasUserAuthorization(UserAuthorization.Microphone))\n{\n    Debug.LogError(\"Microphone permission not granted\");\n}\n\n// Check if already recording\nif (Microphone.IsRecording(null))\n{\n    Debug.LogWarning(\"Already recording\");\n}\n```\n\n## Next Steps\n\n- Learn about [Voice Output (Speech)](voice-output.md)\n- Configure [Audio Setup](../essentials/configuration/audio-setup.md)\n- Explore [Image Input (Vision)](image-input.md)\n