# Creating assistants API

{% hint style="info" %}
This feature is only available in the Pro version.
{% endhint %}

**Step 1: Upload a File (if necessary)**

First, upload the file that the Assistant will use as a resource if necessary. You can skip this part if the Assistant won't use any resource.

```csharp
using Cysharp.Threading.Tasks;
using Glitch9.Apis.OpenAI;
using Glitch9.IO.Files;

public async UniTask<FileObject> UploadFileAsync(string filePath, UploadPurpose purpose)
{
    var fileUploadRequest = new FileUploadRequest.Builder()
        .SetFile(new FormFile(filePath))
        .SetPurpose(purpose)
        .Build();
        
    return await OpenAIClient.DefaultInstance.Upload(fileUploadRequest);
}
```

#### Step 2: Create the AssistantsAPIv2

After uploading the file, create an instance of the `AssistantsAPIv2` with the specified options:

#### Required Settings

* **Id**: Unique identifier for the AssistantsAPIv2 instance.
* **Model**: The GPT model to be used by the tool. Defaults to GPT-3.5 Turbo.
* **Name**: The name of the assistant.
* **Description**: The description of the assistant's functionality.
* **Instructions**: Guide the personality and define the goals of the Assistant.

#### **Optional Settings**

* **Tools**: Provide the Assistant with access to up to 128 tools.
* **ToolResources**: Give tools like code\_interpreter and file\_search access to files.
* **ForcedTool**: Forces the AssistantsAPI to use the specified tool on every request.
* **Metadata**: Gets or sets the metadata associated with this assistant.
* **Temperature**: The sampling temperature to use, between 0 and 2. Higher values make the output more random, while lower values make it more focused and deterministic.
* **TopP**: The nucleus sampling parameter, where the model considers the results of the tokens with top\_p probability mass.
* **ResponseFormat**: The response format for this assistant. Defaults to auto.
* **Stream**: Indicates whether the assistant should stream responses. Defaults to false.
* **MinTokenRequirementPerRequest**: The minimum number of tokens required per request.
* **MaxRequestLength**: The maximum number of characters the assistant's responses can contain. Use -1 for no limit.
* **AssistantFetchCount**: The maximum number of assistants to fetch when finding existing assistants on OpenAI server.
* **InitialDelayForRunStatusCheckSec**: The initial delay in seconds before checking the run status for the first time.
* **RecurringRunStatusCheckIntervalSec**: The recurring interval in seconds for checking the run status.
* **RunOperationTimeoutSec**: The timeout in seconds for the run operation.
* **Client**: The custom client to use for the assistant's operations.
* **OnTokensValidated**: Event handler that is called when the tokens are validated.
* **OnTokensConsumed**: Event handler that is called when tokens are consumed during the assistant's operation.
* **Exception**: Event handler that is called when an error occurs during the assistant's operation.
* **OnRunCancel**: Action that is called when a run is canceled.

```csharp
using System.Collections.Generic;
using Cysharp.Threading.Tasks;
using Glitch9.Apis.OpenAI;
using Glitch9.IO.Files;

public async UniTask<AssistantsAPIv2> CreateAsync(string filePath)
{
    // Upload the file and get the file object
    var fileObject = await UploadFileAsync(filePath, UploadPurpose.Assistants);

    // Define the assistant's options
    var options = new APIOptions
    {
        Id = "data_visualizer",
        Name = "Data Visualizer",
        Description = "You are great at creating beautiful data visualizations...",
        Instructions = "Analyze data present in .csv files, understand trends," +
                       "and come up with data visualizations relevant to those trends." +
                       "Share a brief text summary of the trends observed.",
        Model = GPTModel.GPT4o,
        Tools = new ToolCall[]
        {
            new CodeInterpreterCall(),            
        },
        ToolResources = new ToolResources
        {
            CodeInterpreter = new ToolResource
            {
                FileIds = new List<string> { fileObject.Id }
            }
        },   
    };

    // Create and initialize the AssistantsAPIv2 instance
    var api = new AssistantsAPIv2(options);
    await api.InitializeAsync();

    return api;
}
```

#### Step 3. Using the Assistant

```csharp
using System.Collections.Generic;
using Cysharp.Threading.Tasks;
using Glitch9.Apis.OpenAI;
using Glitch9.IO.Files;

public async UniTask RunAsync()
{
    // Create the Assistant
    var api = await CreateAssistantAsync("path/to/your/revenue-forecast.csv");
    
    // Create an attachment file
    var attachmentFile = new AttachmentFile("file-BK7bzQj3FfZFXr7DbL6xJwfo", AttachmentTarget.CodeInterpreter);
    
    // Create a thread with messages
    var message = new Message
    {
        Role = "user",
        Content = "Create 3 data visualizations based on the trends in this file.",
        Attachments = new List<Attachment>
        {
            new Attachment(attachmentFile),
        }
    };
    
    var result = await api.RequestAsync(message); 
}
```
