# 🎙️ Speech to text

Integrating OpenAI's Speech-to-Text (STT) capabilities into your Unity project enables you to transcribe audio content into written text. This feature is powered by OpenAI's advanced speech recognition models, making it invaluable for applications that involve voice commands, audio content accessibility, or the processing of spoken user inputs.

For detailed information about the Speech-to-Text API, including the models available, parameter options, and best practices for audio files, refer to the [Speech API Reference](https://platform.openai.com/docs/api-reference/speech-to-text).

### Speech-to-Text Operations Overview:

* **Audio Transcription**: Convert spoken words from audio files into accurate written text. This process facilitates the understanding and utilization of spoken language within your applications.
* **Audio Translation**: Convert and translation spoken language into written text in English.

### Sample Code for Speech-to-Text Requests:

#### **1. Audio Transcription Request:**

Transcribe audio content to **text**. You'll need to provide the audio file as a `FormFile` (for API requests) or a `AudioClip` object (within Unity).

{% tabs %}
{% tab title="C#" %}
```javascript
var audioFile = new FormFile(path to speech.mp3);
var request = new TranscriptionRequest.Builder()
    .SetModel(WhisperModel.Whisper1)
    .SetFile(audioFile)
    .Build();

var result = await request.ExecuteAsync();

Debug.Log(result.Text);
Debug.Log(result.Language);
Debug.Log(result.Duration);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

audio_file = open("speech.mp3", "rb")
transcript = client.audio.transcriptions.create(
  model="whisper-1",
  file=audio_file
)
```
{% endtab %}
{% endtabs %}

#### **2. Audio Translation Request:**

Transcribe audio content to <mark style="color:red;">**English text**</mark>. You'll need to provide the audio file as a `FormFile` (for API requests) or a `AudioClip` object (within Unity).

{% tabs %}
{% tab title="C#" %}
```javascript
var audioFile = new FormFile(path to speech.mp3);

var request = new TranslationRequest.Builder()
    .SetModel(WhisperModel.Whisper1)
    .SetFile(audioFile)
    .Build();

var result = await request.ExecuteAsync();

Debug.Log(result.Text);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

audio_file = open("speech.mp3", "rb")
transcript = client.audio.translations.create(
  model="whisper-1",
  file=audio_file
)
```
{% endtab %}
{% endtabs %}

