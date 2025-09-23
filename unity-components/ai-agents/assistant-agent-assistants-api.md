---
description: >-
  Use this component to integrate OpenAI's Assistants API into your Unity
  project. This allows AI agents to run tools, maintain threads, and perform
  multi-step actions through OpenAI’s hosted memory.
icon: message-bot
---

# Assistant Agent (Assistants API)

#### What it does

This component enables:

* Text input & response using the OpenAI Assistants API
* Voice input via Speech-to-Text (optional)
* Voice output via Text-to-Speech (optional)
* Image generation when requested by the assistant (optional)
* Tool/function calling through OpenAI’s built-in tool invocation system
* Multi-step reasoning using threads, runs, and memory management provided by the Assistants API

***

<figure><img src="../../.gitbook/assets/image (108).png" alt="" width="563"><figcaption></figcaption></figure>

### 1. Assistant Setup

This section defines the identity and behavior of your assistant.

| Field              | Description                                                                          |
| ------------------ | ------------------------------------------------------------------------------------ |
| Selected Assistant | Reference to an `Assistant` object. Defines the assistant's configuration and tools. |
| Assistant Name     | Display name of this assistant in the Editor. For UI purposes.                       |
| Description        | Optional description shown in the Editor or UI.                                      |
| Instructions       | System prompt to define the assistant’s personality and role.                        |
| Tools              | List of tools (function names) the assistant can use. Set in the `Assistant` object. |

***

### 2. Assistant Behavior

These options control how the assistant responds.

| Field           | Description                                                                                                                                                                                                                                    |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Model           | OpenAI model to use (e.g., GPT-4o). Set per run.                                                                                                                                                                                               |
| Response Format | Select the format of the assistant's response: `text`, `json`, `json_schema`, or `auto`. Use `text` for natural language replies, and `json` or `json_schema` for structured responses. `auto` lets the model choose the most suitable format. |
| Stream          | Enable to receive streaming responses from the assistant.                                                                                                                                                                                      |

***

### 3. Advanced Options

Optional fine-tuning of model behavior.

| Field       | Description                                  |
| ----------- | -------------------------------------------- |
| Temperature | Controls randomness. Higher = more creative. |
| Top P       | Controls diversity. Lower = more focused.    |

***

### 4. Modules

Optional components that enhance the assistant’s capabilities.

| Module           | Description                                               |
| ---------------- | --------------------------------------------------------- |
| Function Manager | Executes Unity functions when the assistant calls a tool. |
| Speech-to-Text   | Enables voice input.                                      |
| Text-to-Speech   | Converts assistant responses to voice.                    |
| Image Generator  | Allows assistant to generate images when appropriate.     |

***

### 5. Event Receivers

Use these to trigger UnityEvents in response to assistant activity.

| Event Type               | Description                                        |
| ------------------------ | -------------------------------------------------- |
| Chat Event Receiver      | Triggered when a message is sent or received.      |
| Tool Call Receiver       | Triggered when the assistant requests a tool call. |
| Streaming Event Receiver | Triggered as streamed tokens arrive.               |
| Error Receiver           | Triggered on any exception during chat.            |

***

### 6. Life Cycle Event Receivers

Receive callbacks for lower-level events in the Assistants API flow.

| Receiver Type            | Description                                    |
| ------------------------ | ---------------------------------------------- |
| Assistant Event Receiver | Called on assistant load/update.               |
| Thread Event Receiver    | Called on thread creation/update.              |
| Run Event Receiver       | Called when a new run is started or completed. |
| Message Event Receiver   | Called on message creation/receipt.            |

***

### 7. Required Action Handling

If the assistant uses `required_actions` (e.g., waits for a tool to complete), this section manages how Unity should respond.

| Field                   | Description                                                                            |
| ----------------------- | -------------------------------------------------------------------------------------- |
| Ignore Required Actions | If enabled, assistant won’t wait for function results.                                 |
| Required Action Timeout | Time (in seconds) before a required action times out.                                  |
| On Required Action      | A list of UnityEvents to trigger when a required action is detected (e.g., tool call). |

