---
description: Download a previously uploaded file from AI providers using its file ID
icon: arrow-down-to-bracket
---

# Download File

returns [`IUploadedFile`](https://glitch9inc.github.io/DocFx.AIDevKit/api/Glitch9.AIDevKit.IUploadedFile.html)

Retrieve a file that was previously uploaded to an AI provider.
Supports **OpenAI** and **Google**, returning a normalized `IUploadedFile` object with metadata and access details.

---

## Basic Usage (Download by File ID)

```csharp
using UnityEngine;
using Glitch9.AIDevKit;

// File ID returned from a previous UploadFileTask
string fileId = "file-abc123";

IUploadedFile file = await Api.OpenAI.DownloadFile(fileId)
    .ExecuteAsync();

Debug.Log($"Downloaded: {file.Id} ({file.FileName}, {file.SizeBytes} bytes)");
```

---

## Google Download

```csharp
string gcsFileId = "gcs-xyz456";

IUploadedFile gcsFile = await Api.Google.DownloadFile(gcsFileId)
    .ExecuteAsync();

Debug.Log($"Downloaded from GCS: {gcsFile.Uri}");
```
