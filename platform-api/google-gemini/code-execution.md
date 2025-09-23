# 💻 Code execution

#### Enable code execution on the model <a href="#enable-on-model" id="enable-on-model"></a>

You can enable code execution on the model, as shown here:

{% tabs %}
{% tab title="C#" %}
```javascript
using Glitch9.AIDevKit.Google.GenerativeAI;

var model = new GenerativeModel(
    GeminiModel.Gemini15Pro, 
    tools: "code_execution");

var response = await model.GenerateContentAsync(
    "What is the sum of the first 50 prime numbers? " +
    "Generate and run code for the calculation, and make sure you get all 50.");

Debug.Log(response.Text);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
import os
import google.generativeai as genai

genai.configure(api_key=os.environ['API_KEY'])

model = genai.GenerativeModel(
    model_name='gemini-1.5-pro',
    tools='code_execution')

response = model.generate_content((
    'What is the sum of the first 50 prime numbers? '
    'Generate and run code for the calculation, and make sure you get all 50.'))

print(response.text)
```
{% endtab %}
{% endtabs %}

The output might look something like this:

````
```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        int sumOfFirst50Primes = SumOfPrimes(50);
        Console.WriteLine($"The sum of the first 50 prime numbers is: {sumOfFirst50Primes}");
    }

    static bool IsPrime(int n)
    {
        if (n <= 1)
            return false;
        for (int i = 2; i <= Math.Sqrt(n); i++)
        {
            if (n % i == 0)
                return false;
        }
        return true;
    }

    static int SumOfPrimes(int n)
    {
        List<int> primes = new List<int>();
        int i = 2;
        while (primes.Count < n)
        {
            if (IsPrime(i))
                primes.Add(i);
            i++;
        }
        int sum = 0;
        foreach (int prime in primes)
        {
            sum += prime;
        }
        return sum;
    }
}
```

**Explanation:**

1. **`is_prime(n)` Function:**
   - Takes an integer `n` as input.
   - Returns `False` for numbers less than or equal to 1 (not prime).
   - Iterates from 2 up to the square root of `n`. If `n` is divisible by any
     number in this range, it's not prime, and we return `False`.
   - If the loop completes without finding a divisor, the number is prime, and
     we return `True`.

2. **`sum_of_primes(n)` Function:**
   - Takes an integer `n` (number of primes desired) as input.
   - Initializes an empty list `primes` to store the prime numbers.
   - Starts a loop, iterating through numbers starting from 2.
   - For each number `i`, it checks if it's prime using the `is_prime()` function.
   - If `i` is prime, it's appended to the `primes` list.
   - The loop continues until the `primes` list has `n` prime numbers.
   - Finally, it calculates and returns the sum of all the prime numbers in the
     `primes` list.

3. **Main Part:**
   - Calls `sum_of_primes(50)` to get the sum of the first 50 prime numbers.
   - Prints the result.

**Output:**

```
The sum of the first 50 prime numbers is: 5117
```
````

#### Enable code execution on the request <a href="#enable-on-request" id="enable-on-request"></a>

Alternatively, you can enable code execution on the call to `generate_content`:

{% tabs %}
{% tab title="C#" %}
```javascript
using Glitch9.AIDevKit.Google.GenerativeAI;

GenerativeModel model = new(
    GeminiModel.Gemini15Pro);

var response = await model.GenerateContentAsync(
    "What is the sum of the first 50 prime numbers? " +
    "Generate and run code for the calculation, and make sure you get all 50.",
    tools: "code_execution");

Debug.Log(response.GetOutputText());
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
import os
import google.generativeai as genai

genai.configure(api_key=os.environ['API_KEY'])

model = genai.GenerativeModel(model_name='gemini-1.5-pro')

response = model.generate_content(
    ('What is the sum of the first 50 prime numbers? '
    'Generate and run code for the calculation, and make sure you get all 50.'),
    tools='code_execution')

print(response.text)
```
{% endtab %}
{% endtabs %}

#### Use code execution in chat <a href="#code-in-chat" id="code-in-chat"></a>

You can also use code execution as part of a chat.

{% tabs %}
{% tab title="C#" %}
```javascript
using Glitch9.AIDevKit.Google.GenerativeAI;

var model = new GenerativeModel(GeminiModel.Gemini15Pro);

var chat = model.StartChat();

var response = await chat.SendMessageAsync(
    "What is the sum of the first 50 prime numbers? " +
    "Generate and run code for the calculation, and make sure you get all 50.");

Debug.Log(response.Text);
```
{% endtab %}

{% tab title="Python (Official SDK)" %}
```python
import os
import google.generativeai as genai

genai.configure(api_key=os.environ['API_KEY'])

model = genai.GenerativeModel(model_name='gemini-1.5-pro')

chat = model.start_chat()

response = chat.send_message((
    'What is the sum of the first 50 prime numbers? '
    'Generate and run code for the calculation, and make sure you get all 50.'))

print(response.text)
```
{% endtab %}
{% endtabs %}

### Code execution versus function calling <a href="#code-execution-vs-function-calling" id="code-execution-vs-function-calling"></a>

Code execution and [function calling](https://ai.google.dev/gemini-api/docs/function-calling) are similar features:

* Code execution lets the model run code in the API backend in a fixed, isolated environment.
* Function calling lets you run the functions that the model requests, in whatever environment you want.

In general you should prefer to use code execution if it can handle your use case. Code execution is simpler to use (you just enable it) and resolves in a single `GenerateContent` request (thus incurring a single charge). Function calling takes an additional `GenerateContent` request to send back the output from each function call (thus incurring multiple charges).

For most cases, you should use function calling if you have your own functions that you want to run locally, and you should use code execution if you'd like the API to write and run Python code for you and return the result.

{% embed url="https://ai.google.dev/gemini-api/docs/code-execution" %}
Google official document
{% endembed %}
