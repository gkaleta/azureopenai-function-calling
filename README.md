# Azureopenai-Function-Calling
## OpenAI Function Calling tool - working in AzureOpenai

| API | Description |
| --- | --- |
| OpenAI | Ensure you have 1.8x or 1.9 |
| LLM Model | Ensure you using gpt-4-1106 preview |
| Azure API | You must use 2023-12-01-preview |
| location | In the writing of this 22th Jan avoid SouthUK - this was tested with CentralSweden |

Make sure you have your os env vars in place:
 
 ```bash
export AZURE_OPENAI_ENDPOINT="https://abc.openai.azure.com/"
export AZURE_OPENAI_KEY="abc"
```

## Python sample app

 ```python
import os
from openai import AzureOpenAI
import json

client = AzureOpenAI(
  azure_endpoint = os.getenv("AZURE_OPENAI_ENDPOINT"),
  api_key=os.getenv("AZURE_OPENAI_KEY"),
  api_version="2023-12-01-preview"
)
deployment_id="gpt-4-1106-preview" #random name for the 1106 preview model
def get_current_weather(location, unit="fahrenheit"):
    """Get the current weather in a given location"""
    if "tokyo" in location.lower():
        return json.dumps({"location": "Tokyo", "temperature": "10", "unit": unit})
    elif "san francisco" in location.lower():
        return json.dumps({"location": "San Francisco", "temperature": "72", "unit": unit})
    elif "paris" in location.lower():
        return json.dumps({"location": "Paris", "temperature": "22", "unit": unit})
    else:
        return json.dumps({"location": location, "temperature": "unknown"})
# Step 1: send the conversation and available functions to the model
messages = [{"role": "user", "content": "What's the weather like in San Francisco, Tokyo, and Paris?"}]
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_current_weather",
            "description": "Get the current weather in a given location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "The city and state, e.g. San Francisco, CA",
                    },
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                },
                "required": ["location"],
            },
        },
    }
]
response = client.chat.completions.create(
    model=deployment_id,
    messages=messages,
    tools=tools,
    tool_choice="auto",
)
response_message = response.choices[0].message
print(response_message)



for tool_call in response_message.tool_calls:
    # get the function name
    function_name = tool_call.function.name
    # get the function arguments
    arguments = json.loads(tool_call.function.arguments)
    # print the function name and arguments
    print(f'Function Name: {function_name}')
    print(f'Arguments: {arguments}')
    
 ```

Output should look like: 

 ```ruby
Function Name: get_current_weather
Arguments: {'location': 'San Francisco, CA', 'unit': 'celsius'}
Function Name: get_current_weather
Arguments: {'location': 'Tokyo, Japan', 'unit': 'celsius'}
Function Name: get_current_weather
Arguments: {'location': 'Paris, France', 'unit': 'celsius'}
 ```



