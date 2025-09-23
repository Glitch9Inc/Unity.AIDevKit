# Voice Transcriber

The `VoiceTranscriber` toolkit is an integral part of the **OpenAI with unity** asset, enabling the conversion of speech to text. This tool is ideal for applications requiring voice command recognition, dialogue systems, accessibility features, or any functionality where user voice input needs to be interpreted as text.

### Step 1: Creating a VoiceTranscriber Instance

1. In Unity's top menu, navigate to `Assets > Create > Glitch9/OpenAI/Toolkits > Voice Transcriber`.
2. A `VoiceTranscriber` instance will appear in your project directory. Click on it to adjust its properties in the Inspector.

### Step 2: Configuring VoiceTranscriber

<figure><img src="../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

1. **Audio File Path**: Set the path where audio recordings will be saved.
2. Review additional properties that may be provided for more advanced settings and preferences.

### Step 3: Implementing VoiceTranscriber in Your Unity Scene

1. Assign the `VoiceTranscriber` ScriptableObject to a controller script within your scene that will handle voice recording and transcription.
2. Add a reference to the `VoiceTranscriber` in your script to invoke its methods for recording and transcribing audio.

```csharp
csharpCopy codepublic VoiceTranscriber voiceTranscriber;

// Call this method to start recording voice input from the user.
public void StartRecordingVoice() {
    voiceTranscriber.StartRecording();
}
```

### Step 4: Recording and Transcribing Speech

1. Use the `StartRecording` method to begin capturing audio from the user's microphone.
2. Once the desired audio has been captured, call the `StopAndTranscribeRecordingAsync` method to end the recording and start the transcription process.

```csharp
csharpCopy code// Call this method to stop recording and transcribe the captured audio.
public async void StopAndTranscribe() {
    AudioFile transcriptionResult = await voiceTranscriber.StopAndTranscribeRecordingAsync();
    // Use the transcribed text from transcriptionResult
}
```

### Step 5: Handling Transcription Results

* The transcription result will be contained within the `AudioFile` object returned by the `StopAndTranscribeRecordingAsync` method, including both the audio clip and the transcribed text.

### Step 6: Managing Transcriptions and Recordings

* Maintain a reference to the audio recordings and transcriptions using the `GetRecordings` method, which provides access to all recorded and transcribed sessions.

### Step 7: Customizing the Transcription Process

* Adjust settings such as recording duration and audio frequency for optimized performance specific to your application's environment and requirements.

### Step 8: Error Handling and User Feedback

* Implement error handling to manage potential issues during recording or transcription.
* Provide users with feedback when recording starts and ends, and inform them of any errors or transcription status updates.

### Best Practices

* Test and calibrate the microphone input settings in various environments to ensure reliable voice capture.
* Prompt users to speak clearly and consider implementing a voice activity detection system to initiate and terminate recordings efficiently.
* Handle personal user data responsibly, especially if recording sensitive information.
