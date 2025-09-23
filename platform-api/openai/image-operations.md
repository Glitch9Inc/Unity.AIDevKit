# 🖼️ Image operations

To integrate OpenAI's image generation, editing, and variation capabilities into your Unity project, you'll primarily interact with three types of requests using the **OpenAI with unity** asset. These requests are designed to leverage OpenAI's advanced image models, such as DALL·E, to create, modify, and generate variations of images based on textual prompts or existing images.

For a deep dive into the technical details and options available for these requests, refer to the [Images API Reference](https://platform.openai.com/docs/api-reference/images).

### Image Operations Overview:

1. **Image Creation**: Generate new images from textual prompts.
2. **Image Editing**: Modify existing images based on textual instructions.
3. **Image Variations**: Create different variations of an input image.

### Sample Code for Image Requests:

#### **1. Image Creation Request:**

Generate new images from a textual description using `ImageCreationRequest`.

{% tabs %}
{% tab title="C#" %}
```javascript
var request = new ImageCreationRequest.Builder()
    .SetModel(ImageModel.DallE3)
    .SetPrompt("A cute baby sea otter")
    .SetN(1) // Number of images to generate
    .SetSize(ImageSize._1024x1024)
    .Build();

var result = await request.ExecuteAsync();

Texture2D[] images = result.Textures;
// Now you can use these textures in your Unity project
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

client.images.generate(
  model="dall-e-3",
  prompt="A cute baby sea otter",
  n=1,
  size="1024x1024"
)
```
{% endtab %}
{% endtabs %}

#### **2. Image Editing Request:**

Edit an existing image based on a provided prompt. You'll need to supply the image as a `FormFile` (for API requests) or a `Texture2D` object (within Unity).

Using Texture2D from Resources folder.

{% tabs %}
{% tab title="C#" %}
```csharp
// Using Texture2D
var otter = Resources.Load<Texture2D>("otter");
var mask = Resources.Load<Texture2D>("mask");

var request = new ImageEditRequest.Builder()
    .SetImage(otter)
    .SetMask(mask)
    .SetPrompt("A cute baby sea otter wearing a beret")
    .SetN(2)
    .SetSize(ImageSize._1024x1024)
    .Build();

var result = await request.ExecuteAsync();

Texture2D[] images = result.Textures;
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

client.images.edit(
  image=open("otter.png", "rb"),
  mask=open("mask.png", "rb"),
  prompt="A cute baby sea otter wearing a beret",
  n=2,
  size="1024x1024"
)
```
{% endtab %}
{% endtabs %}

Using FormFile with file paths.

{% tabs %}
{% tab title="C#" %}
```csharp
// Using FormFile
var otterFile = new FormFile(path to otter.png);
var maskFile = new FormFile(path to mask.png);

var request = new ImageEditRequest.Builder()
    .SetImage(otterFile)
    .SetMask(maskFile)
    .SetPrompt("A cute baby sea otter wearing a beret")
    .SetN(2)
    .SetSize(ImageSize._1024x1024)
    .Build();
    
var result = await request.ExecuteAsync();

Texture2D[] images = result.Textures;
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

client.images.edit(
  image=open("otter.png", "rb"),
  mask=open("mask.png", "rb"),
  prompt="A cute baby sea otter wearing a beret",
  n=2,
  size="1024x1024"
)
```
{% endtab %}
{% endtabs %}

#### **3. Image Variation Request:**

Generate variations of an input image. Similar to image editing, the image should be provided as a `FormFile` or `Texture2D`.

{% tabs %}
{% tab title="C#" %}
```csharp
var image = Resources.Load<Texture2D>("image_edit_original");

var request = new ImageVariationRequest.Builder()
    .SetImage(image)
    .SetN(2)
    .SetSize(ImageSize._1024x1024)
    .Build();

var result = await request.ExecuteAsync();

Texture2D[] images = result.Textures;
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
from openai import OpenAI
client = OpenAI()

response = client.images.create_variation(
  image=open("image_edit_original.png", "rb"),
  n=2,
  size="1024x1024"
)
```
{% endtab %}
{% endtabs %}

