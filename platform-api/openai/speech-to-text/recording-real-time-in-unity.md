# Recording real-time in Unity

To record voice in real-time within your Unity project, you can utilize the AudioRecorder class to accomplish this task.

{% tabs %}
{% tab title="C#" %}
```csharp
var recorder = new AudioRecorder();
recorder.StartRecording(); // Start recording the voice

// Wait until the recording is done

var audioClip = recorder.StopRecording(); // Stop recording and get the audio clip

// Now you can use the audio clip to request transcription or translation
// For example:

var request = new TranscriptionRequest.Builder()
    .SetModel(WhisperModel.Whisper1)
    .SetFile(audioClip)
    .Build();

var result = await request.ExecuteAsync();

Debug.Log(result.Text);
Debug.Log(result.Language);
Debug.Log(result.Duration);
```
{% endtab %}
{% endtabs %}

#### MonoBehaviour Example

{% tabs %}
{% tab title="C#" %}
```csharp
using UnityEngine;
using System;
using System.IO;
using System.Net.Http;
using System.Threading.Tasks;
using Newtonsoft.Json.Linq;

public class OpenAISpeechToText : MonoBehaviour
{
    private AudioRecorder _audioRecorder;

    private void Start()
    {
        _audioRecorder = new AudioRecorder();
    }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.R))
        {
            _audioRecorder.StartRecording();
        }

        if (Input.GetKeyDown(KeyCode.S))
        {
            _audioRecorder.StopRecording();
        }

        if (Input.GetKeyDown(KeyCode.P))
        {
            _audioRecorder.PlayRecording(gameObject);
        }

        if (Input.GetKeyDown(KeyCode.T))
        {
            StartCoroutine(TranscribeAudio());
        }
    }

    private async UniTask TranscribeAudio()
    {
        AudioClip audioClip = _audioRecorder.GetRecording();
        if (audioClip == null)
        {
            Debug.LogWarning("No audio clip found for transcription.");
            return;
        }

       var request = new TranscriptionRequest.Builder()
            .SetModel(WhisperModel.Whisper1)
            .SetFile(audioClip)
            .Build();

        var result = await request.ExecuteAsync();

        Debug.Log(result.Text);
        Debug.Log(result.Language);
        Debug.Log(result.Duration);
    }
}
```
{% endtab %}
{% endtabs %}
