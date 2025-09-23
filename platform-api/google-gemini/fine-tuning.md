# ⚙️ Fine-tuning

### Create tuned model <a href="#create_tuned_model" id="create_tuned_model"></a>

To create a tuned model, you need to pass your dataset to the model in the `genai.create_tuned_model` method. You can do this by directly defining the input and output values in the call or importing from a file into a dataframe to pass to the method.

For this example, you will tune a model to generate the next number in the sequence. For example, if the input is `1`, the model should output `2`. If the input is `one hundred`, the output should be `one hundred one`.

{% tabs %}
{% tab title="C#" %}
```csharp
var baseModel = (await GenerativeAI.DefaultInstance.Models.List())
    .FirstOrDefault(m => m.SupportedGenerationMethods.Contains("createTunedModel"));

Debug.Log(baseModel.ToString());

/* log
Model(name='models/gemini-1.0-pro-001',
      base_model_id='',
      version='001',
      display_name='Gemini 1.0 Pro',
      description=('The best model for scaling across a wide range of tasks. This is a stable '
                   'model that supports tuning.'),
      input_token_limit=30720,
      output_token_limit=2048,
      supported_generation_methods=['generateContent', 'countTokens', 'createTunedModel'],
      temperature=0.9,
      top_p=1.0,
      top_k=1)
*/
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
base_model = [
    m for m in genai.list_models()
    if "createTunedModel" in m.supported_generation_methods][0]
base_model
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="C#" %}
```csharp
var name = $"generate-num-{UnityEngine.Random.Range(0, 10000)}";

var tunedModelRequest = new TunedModelRequest.Builder()
    .SetSourceModel(baseModel.Name)
    .SetTrainingData(
        new TuningExample("1", "2"),
        new TuningExample("3", "4"),
        new TuningExample("-3", "-2"),
        new TuningExample("twenty two", "twenty three"),
        new TuningExample("two hundred", "two hundred one"),
        new TuningExample("ninety nine", "one hundred"),
        new TuningExample("8", "9"),
        new TuningExample("-98", "-97"),
        new TuningExample("1,000", "1,001"),
        new TuningExample("10,100,000", "10,100,001"),
        new TuningExample("thirteen", "fourteen"),
        new TuningExample("eighty", "eighty one"),
        new TuningExample("one", "two"),
        new TuningExample("three", "four"),
        new TuningExample("seven", "eight"))
    .SetName(name)
    .SetEpochCount(100)
    .SetBatchSize(4)
    .SetLearningRate(0.001f)
    .Build();

var operation = 
    await GenerativeAI.DefaultInstance.TunedModels.Create(tunedModelRequest);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
import random

name = f'generate-num-{random.randint(0,10000)}'
operation = genai.create_tuned_model(
    # You can use a tuned model here too. Set `source_model="tunedModels/..."`
    source_model=base_model.name,
    training_data=[
        {
             'text_input': '1',
             'output': '2',
        },{
             'text_input': '3',
             'output': '4',
        },{
             'text_input': '-3',
             'output': '-2',
        },{
             'text_input': 'twenty two',
             'output': 'twenty three',
        },{
             'text_input': 'two hundred',
             'output': 'two hundred one',
        },{
             'text_input': 'ninety nine',
             'output': 'one hundred',
        },{
             'text_input': '8',
             'output': '9',
        },{
             'text_input': '-98',
             'output': '-97',
        },{
             'text_input': '1,000',
             'output': '1,001',
        },{
             'text_input': '10,100,000',
             'output': '10,100,001',
        },{
             'text_input': 'thirteen',
             'output': 'fourteen',
        },{
             'text_input': 'eighty',
             'output': 'eighty one',
        },{
             'text_input': 'one',
             'output': 'two',
        },{
             'text_input': 'three',
             'output': 'four',
        },{
             'text_input': 'seven',
             'output': 'eight',
        }
    ],
    id = name,
    epoch_count = 100,
    batch_size=4,
    learning_rate=0.001,
)
```
{% endtab %}
{% endtabs %}

Your tuned model is immediately added to the list of tuned models, but its status is set to "creating" while the model is tuned.

{% tabs %}
{% tab title="C#" %}
```csharp
var model = 
    await GenerativeAI.DefaultInstance.TunedModels.Get(name);

Debug.Log(model.ToString());

/* log
TunedModel(name='tunedModels/generate-num-2946',
   source_model='models/gemini-1.0-pro-001',
   base_model='models/gemini-1.0-pro-001',
   display_name='',
   description='',
   temperature=0.9,
   top_p=1.0,
   top_k=1,
   state=<State.CREATING: 1>,
   create_time=datetime.datetime(2024, 2, 21, 20, 4, 16, 448050, tzinfo=datetime.timezone.utc),
   update_time=datetime.datetime(2024, 2, 21, 20, 4, 16, 448050, tzinfo=datetime.timezone.utc),
   tuning_task=TuningTask(start_time=datetime.datetime(2024, 2, 21, 20, 4, 16, 890698, tzinfo=datetime.timezone.utc),
                          complete_time=None,
                          snapshots=[],
                          hyperparameters=Hyperparameters(epoch_count=100,
                                                          batch_size=4,
                                                          learning_rate=0.001)))
 */
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
base_model = [

model = genai.get_tuned_model(f'tunedModels/{name}')

model

TunedModel(name='tunedModels/generate-num-2946',
           source_model='models/gemini-1.0-pro-001',
           base_model='models/gemini-1.0-pro-001',
           display_name='',
           description='',
           temperature=0.9,
           top_p=1.0,
           top_k=1,
           state=<State.CREATING: 1>,
           create_time=datetime.datetime(2024, 2, 21, 20, 4, 16, 448050, tzinfo=datetime.timezone.utc),
           update_time=datetime.datetime(2024, 2, 21, 20, 4, 16, 448050, tzinfo=datetime.timezone.utc),
           tuning_task=TuningTask(start_time=datetime.datetime(2024, 2, 21, 20, 4, 16, 890698, tzinfo=datetime.timezone.utc),
                                  complete_time=None,
                                  snapshots=[],
                                  hyperparameters=Hyperparameters(epoch_count=100,
                                                                  batch_size=4,
                                                                  learning_rate=0.001)))
```
{% endtab %}
{% endtabs %}

### Evaluate your model <a href="#evaluate_your_model" id="evaluate_your_model"></a>

You can use the `GenerativeAI.DefaultInstance.TunedModels.GenerateText` method and specify the name of your model to test your model performance.

{% tabs %}
{% tab title="C#" %}
```csharp
var model = new GenerativeModel("name");

var result = await model.GenerateContentAsync("55");
Debug.Log(result.Text);

// '56'

result = await model.GenerateContentAsync("123455");
Debug.Log(result.Text);

// '123456'

result = await model.GenerateContentAsync("four");
Debug.Log(result.Text);

// 'five'

result = await model.GenerateContentAsync("quatre"); // French 4
Debug.Log(result.Text); // French 5 is "cinq"

// 'cinq'

result = await model.GenerateContentAsync("III"); // Roman numeral 3
Debug.Log(result.Text); // Roman numeral 4 is IV

// 'IV'

result = await model.GenerateContentAsync("七"); // Japanese 7
Debug.Log(result.Text); // Japanese 8 is 八!

// '八'
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
base_model = [

model = genai.GenerativeModel(model_name=f'tunedModels/{name}')

result = model.generate_content('55')
result.text

'56'

result = model.generate_content('123455')
result.text

'123456'

result = model.generate_content('four')
result.text

'five'

result = model.generate_content('quatre') # French 4
result.text                               # French 5 is "cinq"

'cinq'

result = model.generate_content('III')    # Roman numeral 3
result.text                               # Roman numeral 4 is IV

'IV'

result = model.generate_content('七')  # Japanese 7
result.text                            # Japanese 8 is 八!

'八'
```
{% endtab %}
{% endtabs %}

It really seems to have picked up the task despite the limited examples, but "next" is a relatively simple concept, see the [tuning guide](https://ai.google.dev/gemini-api/docs/model-tuning) for more guidance on improving performance.

### Update the description <a href="#update_the_description" id="update_the_description"></a>

You can update the description of your tuned model any time using the `GenerativeAI.DefaultInstance.TunedModels.Patch` method.

{% tabs %}
{% tab title="C#" %}
```csharp
await GenerativeAI.DefaultInstance.TunedModels.Patch(
    "name", 
    new UpdateMask("description", "This is my model."));

var model = await GenerativeAI.DefaultInstance.TunedModels.Get("name");

Debug.Log(model.Data.Description);

// log: "This is my model."
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
genai.update_tuned_model(f'tunedModels/{name}', {"description":"This is my model."});

model = genai.get_tuned_model(f'tunedModels/{name}')

model.description

'This is my model.'
```
{% endtab %}
{% endtabs %}

### Delete the model <a href="#delete_the_model" id="delete_the_model"></a>

You can clean up your tuned model list by deleting models you no longer need. Use the `GenerativeAI.DefaultInstance.TunedModels.Delete` method to delete a model. If you canceled any tuning jobs, you may want to delete those as their performance may be unpredictable.

{% tabs %}
{% tab title="C#" %}
```csharp
await GenerativeAI.DefaultInstance.TunedModels.Delete("name");
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
genai.delete_tuned_model(f'tunedModels/{name}')
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
If the model no longer exists, it will return an error:

\<class 'google.api\_core.exceptions.NotFound'>: 404 Tuned model tunedModels/generate-num-2946 does not exist.
{% endhint %}

{% embed url="https://ai.google.dev/gemini-api/docs/model-tuning" %}
Google official document
{% endembed %}
