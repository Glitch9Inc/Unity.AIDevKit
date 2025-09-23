---
description: Upload files, images, or audio clips to AI providers (OpenAI / Google) with metadata support
icon: up-from-bracket
--- 

# UploadFile

returns [`IUploadedFile`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.IUploadedFile.html)

Upload text files, images (`Texture2D`), or audio clips (`AudioClip`) via **multipart form-data** to AI providers.

> All uploads are sent as `MimeType.MultipartFormData`. The result is normalized into `IUploadedFile`.

---

## Basic Usage (local path/file)

```csharp
using Glitch9.AIDevKit;
using Glitch9.IO.Files;

// Use any IFile implementation. LocalFile covers raw bytes on disk.
IFile file = new LocalFile("C:/datasets/train.jsonl", mimeType: MimeType.ApplicationJson);

IUploadedFile uploaded = await Api.OpenAI
    .UploadFile(file)                                   // <- convenience form
    .SetOpenAIUploadPurpose(OpenAITypes.UploadPurpose.Assistants) // default is Assistants
    .ExecuteAsync();

DevLog.Info($"Uploaded: {uploaded.Id} ({uploaded.FileName}, {uploaded.ByteSize} bytes)");
```

---

## Upload a Unity `Texture2D`

```csharp
using Glitch9.AIDevKit;
using Glitch9.IO.Files;
using UnityEngine;

Texture2D tex = /* load or generate at runtime */;

// File<T> automatically infers MIME via MIMETypeUtil and encodes Texture2D -> PNG on upload.
IFile imageFile = new File<Texture2D>(tex, "C:/tmp/snapshot.png");

IUploadedFile img = await Api.OpenAI
    .UploadFile(imageFile)
    .SetOpenAIUploadPurpose(OpenAITypes.UploadPurpose.Assistants)
    .ExecuteAsync();

DevLog.Info($"Image uploaded: {img.Id}");
```

---

## Upload an `AudioClip`

```csharp
using Glitch9.AIDevKit;
using Glitch9.IO.Files;
using UnityEngine;

AudioClip clip = /* from mic or asset */;

// File<T> encodes AudioClip -> WAV on upload.
IFile audioFile = new File<AudioClip>(clip, "C:/tmp/recording.wav");

IUploadedFile aud = await Api.OpenAI
    .UploadFile(audioFile)
    .SetOpenAIUploadPurpose(OpenAITypes.UploadPurpose.Assistants)
    .ExecuteAsync();

DevLog.Info($"Audio uploaded: {aud.Id}");
```

---

## Upload from in-memory bytes (no asset)

```csharp
using Glitch9.AIDevKit;
using Glitch9.IO.Files;

byte[] payload = System.Text.Encoding.UTF8.GetBytes("{\"hello\":\"world\"}");
IFile mem = new LocalFile(payload, "C:/virtual/data.json", mimeType: MimeType.ApplicationJson);

var up = await Api.OpenAI
    .UploadFile(mem)
    .SetOpenAIUploadPurpose(OpenAITypes.UploadPurpose.Batch)
    .ExecuteAsync();
```

---

## Google upload (with metadata)

```csharp
using Glitch9.AIDevKit;
using Glitch9.IO.Files;

var metadata = new GoogleTypes.UploadMetadata
{
    Name = "my-dataset.jsonl",
    Description = "Training corpus v3",
    MimeType = MimeType.ApplicationJson
};

IFile file = new LocalFile("C:/datasets/train.jsonl", mimeType: MimeType.ApplicationJson);

IUploadedFile gcs = await Api.Google
    .UploadFile(file)               // provider is fixed by the Api facade
    .SetGoogleUploadMetadata(metadata)
    .ExecuteAsync();

DevLog.Info($"GCS uploaded: {gcs.Uri}");
```

---

## Provider options

* **OpenAI**

  * `SetOpenAIUploadPurpose(OpenAITypes.UploadPurpose purpose)`
  * Common: `Assistants` (default), `FineTuning`, `Batch`, `Vision`
  * Convenience call: `Api.OpenAI.UploadFile(IFile)`

* **Google**

  * `SetGoogleUploadMetadata(GoogleTypes.UploadMetadata metadata)`
  * Convenience call: `Api.Google.UploadFile(IFile)`

---

## Notes & best practices

* **Constructor inputs** (internals): You can create uploads from

  * `IFile` (e.g., `LocalFile`, `File<Texture2D>`, `File<AudioClip>`)
  * `Texture2D` / `AudioClip` via `File<T>` which handles encoding (PNG/WAV)
  * In-memory bytes via `new LocalFile(byte[], path, mimeType: …)`
* **MIME**:

  * `File<T>` auto-infers MIME via `MIMETypeUtil`.
  * `LocalFile` should be given an explicit `MimeType` when possible.
* **Asset loading**:

  * `File<T>` lazily loads/encodes assets; `EnsureAssetLoadedAsync()` if you need it preloaded.
* **Size limits**:

  * Providers enforce max size/format. Chunk or compress large payloads.
* **Async**:

  * `ExecuteAsync()` returns `UniTask<IUploadedFile>`. For progress, use your `TaskManager` hooks.
* **Error handling**:

  * Check `IFile.LastError` after reads and wrap uploads with `try/catch`.

---
