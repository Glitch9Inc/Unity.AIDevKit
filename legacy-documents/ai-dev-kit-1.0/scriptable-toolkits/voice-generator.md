# Voice Generator

The `VoiceGenerator` is a component of the **OpenAI with unity** asset that converts text into speech. This functionality is essential for creating interactive and accessible applications, ranging from dynamic character dialogue to voice-driven instructions and information delivery.

### Step 1: Creating a VoiceGenerator Instance

1. In the Unity Editor's top menu, go to `Assets > Create > Glitch9/OpenAI/Toolkits > Voice Generator`.
2. This action will generate a `VoiceGenerator` instance in your Project pane. Select it to view and modify its properties in the Inspector.

### Step 2: Configuring VoiceGenerator

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

1. **Audio File Path**: Specify the directory where the audio files will be saved after generation.
2. Familiarize yourself with the various settings such as supported voices, audio file path, and the default save path which may be automatically populated based on your project settings.

### Step 3: Implementing VoiceGenerator in Your Scene

1. Drag the `VoiceGenerator` ScriptableObject into a relevant script component within your scene that will control the text-to-speech operations.
2. Reference the `VoiceGenerator` in your script to call its methods when needed.

```csharp
csharpCopy codepublic VoiceGenerator voiceGenerator;

public async void GenerateSpeechFromText(string textToSpeak) {
    AudioFile result = await voiceGenerator.Create(textToSpeak, VoiceActor.Alloy);
    // Handle the generated speech audio
}
```

### Step 4: Generating Voice Audio

1. To create a voice audio clip from text, call the `Create` method with the text string and the desired `VoiceActor`.
2. Use `await` or `UniTask` to manage the asynchronous nature of the operation and obtain the `AudioFile` result which represents the generated speech audio.

### Step 5: Customizing the Generated Voice

1. **Voice**: Choose from the available `VoiceActor` options to select the desired voice for the speech.
2. **Speed**: Adjust the speed of speech to match your application's needs (e.g., slow for clarity or fast for brevity).

### Step 6: Retrieving and Using Generated Audio

* Access the generated audio through the `AudioFile` returned by the `Create` method.
* Play the audio clip within your scene or attach it to UI elements as required.

### Step 7: Managing Audio Files

* The `GetRecordings` method can be used to retrieve a list of all the generated audio clips.
* Implement a cleanup routine if your application dynamically generates a lot of speech to avoid excessive memory use or storage consumption.

### Step 8: Error Handling and Validation

* Validate the input text for length and content to ensure it meets OpenAI's API guidelines.
* Implement error handling to catch and manage exceptions or failed operations gracefully.

### Best Practices

* Keep the generated speech concise to maintain user engagement and reduce processing time.
* Regularly back up or clear out the audio file directory based on the frequency and number of generations.
* Utilize user feedback to refine the choice of `VoiceActor` and speech speed for the best user experience.
