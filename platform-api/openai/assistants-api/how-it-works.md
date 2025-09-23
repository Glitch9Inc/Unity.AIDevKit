# How it works

The Assistants API is designed to help developers build powerful AI assistants capable of performing a variety of tasks.

Assistants can call OpenAI’s models with specific instructions to tune their personality and capabilities. They can access multiple tools in parallel, which can be OpenAI-hosted tools like <mark style="color:blue;">**code\_interpreter**</mark> and <mark style="color:blue;">**file\_search**</mark>, or [<mark style="color:red;">**custom tools**</mark>](creating-custom-functions.md) you build and host via function calling. Assistants can also access persistent threads to store message history and manage context length. Furthermore, they can handle files in various formats for tasks that require file manipulation.

#### Understanding objects related to AssistantsAPI

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption><p>This image is from OpenAI's official document: <a href="https://platform.openai.com/docs/assistants/how-it-works/objects">https://platform.openai.com/docs/assistants/how-it-works/objects</a></p></figcaption></figure>

* **Assistant**: A purpose-built AI that uses OpenAI’s models and calls tools.
* **Thread**: Represents a conversation session between an Assistant and a user, storing messages and handling truncation.
* **Message**: A message created by an Assistant or a user, which can include text, images, and other files.
* **Run**: An invocation of an Assistant on a Thread, where the Assistant uses its configuration and the Thread’s messages to perform tasks.
* **Run Step**: A detailed list of steps taken by the Assistant during a Run, which includes calling tools and creating messages.

#### Context Window Management

The API automatically manages truncation to stay within the model's maximum context length. You can customize this by specifying `max_prompt_tokens` and `max_completion_tokens`.

#### Understanding Run Lifecycle

Run objects can have multiple statuses:

* <mark style="color:purple;">**RunStatus**</mark>**.Queued**: The run is waiting to start.
* <mark style="color:purple;">**RunStatus**</mark>**.InProgress**: The run is currently executing.
* <mark style="color:purple;">**RunStatus**</mark>**.Completed**: The run successfully completed.
* <mark style="color:purple;">**RunStatus**</mark>**.RequiresAction**: The run requires additional information.
* <mark style="color:purple;">**RunStatus**</mark>**.Expired**: The run expired due to time constraints.
* <mark style="color:purple;">**RunStatus**</mark>**.Cancelling**: The run is being cancelled.
* <mark style="color:purple;">**RunStatus**</mark>**.Cancelled**: The run was successfully cancelled.
* <mark style="color:purple;">**RunStatus**</mark>**.Failed**: The run failed due to an error.
* <mark style="color:purple;">**RunStatus**</mark>**.Incomplete**: The run ended due to token limits being reached.

#### Data Access Guidance

Assistants, Threads, Messages, and Vector Stores created via the API are scoped to the Project they are created in. Implement authorization and restrict API key access to ensure data security.

#### Example C# Code

```csharp
// Define your custom object for handling chat responses and media attachments.
public class Chat
{
    public ChatRole Role { get; set; }  // It always has to be ChatRole.User if it's an input message.
    public string Message { get; set; }  // Text message content.
    public List<OpenAIFile> Files { get; set; }  // Files attached to the message.
    public List<UnityImageFile> Images { get; set; }  // Images attached to the message.

    // Converts the EditorChat object into a MessageRequest for API processing.
    public MessageRequest ToMessageRequest()
    {
        MessageRequest.Builder builder = new MessageRequest.Builder();
        builder.SetPrompt(Message);
        if (Images != null && Images.Any())
            builder.SetImages(Images.ToArray());
        if (Files != null && Files.Any())
            builder.SetFiles(Files.ToArray());
        return builder.Build();
    }
}

// Integration class for OpenAI's AssistantsAPIv2 using Unity.
using Cysharp.Threading.Tasks;
using Glitch9.Apis.OpenAI.AssistantsAPI;
using System.Collections.Generic;
using System.Linq;

namespace Glitch9.Apis.OpenAI
{
    public class ChatGPT
    {
        private static class DefaultValues
        {
            internal const GPTModel MODEL = GPTModel.GPT4o;  // Default GPT model used.
        }

        private static readonly AssistantOptions k_Options = new()
        {
            Id = "chat_gpt",
            Model = DefaultValues.MODEL,
            Name = "ChatGPT",
            Description = "A chatbot that can help you with anything you need.",
            Instructions = "You are a chatbot designed to output. You can help users with any questions they have.",
            ResponseFormat = ChatFormat.Text  // The format of chat responses.
        };

        public AssistantsAPIv2 Api;

        public EditorChatGPT()
        {
            // Initialize the API with predefined options.
            Api = new AssistantsAPIv2(k_Options);
            Api.InitializeAsync().Forget();
        }

        // Processes a chat message and manages tool execution if needed.
        public async UniTask<Chat> SubmitMessageAsync(Chat inputChat)
        {
            var msgRequest = inputChat.ToMessageRequest();
            var result = await Api.RequestAsync(msgRequest);

            if (result == null || result.IsFailure)
                return new Chat { Role = ChatRole.Assistant, Message = "There was an error processing your request." };

            // Return the result as a new EditorChat instance.
            return new Chat { Role = ChatRole.Assistant, Message = result.GetResult() };
        }

        // Additional methods to manage threads and responses can be implemented here.
    }
}

```

Now you have explored how Assistants work, the next step is to explore Assistant Tools, covering topics like Function calling, File Search, and Code Interpreter.
