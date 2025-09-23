---
description: Upload files, images, or audio clips to AI providers (OpenAI / Google) with metadata support
icon: up-from-bracket
--- 

# UploadFile

returns [`IUploadedFile`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.IUploadedFile.html)

Upload text files, images (`Texture2D`), or audio clips (`AudioClip`) via **multipart form-data** to AI providers.

> Internally, all uploads are sent as `MimeType.MultipartFormData`. The result is normalized into `IUploadedFile`.

---

## Basic Usage (Local File)

```csharp
using UnityEngine;
using Glitch9.AIDevKit;

// Any IFile implementation is supported (disk, memory, stream)
IFile file = new LocalDiskFile("C:/datasets/train.jsonl", MimeType.ApplicationJson);

IUploadedFile uploaded = await new UploadFileTask(Api.OpenAI, file)
    .SetOpenAIUploadPurpose(OpenAITypes.UploadPurpose.Assistants) // default is Assistants
    .ExecuteAsync();

Debug.Log($"Uploaded: {uploaded.Id} ({uploaded.FileName}, {uploaded.SizeBytes} bytes)");
```

---

## Upload an Image (Texture2D)

```csharp
Texture2D tex = ...; // Load or generate at runtime

IUploadedFile uploadedImage = await new UploadFileTask(Api.OpenAI, tex)
    .SetOpenAIUploadPurpose(OpenAITypes.UploadPurpose.Assistants)
    .ExecuteAsync();

Debug.Log($"Image uploaded: {uploadedImage.Id}");
```

---

## Upload an AudioClip

```csharp
AudioClip clip = ...; // Load from mic or asset

IUploadedFile uploadedAudio = await new UploadFileTask(Api.OpenAI, clip)
    .SetOpenAIUploadPurpose(OpenAITypes.UploadPurpose.Assistants)
    .ExecuteAsync();

Debug.Log($"Audio uploaded: {uploadedAudio.Id}");
```

---

## Google Upload (with Metadata)

```csharp
var metadata = new GoogleTypes.UploadMetadata
{
    Name = "my-dataset.jsonl",
    Description = "Training corpus v3",
    MimeType = MimeType.ApplicationJson
};

IFile file = new LocalDiskFile("C:/datasets/train.jsonl", MimeType.ApplicationJson);

IUploadedFile gcsFile = await new UploadFileTask(Api.Google, file)
    .SetGoogleUploadMetadata(metadata)
    .ExecuteAsync();

Debug.Log($"GCS uploaded: {gcsFile.Uri}");
```

---

## Provider Options

* **OpenAI**

  * `SetOpenAIUploadPurpose(OpenAITypes.UploadPurpose purpose)`
  * Common purposes: `Assistants` (default), `FineTuning`, `Batch`, `Vision`
  * Automatically sets `Api = Api.OpenAI`

* **Google**

  * `SetGoogleUploadMetadata(GoogleTypes.UploadMetadata metadata)`
  * Metadata includes name, description, MIME type, and visibility
  * Automatically sets `Api = Api.Google`

---

## Notes & Best Practices

* **Required parameter**: You must use one of the constructors:

  * `UploadFileTask(Api api, IFile file)`
  * `UploadFileTask(Api api, Texture2D image)`
  * `UploadFileTask(Api api, AudioClip clip)`

* **MIME type**: Always provide the correct MIME type for `IFile`.

* **Size limits**: Each provider has maximum file size and format restrictions. Use chunking or compression if needed.

* **Async**: `ExecuteAsync()` returns `UniTask<IUploadedFile>`. For progress, subscribe to higher-level `TaskManager` events.

* **Error handling**: Handle exceptions for network errors, invalid formats, or auth failures.

---

## Example: Upload and Use in Fine-Tuning

```csharp
IFile prompts = new LocalDiskFile("C:/work/prompts.jsonl", MimeType.ApplicationJson);

var uploaded = await new UploadFileTask(Api.OpenAI, prompts)
    .SetOpenAIUploadPurpose(OpenAITypes.UploadPurpose.FineTuning)
    .ExecuteAsync();

await new CreateFineTuneTask()
    .SetTrainingFile(uploaded.Id)
    .SetModel("gpt-4o-mini")
    .ExecuteAsync();
```
