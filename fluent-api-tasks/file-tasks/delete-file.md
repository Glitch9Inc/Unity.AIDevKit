---
description: Delete a previously uploaded file from AI providers  by file ID
icon: trash-can-arrow-up
---
 
# DeleteFile

returns `bool`

Remove a file that was previously uploaded to an AI provider.
Supports **OpenAI** and **Google**. Returns `true` on success.

---

## Basic Usage

```csharp
using Glitch9.AIDevKit;

string fileId = "file-abc123";

bool ok = await Api.OpenAI.DeleteFile(fileId)
    .ExecuteAsync();

if (ok) Debug.Log("File deleted.");
```

## Example: Safe Delete with Existence Check

```csharp
var list = await Api.OpenAI.ListFiles().ExecuteAsync();
bool exists = list.Any(f => f.Id == "file-abc123"); // assuming IUploadedFile.Id exists in your model

if (exists)
{
    bool ok = await Api.OpenAI.DeleteFile("file-abc123").ExecuteAsync();
    Debug.Log(ok ? "Deleted." : "Delete failed.");
}
```
