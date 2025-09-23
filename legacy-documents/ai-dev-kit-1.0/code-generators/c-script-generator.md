# C# Script Generator

**C# Script Generator** streamlines the creation of C# scripts for Unity developers. By leveraging OpenAI's GPT models, it generates scripts based on natural language prompts, facilitating rapid development of functionalities that would typically require more time to code manually.

### Accessing the C# Script Generator

To access the C# Script Generator:

1. Navigate to the Project window within the Unity Editor.
2. Right-click on the desired directory where you wish to generate the new script.
3.  Find the <mark style="color:purple;">**Generate C# Script with GPT**</mark> button located directly under the <mark style="color:purple;">**Create**</mark> button at the top of the context menu.

    <figure><img src="../../../.gitbook/assets/image (39).png" alt="" width="563"><figcaption></figcaption></figure>

### Using the C# Script Generator

#### Step 1: Inputting Your Prompt

<figure><img src="../../../.gitbook/assets/image (37).png" alt="" width="447"><figcaption></figcaption></figure>

After opening the C# Script Generator window, you'll be prompted to describe the script you want to generate. This description should be as detailed as possible to ensure the generated script meets your needs. Examples of prompts include:

* "Implement a generic singleton pattern for game managers."
* "Create a player movement script using Rigidbody for a 3D platformer."

#### Step 2: Configuring Script Options

* **Namespace**: Optionally, specify the namespace for your script. This helps in organizing your code and preventing naming conflicts.
* **Model Selection**: Choose between available GPT models for script generation. Different models may offer varying levels of creativity and accuracy.

#### Step 3: Generating the Script

Once you have input your prompt and configured the script options, click the "Generate" button. The tool will communicate with OpenAI's API and generate a C# script based on your description.

#### Step 4: Reviewing and Confirming

<figure><img src="../../../.gitbook/assets/image (38).png" alt="" width="448"><figcaption></figcaption></figure>

* **Review**: The generated script will be displayed for your review. You can read through the code to ensure it aligns with your project requirements.
* **Confirmation**: If the script meets your expectations, confirm the generation to add the script to your project. If not, you can modify the prompt and regenerate the script.

### Best Practices

* **Detailed Prompts**: Provide detailed and specific prompts to improve the relevance and accuracy of the generated scripts.
* **Prompt Examples**: Use the "Show me some examples" feature for inspiration or to better understand how to formulate effective prompts.
* **Review Generated Code**: Always review the generated code for accuracy, efficiency, and adherence to best practices before integrating it into your project.
