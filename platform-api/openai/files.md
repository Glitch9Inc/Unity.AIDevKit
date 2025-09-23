---
description: >-
  Files are used to upload documents that can be used with features like
  Assistants, Fine-tuning, and Batch API.
---

# 💾 Files

### [Upload file](https://platform.openai.com/docs/api-reference/files/create)

Upload a file that can be used across various endpoints. Individual files can be up to 512 MB, and the size of all files uploaded by one organization can be up to 100 GB.

{% tabs %}
{% tab title="Assistants API" %}
* Supports files up to 2 million tokens and of specific file types.
* See the [Assistants Tools guide](https://platform.openai.com/docs/assistants/tools) for details.
{% endtab %}

{% tab title="Fine-tuning API " %}
* Only supports `.jsonl` file.
* The input also has certain required formats for fine-tuning [chat](https://platform.openai.com/docs/api-reference/fine-tuning/chat-input) or [completions](https://platform.openai.com/docs/api-reference/fine-tuning/completions-input) models.
{% endtab %}

{% tab title="Batch API" %}
* Only supports `.jsonl` file.
* Up to 100 MB in size.
* The input also has a specific required [format](https://platform.openai.com/docs/api-reference/batch/request-input).
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="C#" %}
```csharp
var file = new FormFile("path/to/mydata.jsonl");

var request = new FileUploadRequest.Builder()
    .SetFile(file)
    .SetPurpose(UploadPurpose.FineTune)
    .Build();

var result = await OpenAI.DefaultInstance.Files.Upload(request);

Debug.Log(result.Id);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

client.files.create(
  file=open("mydata.jsonl", "rb"),
  purpose="fine-tune"
)
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Please [contact OpenAI](https://help.openai.com/) if you need to increase these storage limits.
{% endhint %}

***

### [List files](https://platform.openai.com/docs/api-reference/files/list)

Returns a list of files that belong to the user's organization.

{% tabs %}
{% tab title="C#" %}
```csharp
var fileList = await OpenAI.DefaultInstance.Files.List();

foreach (var fileData in fileList)
{
    Debug.Log(fileData.Id);
    Debug.Log(fileData.CreatedAt);
    Debug.Log(fileData.Purpose);
}
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

client.files.list()
```
{% endtab %}
{% endtabs %}

***

### [Retrieve file](https://platform.openai.com/docs/api-reference/files/retrieve)

Returns information about a specific file.

{% tabs %}
{% tab title="C#" %}
```csharp
var fileData = await OpenAI.DefaultInstance.Files.Retrieve("file-abc123");

Debug.Log(fileData.Id);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

client.files.retrieve("file-abc123")
```
{% endtab %}
{% endtabs %}

***

### [Delete file](https://platform.openai.com/docs/api-reference/files/delete)

Delete a file.

{% tabs %}
{% tab title="C#" %}
```csharp
bool deleted = await OpenAI.DefaultInstance.Files.Delete("file-abc123");

if (deleted)
{
    Debug.Log("File deleted successfully");
}
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

client.files.delete("file-abc123")
```
{% endtab %}
{% endtabs %}

***

### [Retrieve file content](https://platform.openai.com/docs/api-reference/files/retrieve-contents)

Returns the contents of the specified file.

{% tabs %}
{% tab title="C#" %}
```csharp
var content = await OpenAI.DefaultInstance.Files.RetrieveFileContent("file-abc123");

Debug.Log(content);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

content = client.files.content("file-abc123")
```
{% endtab %}
{% endtabs %}
