# Creating custom functions

Custom functions allow you to extend the capabilities of your Assistant by defining specific actions that the Assistant can perform. Here's how to create custom functions using the `FunctionCall` class. You can create a `FunctionCall` instance by specifying the function's name, description, and an optional delegate that will execute the function. There are two methods for creating a `FunctionCall` instance:

### **1. Without Parameters Schema**

If you need a `FunctionCall` that returns response in <mark style="color:blue;">**Text**</mark> (`string`) format, use the following method to create a `FunctionCall` instance with the specified name, description, and delegate.

```csharp
// Creating functionCall without parameters schema
var functionCall = FunctionCall.Create("MyFunction", "This is a sample function", new MyFunctionDelegate());
```

### **2. With Parameters Schema**

To create a `FunctionCall` that returns a Run with custom object argument(<mark style="color:green;">`T`</mark>), define your custom argument class with properties with `JsonSchemaAttribute`. Only `string` and `enum` types are supported.

<pre class="language-csharp"><code class="lang-csharp">using Glitch9.IO.Json.Schema;

// Define your custom return class with JsonSchemaAttributes.
public class GetCurrentTemperatureArgument
{
<strong>    // Example string property: Define properties according to the expected API response structure.
</strong>    [JsonSchema("location", Description = "The city and state, e.g., San Francisco, CA.", required: true)]
    public string Location { get; set; }
    
    // Example enum property: If your response includes fields that can be categorized into enums,
    // define an enum and use it as the type for such properties.
    [JsonSchema("unit", Description = "The temperature unit to use. Infer this from the user's location.", required: false)]
    public TemperatureUnit Unit { get; set; }
    
    // Properties without JsonSchemaAttribute won't be included in Parameters Schema.
    [JsonProperty("temperature")]
    public string Temperature { get; set; }
}

// Example enum to represent a category or type of response.
// "enum": ["Celsius", "Fahrenheit"],
public enum TemperatureUnit
{
    Celsius,
    Fahrenheit
}
</code></pre>

Then use the following method create a `FunctionCall` instance with the specified name, description, and delegate. The parameters for the function are generated based on the specified type <mark style="color:green;">`T`</mark>.

```csharp
// Creating functionCall with parameters schema
var functionCall = FunctionCall.Create<GetCurrentTemperatureArgument>("get_current_temperature", "Get the current temperature for a specific location.", new MyFunctionDelegate());
```

### 3. Extending the Function Delegate

A function delegate is a class that extends the `FunctionDelegate` class. This delegate will delegate [submit tool outputs to run](https://platform.openai.com/docs/api-reference/runs/submitToolOutputs) when OpenAI AssistantsAPI returns a **Run** with `RunStatus.RequiresAction`.

```csharp
using Cysharp.Threading.Tasks;
using Glitch9.AIDevelopmentKit;

public class GetCurrentTemperatureDelegate<GetCurrentTemperatureArgument> : FunctionDelegate
{ 
    // Example dummy function hard coded to return the same weather
    // In production, this could be your backend API or an external API
    public override async UniTask<GetCurrentTemperatureArgument> ExecuteInternal(GetCurrentTemperatureArgument argument)
    {        
        if (argument.Location.ToLowerCase().Contains("tokyo"))
        {
            argument.Temperature = "10";
            argument.Unit = TemperatureUnit.Celsius;
            return argument;
        }
        else if (argument.Location.ToLowerCase().Contains("san francisco"))
        {
            argument.Temperature = "72";
            argument.Unit = TemperatureUnit.Fahrenheit;
            return argument;
        }
        else if (argument.Location.ToLowerCase().Contains("paris"))
        {
            argument.Temperature = "22";
            argument.Unit = TemperatureUnit.Fahrenheit;
            return argument;
        }
        else
        {
            argument.Temperature = "unknown";
            return argument;
        }
    }
}
```

{% hint style="warning" %}
If no function delegate is provided to a `FunctionCall` and the AssistantsAPI returns a **Run** with `RunStatus.RequiresAction`, `AssistantsAPIv2` will wait until required outputs are submitted with `AssistantsAPIv2.SubmitToolOutput`.
{% endhint %}
