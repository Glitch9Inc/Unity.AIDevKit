# 🗣️ Text to speech

Text-to-Speech (TTS) is a process where written text is converted into spoken voice output. TTS technologies allow applications to "read" text aloud, providing an auditory version of written materials. This is particularly useful in applications requiring auditory feedback, such as navigational aids, accessibility features for visually impaired users, or interactive voice responses in games and apps.

For a deep dive into the technical details and options available for these requests, refer to the [Speech API Reference](https://platform.openai.com/docs/api-reference/audio).

### Sample Code for Speech Requests:

{% tabs %}
{% tab title="JavaScript" %}
```javascript
var request = new SpeechRequest.Builder()
    .SetModel(TTSModel.TTS1)
    .SetVoice(VoiceActor.Alloy)
    .SetInput("The quick brown fox jumped over the lazy dog.")
    .Build();

var result = await request.ExecuteAsync();

AudioClip audioClip = result.AudioOutput;

// Now you can use the audio clip in your project.
// For example:
var audioSource = gameObject.AddComponent<AudioSource>();
audioSource.clip = audioClip;
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from pathlib import Path
import openai

speech_file_path = Path(__file__).parent / "speech.mp3"
response = openai.audio.speech.create(
  model="tts-1",
  voice="alloy",
  input="The quick brown fox jumped over the lazy dog."
)
response.stream_to_file(speech_file_path)
```
{% endtab %}
{% endtabs %}
