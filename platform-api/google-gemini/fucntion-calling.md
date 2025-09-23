# ▶️ Fucntion calling

### Define an API function <a href="#define-api-function" id="define-api-function"></a>

Create a function that makes an API request. This function should be defined within the code of your application, but could call services or APIs outside of your application. The Gemini API does _not_ call this function directly, so you can control how and when this function is executed through your application code. For demonstration purposes, this tutorial defines a mock API function that just returns the requested lighting values:

{% tabs %}
{% tab title="C#" %}
```csharp
// Set the brightness and color temperature of a room light. (mock API).
// brightness: Light level from 0 to 100. Zero is off and 100 is full brightness
// colorTemp: Color temperature of the light fixture, which can be `daylight`, `cool` or `warm`.
public Dictionary<string, object> SetLightValues(int brightness, string colorTemp)
{
    // Implement the real API call here

    // Return the set brightness and color temperature.
    return new Dictionary<string, object>
    {
        { "brightness", brightness },
        { "colorTemperature", colorTemp }
    };
}
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
def set_light_values(brightness, color_temp):
    """Set the brightness and color temperature of a room light. (mock API).

    Args:
        brightness: Light level from 0 to 100. Zero is off and 100 is full brightness
        color_temp: Color temperature of the light fixture, which can be `daylight`, `cool` or `warm`.

    Returns:
        A dictionary containing the set brightness and color temperature.
    """
    return {
        "brightness": brightness,
        "colorTemperature": color_temp
    }
```
{% endtab %}
{% endtabs %}

When you create a function to be used in a function call by the model, you should include as much detail as possible in the function and parameter descriptions. The generative model uses this information to determine which function to select and how to provide values for the parameters in the function call.

{% hint style="warning" %}
For any production application, you should validate the data being passed to the API function from the model _before_ executing the function.
{% endhint %}

{% hint style="info" %}
For programing languages other than Python, you must create create a _separate_ function declaration for your API. See the other language programming tutorials more details.
{% endhint %}

### Declare functions during model initialization <a href="#declare-functions-initialization" id="declare-functions-initialization"></a>

When you want to use function calling with a model, you must declare your functions when you initialize the model object. You declare functions by setting the model's `tools` parameter:

{% tabs %}
{% tab title="C# (AI Dev Kit)" %}
```csharp
var model = new GenerativeModel(
    GeminiModel.Gemini15Flash,
    tools: "set_light_values");
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
model = genai.GenerativeModel(model_name='gemini-1.5-flash',
                              tools=[set_light_values])
```
{% endtab %}
{% endtabs %}

### Generate a function call <a href="#generate-function-call" id="generate-function-call"></a>

Once you have initialized model with your function declarations, you can prompt the model with the defined function. You should use use function calling using chat prompting (`sendMessage()`), since function calling generally benefits from having the context of previous prompts and responses.

{% tabs %}
{% tab title="C#" %}
```csharp
var chat = model.StartChat();
var response = await chat.SendMessageAsync("Dim the lights so the room feels cozy and warm.");
Debug.Log(response.Text);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
chat = model.start_chat()
response = chat.send_message('Dim the lights so the room feels cozy and warm.')
response.text
```
{% endtab %}
{% endtabs %}

The AI Development Kit's [`ChatSession`](https://ai.google.dev/api/python/google/generativeai/ChatSession) object simplifies managing chat sessions by handling the conversation history for you. You can use the `enable_automatic_function_calling` to have the SDK automatically

{% tabs %}
{% tab title="C#" %}
```csharp
// Create a chat session that automatically makes suggested function calls
chat = model.StartChat(enableAutomaticFunctionCalling: true);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
# Create a chat session that automatically makes suggested function calls
chat = model.start_chat(enable_automatic_function_calling=True)
```
{% endtab %}
{% endtabs %}

**Warning:** **Do not use this feature in production applications** as there are no data input verification checks for automatic function calls.

### Parallel function calling <a href="#parallel_function_calling" id="parallel_function_calling"></a>

In addition to basic function calling described above, you can also call multiple functions in a single turn. This section shows an example for how you can use parallel function calling.

Step 1. Define the arguments for tools.

{% tabs %}
{% tab title="C#" %}
```csharp
// Powers the spinning disco ball.
public class PowerDiscoBallArg
{
    [JsonSchema("power", 
    Description = "Whether to power the disco ball or not.", 
    Required = true)]
    public bool Power { get; set; }
}

// Play some music matching the specified parameters.
public class StartMusicArg
{
    [JsonSchema("energetic", 
    Description = "Whether the music is energetic or not.", 
    Required = true)]
    public bool Energetic { get; set; }

    [JsonSchema("loud", 
    Description = "Whether the music is loud or not.", 
    Required = true)]
    public bool Loud { get; set; }

    [JsonSchema("bpm", 
    Description = "The beats per minute of the music.", 
    Required = true)]
    public int Bpm { get; set; }
}

// Dim the lights.
public class DimLightsArg
{
    [JsonSchema("brightness", 
    Description = "The brightness of the lights, 0.0 is off, 1.0 is full.", 
    Required = true)]
    public float Brightness { get; set; }
}
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
def power_disco_ball(power: bool) -> bool:
    """Powers the spinning disco ball."""
    print(f"Disco ball is {'spinning!' if power else 'stopped.'}")
    return True


def start_music(energetic: bool, loud: bool, bpm: int) -> str:
    """Play some music matching the specified parameters.

    Args:
      energetic: Whether the music is energetic or not.
      loud: Whether the music is loud or not.
      bpm: The beats per minute of the music.

    Returns: The name of the song being played.
    """
    print(f"Starting music! {energetic=} {loud=}, {bpm=}")
    return "Never gonna give you up."


def dim_lights(brightness: float) -> bool:
    """Dim the lights.

    Args:
      brightness: The brightness of the lights, 0.0 is off, 1.0 is full.
    """
    print(f"Lights are now set to {brightness:.0%}")
    return True
```
{% endtab %}
{% endtabs %}

Step 2. Define the delegates for tools.

{% tabs %}
{% tab title="C#" %}
```csharp
// Define function delegates
public class PowerDiscoBallDelegate : FunctionDelegate<PowerDiscoBallArg, Result<bool>>
{
    public override UniTask<Result<bool>> Invoke(PowerDiscoBallArg argument)
    {
        bool result = PowerDiscoBall(argument.Power);
        return UniTask.FromResult(Result.Success(result));
    }

    private bool PowerDiscoBall(bool power)
    {
        // Print the status of the disco ball
        Debug.Log($"Disco ball is {(power ? "spinning!" : "stopped.")}");

        // Return true to indicate success
        return true;
    }
}

public class StartMusicDelegate : FunctionDelegate<StartMusicArg, Result<string>>
{
    public override UniTask<Result<string>> Invoke(StartMusicArg argument)
    {
        string result = StartMusic(argument.Energetic, argument.Loud, argument.Bpm);
        return UniTask.FromResult(Result.Success<string>(result));
    }

    private string StartMusic(bool energetic, bool loud, int bpm)
    {
        // Simulate starting music
        return $"Music started with energetic={energetic}, loud={loud}, bpm={bpm}";
    }
}


public class DimLightsDelegate : FunctionDelegate<DimLightsArg, Result<bool>>
{
    public override UniTask<Result<bool>> Invoke(DimLightsArg argument)
    {
        bool result = DimLights(argument.Brightness);
        return UniTask.FromResult(Result.Success(result));
    }

    private bool DimLights(float brightness)
    {
        // Simulate dimming lights
        Debug.Log($"Lights dimmed to brightness level: {brightness}");
        return true;
    }
}
```
{% endtab %}
{% endtabs %}

Step 3. Now call the model with an instruction that could use all of the specified tools.

{% tabs %}
{% tab title="C#" %}
```csharp
// Set the model up with tools.
Tool houseFns = new Tool(
    new PowerDiscoBallDelegate(),
    new StartMusicDelegate(),
    new DimLightsDelegate()
);

var model = new GenerativeModel(GeminiModel.Gemini15Flash, tools: houseFns);

// Call the API.
var chat = model.StartChat();
var response = await chat.SendMessageAsync("Turn this place into a party!");

// Print out each of the function calls requested from this single call.
foreach (var part in response.Parts)
{
    if (part.FunctionCall != null)
    {
        var fn = part.FunctionCall;
        var args = string.Join(", ", fn.Args.Select(kv => $"{kv.Key}={kv.Value}"));
        Debug.Log($"{fn.Name}({args})");
    }
}
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
# Set the model up with tools.
house_fns = [power_disco_ball, start_music, dim_lights]

model = genai.GenerativeModel(model_name="gemini-1.5-flash", tools=house_fns)

# Call the API.
chat = model.start_chat()
response = chat.send_message("Turn this place into a party!")

# Print out each of the function calls requested from this single call.
for part in response.parts:
    if fn := part.function_call:
        args = ", ".join(f"{key}={val}" for key, val in fn.args.items())
        print(f"{fn.name}({args})")
```
{% endtab %}
{% endtabs %}

Each of the printed results reflects a single function call that the model has requested. To send the results back, include the responses in the same order as they were requested.

{% tabs %}
{% tab title="C#" %}
```csharp
// Simulate the responses from the specified tools.
var responses = new Dictionary<string, object>
{
    { "power_disco_ball", true },
    { "start_music", "Never gonna give you up." },
    { "dim_lights", true }
};

// Build the response parts.
var responseParts = responses.Select(kv =>
    new Part(new FunctionResponse(
        kv.Key, 
        new Dictionary<string, object> { { "result", kv.Value } }))).ToList();

response = await chat.SendMessageAsync(responseParts);
Debug.Log(response.Text);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
# Simulate the responses from the specified tools.
responses = {
    "power_disco_ball": True,
    "start_music": "Never gonna give you up.",
    "dim_lights": True,
}

# Build the response parts.
response_parts = [
    genai.protos.Part(function_response=genai.protos.FunctionResponse(name=fn, response={"result": val}))
    for fn, val in responses.items()
]

response = chat.send_message(response_parts)
print(response.text)
```
{% endtab %}
{% endtabs %}

```
Let's get this party started! 
I've turned on the disco ball, started playing some upbeat music, and dimmed the lights. 🎶✨  
Get ready to dance! 🕺💃
```

### Function call data type mapping <a href="#map-data-types" id="map-data-types"></a>

When using the AI Development Kit to extract schemas from C# methods, certain cases, such as nested dictionary-objects, may not be handled correctly. However, the API does support these types. The supported types include:

* `int`
* `float`
* `bool`
* `string`
* `List<AllowedType>`
* `Dictionary<string, AllowedType>`

**Important:** The SDK converts method parameter type annotations to a format the API understands. The API only supports a limited selection of parameter types, and the C# SDK's automatic conversion only supports a subset of the allowed types above.

First peek inside the model's `JsonSchema` attribute, you can see how it describes the function(s) you passed it to the model:

{% tabs %}
{% tab title="C#" %}
<pre class="language-csharp"><code class="lang-csharp">using Cysharp.Threading.Tasks;
using Glitch9.AIDevKit.Google.GenerativeAI;
using UnityEngine;
<strong>
</strong>// Define the schema for the function arguments.
public class CalculatorArg
{
    [JsonSchema("a", Description = "The first number.", Required = true)]
    public float a { get; set; }

    [JsonSchema("b", Description = "The second number.", Required = true)]
    public float b { get; set; }
}

// Define the function delegate.
public class MultiplyDelegate : FunctionDelegate&#x3C;CalculatorArg, Result&#x3C;float>>
{
    public override UniTask&#x3C;Result&#x3C;float>> Invoke(CalculatorArg argument)
    {
        float result = argument.a * argument.b;
        return UniTask.FromResult(Result.Success(result));
    }
}

// Set the model up with tools.
Tool multiplyTool = new Tool(new MultiplyDelegate());
var model = new GenerativeModel(GeminiModel.Gemini15Flash, tools: multiplyTool);

// Call the API.
var chat = model.StartChat();
var response = await chat.SendMessageAsync("What is 2 times 3?");

// Print out the response.
Debug.Log(response.Text);
</code></pre>
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
def multiply(a:float, b:float):
    """returns a * b."""
    return a*b

model = genai.GenerativeModel(model_name='gemini-1.5-flash',
                             tools=[multiply])

model._tools.to_proto()
```
{% endtab %}
{% endtabs %}

Execute the function yourself:

{% tabs %}
{% tab title="C#" %}
```csharp
var fc = response.Candidates[0].Content.Parts[0].FunctionCall;
Debug.Assert(fc.Name == "multiply");

if (!fc.Args.TryGetValue("a", out var aObj) ||
    !fc.Args.TryGetValue("b", out var bObj) ||
    aObj is not float a || bObj is not float b)
{
    Debug.LogError("Invalid arguments.");
    return;
}

float result = a * b;
Debug.Log(result);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
fc = response.candidates[0].content.parts[0].function_call
assert fc.name == 'multiply'

result = fc.args['a'] * fc.args['b']
#result: 76358547152.0 
```
{% endtab %}
{% endtabs %}

Send the result to the model, to continue the conversation:

{% tabs %}
{% tab title="C#" %}
```csharp
var responseParts = new List<Part>
{
    new Part(
        new FunctionResponse(
            "multiply", 
            new Dictionary<string, object> { { "result", result } }))
};

response = await chat.SendMessageAsync(responseParts);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
response = chat.send_message(
    genai.protos.Content(
    parts=[genai.protos.Part(
        function_response = genai.protos.FunctionResponse(
          name='multiply',
          response={'result': result}))]))
```
{% endtab %}
{% endtabs %}

{% embed url="https://ai.google.dev/gemini-api/docs/function-calling" %}
Google official document
{% endembed %}
