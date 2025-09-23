# Chat Streamer

The `ChatStreamer` toolkit enables real-time chat capabilities within Unity, powered by OpenAI's GPT models. It allows for dynamic text-based interactions that can be integrated into games or applications for a variety of purposes, including non-player character (NPC) dialogue, interactive storytelling, customer service bots, and more.

### Step 1: Creating a Chat Streamer Instance

1. Go to the Unity Editor menu and select `Assets > Create > Glitch9/OpenAI/Toolkits > Chat Streamer`.
2. A new instance of `ChatStreamer` will appear in your Project view. Select it to configure its properties in the Inspector window.

### Step 2: Configuring Chat Streamer

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

1. **Model**: Select the GPT model you wish to use for generating responses.
2. **Instruction**: Define an initial instruction or prompt that will guide the AI's conversation style or topic.
3. **Frequency Penalty**: Adjust this value to discourage the AI from repeating itself.
4. **Max Tokens**: Set the maximum length of the generated response.
5. **Temperature**: Control the randomness of the response. A lower temperature results in more predictable responses.

### Step 3: Initializing Chat Streamer

1. Attach the `ChatStreamer` instance to a script or GameObject in your scene where the chat will be managed.
2. Call the `Initialize` method from your script to prepare the `ChatStreamer` for use. You can pass an `Action<string>` delegate to handle incoming messages.

```csharp
public ChatStreamer chatStreamer;

void Start() {
    chatStreamer.Initialize(OnMessageReceived);
}

void OnMessageReceived(string message) {
    // Handle the new message, such as updating the UI
}
```

### Step 4: Starting a Chat Session

1. To begin streaming chat messages, call the `EnterChat` method with the user's input prompt.
2. The `EnterChat` method is asynchronous. Use `await` or `UniTask` to wait for the method to complete and to receive the response.

```csharp
csharpCopy codepublic async void SendMessageToChat(string userInput) {
    ChatCompletionResult result = await chatStreamer.EnterChat(userInput);
    // The result contains the AI's response.
}
```

### Step 5: Handling Chat Responses

1. Use the `OnStream` event to update the chat interface with both the user's messages and the AI's responses.
2. Append each new message to the chat log, ensuring a continuous conversation flow.

### Step 6: Customizing the Chat Behavior

* Adjust the `Frequency Penalty`, `Max Tokens`, and `Temperature` settings to fine-tune the AI's responses based on the context of your application.
* Utilize the `SetFrequencyPenalty`, `SetMaxTokens`, and `SetTemperature` methods to change the conversation dynamics on the fly.

### Step 7: Saving Chat Logs

* If your application requires saving the chat history, access the `Messages` property to retrieve the current chat log.
* Serialize and save the messages to a file or database as needed.

### Step 8: Finalizing the Chat Session

* To end a chat session, you can simply stop sending prompts or provide a user interface option for the player to exit the conversation.
* Optionally, you can add logic to reset or clear the `ChatStreamer` instance for a new conversation.

### Best Practices

* Test different configurations to find the ideal chat behavior for your use case.
* Consider implementing rate-limiting to avoid overloading the user with responses.
* Always provide a user-friendly way to exit the chat or restart the conversation.
