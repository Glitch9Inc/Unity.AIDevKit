---
description: List uploaded files from AI providers with provider-specific query filters
icon: list-tree
---
 
# ListFiles

returns [`IUploadedFile[]`](https://glitch9inc.github.io/AIDevKit/api/Glitch9.AIDevKit.IUploadedFile.html)

Fetch a collection of files stored at the provider (OpenAI / Google / ElevenLabs).
Optionally pass a provider-specific `IQuery` to filter, sort, and paginate.

---

## Basic Usage

```csharp
using Glitch9.AIDevKit;

IUploadedFile[] files = await Api.OpenAI.ListFiles()
    .ExecuteAsync();

foreach (var f in files)
{
    Debug.Log($"{f.Id}  {f.FileName}  {f.MimeType}  {f.ByteSize}B");
}
```

---

## With Query (Filtering / Sorting / Pagination)

`ListFiles` accepts an `IQuery`. Use the query type that matches the provider:

### OpenAI — `CursorQuery` (cursor-based)

```csharp
using Glitch9.IO.Networking.RESTApi;

var q = new CursorQuery
{
    Limit = 50,
    Order = SortOrder.Descending
};

IUploadedFile[] page1 = await Api.OpenAI.ListFiles(q).ExecuteAsync();

// fetch next page using last item’s ID as cursor
q.After = page1.Length > 0 ? page1[^1].Id : null;
IUploadedFile[] page2 = await Api.OpenAI.ListFiles(q).ExecuteAsync();
```

---

### Google Gemini — `TokenQuery` (page-token based)

```csharp
using Glitch9.IO.Networking.RESTApi;

var q = new TokenQuery
{
    PageSize = 50,
    IncludeArchived = false
};

IUploadedFile[] first = await Api.Google.ListFiles(q).ExecuteAsync();

// next page
q.PageToken = /* response.nextPageToken from previous result */;
IUploadedFile[] second = await Api.Google.ListFiles(q).ExecuteAsync();
```

---

### ElevenLabs — `ElevenLabsQuery` (rich filters)

```csharp
using Glitch9.AIDevKit.ElevenLabs;

var q = new ElevenLabsQuery
{
    PageSize = 50,
    Search = "narration",
    Sort = "created_at_unix",
    SortDirection = SortDirection.Descending,
    Featured = true
};

IUploadedFile[] voices = await Api.ElevenLabs.ListFiles(q).ExecuteAsync();
```

---

## Notes & Best Practices

* **Pick the correct query** type for the provider.
* **Paginate server-side** with cursors or tokens.
* **Use limits** (`Limit`, `PageSize`) to reduce payloads.
* **Provider differences**:

  * OpenAI → cursor-based
  * Google Gemini → token-based
  * ElevenLabs → voice-specific filters
