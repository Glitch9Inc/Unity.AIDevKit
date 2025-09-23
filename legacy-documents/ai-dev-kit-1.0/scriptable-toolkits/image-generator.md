# Image Generator

The `ImageGenerator` toolkit is a powerful feature of the **OpenAI with unity** asset that allows developers to generate images from textual descriptions within Unity, harnessing the capabilities of OpenAI's DALL-E models.

### Step 1: Creating an ImageGenerator Instance

1. Navigate to the Unity Editor's top menu and select `Assets > Create > Glitch9/OpenAI/Toolkits > Image Generator`.
2. An instance of `ImageGenerator` will be created in your Project view. Click on this instance to access its settings in the Inspector window.

### Step 2: Configuring the ImageGenerator

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

1. **Model**: Choose between DALL-E 2 or DALL-E 3, depending on your image generation needs.
2. **Image Size**: Select the desired output size for your generated images.
3. **Image Quality**: Choose the quality level of the generated images.
4. **Image Style**: Pick a style that will influence the aesthetic of the generated images.
5. **Image File Path**: Define the path where the generated images will be saved.

### Step 3: Using ImageGenerator in Your Script

1. Assign the `ImageGenerator` instance to a script that will handle image generation requests.
2. Utilize the provided methods such as `CreateImages`, `EditImage`, and `CreateVariations` to generate images based on text prompts or edit existing ones.

```csharp
public ImageGenerator imageGenerator;

public async void GenerateImageFromPrompt(string textPrompt) {
    ImageCreateResult result = await imageGenerator.CreateImages(textPrompt, 1);
    // Handle the generated image(s)
}
```

### Step 4: Generating Images

1. To generate new images, call the `CreateImages` method with a text prompt and the number of images you wish to generate.
2. This method is asynchronous. Use `await` or `UniTask` to wait for the generation to complete.

### Step 5: Editing and Creating Variations of Images

1. Use the `EditImage` method to make alterations to an existing image based on a new prompt.
2. To explore different styles or variations of an image, use the `CreateVariations` method with the original image and desired number of variations.

### Step 6: Handling the Generated Images

* Once images are generated, you can retrieve them from the `results` list.
* Use these images within your Unity scene, or provide functionality for the user to save or export them.

### Step 7: Customizing and Error Handling

* Adjust the model, size, quality, and style parameters based on user input or predefined settings to customize the image generation process.
* Implement error handling to catch any issues that may occur during the image generation process, such as invalid prompts or API errors.

### Step 8: Cleanup and Resource Management

* After generating images, you may need to manage the memory used by the textures, especially if generating large or numerous images.
* Ensure that generated images are properly disposed of when no longer needed to free up resources.

### Best Practices

* Provide clear and descriptive prompts to improve the quality of the generated images.
* Be mindful of the image generation limits imposed by the OpenAI API.
* Regularly update the file path to avoid overwriting previous images.
