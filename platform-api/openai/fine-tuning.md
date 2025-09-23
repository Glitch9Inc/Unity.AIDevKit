# ⚙️ Fine-tuning

To enhance the capabilities of OpenAI's models within your Unity project, you can leverage the fine-tuning feature to tailor the models more closely to your specific needs or data. Fine-tuning allows you to train an existing OpenAI model on your own dataset, resulting in a custom model that can offer better performance for your particular application, whether it's generating more relevant text, recognizing specific styles in image generation, or understanding particular dialects in speech recognition.

For comprehensive details on the process, parameters, and management of fine-tuned models, refer to the [Fine-Tuning API Reference](https://platform.openai.com/docs/guides/fine-tuning).

### Fine-Tuning Operations Overview:

* **Model Training**: Adjust an existing OpenAI model by training it on a custom dataset to improve its relevance to your specific requirements.
* **Model Management**: After fine-tuning, manage your custom models, including deployment, monitoring, and versioning, directly within your Unity project.

### Sample Code for Fine-Tuning Requests:

#### **1. Model Training Request:**

Initiate the fine-tuning process by specifying your dataset and the base model to refine.

```csharp
FineTuningRequest request = new FineTuningRequest.Builder()
    .SetTrainingFile("path/to/your/training/data.csv") // Specify the dataset
    .SetBaseModel(FineTuningModel.Gpt3_5Turbo) // Specify the base model for fine-tuning
    .Build();

FineTuningJob fineTuningJob = await request.ExecuteAsync();

Debug.Log($"Fine-tuning started: Job ID {fineTuningJob.Id}");
```

This example illustrates how to start a fine-tuning job. You'll need to prepare a dataset that the model will learn from, which typically involves creating a .csv file with examples of inputs and desired outputs.

#### **2. Model Management Request:**

After fine-tuning, you can list, retrieve, or delete your custom models, ensuring you have the right version deployed for your application's needs.

```csharp
// List your fine-tuned models
ModelObject[] models = await OpenAiClient.DefaultInstance.ListModels();

foreach (var model in models)
{
    Debug.Log($"Model ID: {model.Id}, Status: {model.Status}");
}

// Retrieve a specific fine-tuned model
string modelId = "your-model-id";
ModelObject myModel = await OpenAiClient.DefaultInstance.RetrieveModel(modelId);

Debug.Log($"Retrieved Model: {myModel.Id}");

// Delete a fine-tuned model (if no longer needed)
bool deletionSuccess = await OpenAiClient.DefaultInstance.DeleteModel(modelId);

if (deletionSuccess)
{
    Debug.Log("Model deleted successfully.");
}
```

#### Implementing Fine-Tuning in Unity using OpenAI:

To utilize fine-tuning within your Unity project:

1. **Prepare Your Dataset**: Create a dataset that represents the kind of output you expect from the model. This involves pairing input text with expected output text in a format understandable by the fine-tuning process.
2. **Initiate Fine-Tuning**: Use the `FineTuningRequest` to configure and start the fine-tuning process with your dataset and chosen base model.
3. **Manage Custom Models**: After fine-tuning, utilize the model management functionalities to deploy, monitor, and maintain your custom models within your Unity application.

By fine-tuning OpenAI models to better fit your specific use cases, you unlock enhanced performance and more relevant outputs from the AI, driving greater value and engagement in your Unity applications. Whether for generating unique game content, tailoring chatbot interactions, or improving image generation, fine-tuning provides a powerful tool for customizing AI behaviors to your needs.
