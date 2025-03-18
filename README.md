# Set the BASE_URL and LLM_MODEL
BASE_URL = "https://api.openai.com/v1"  # Default OpenAI API endpoint
LLM_MODEL = "gpt-3.5-turbo"  # Use GPT-3.5 Turbo as the default model

import os
import json
import requests
from openai import OpenAI
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()
API_KEY = os.environ.get("add key") 
BASE_URL = "https://api.openai.com/v1"  # Default OpenAI API endpoint
LLM_MODEL = "gpt-3.5-turbo"  # Use GPT-3.5 Turbo as the default model

# Initialize the OpenAI client
client = OpenAI(
    api_key="add key",
    base_url="https://api.openai.com/v1"
)
def get_current_weather(location):
    """Get the current weather for a location."""
    api_key = os.environ.get("WEATHER_API_KEY")
    url = f"http://api.weatherapi.com/v1/current.json?key={api_key}&q={location}&aqi=no"
    response = requests.get(url)
    data = response.json()
    if "error" in data:
        return f"Error: {data['error']['message']}"
    weather_info = data["current"]
    return json.dumps({
        "location": data["location"]["name"],
        "temperature_c": weather_info["temp_c"],
        "temperature_f": weather_info["temp_f"],
        "condition": weather_info["condition"]["text"],
        "humidity": weather_info["humidity"],
        "wind_kph": weather_info["wind_kph"]
    })

def get_weather_forecast(location, days=3):
    """Get a weather forecast for a location for a specified number of days."""
    api_key = os.environ.get("WEATHER_API_KEY")
    url = f"http://api.weatherapi.com/v1/forecast.json?key={api_key}&q={location}&days={days}&aqi=no"
    response = requests.get(url)
    data = response.json()
    if "error" in data:
        return f"Error: {data['error']['message']}"
    forecast_days = data["forecast"]["forecastday"]
    forecast_data = []
    for day in forecast_days:
        forecast_data.append({
            "date": day["date"],
            "max_temp_c": day["day"]["maxtemp_c"],
            "min_temp_c": day["day"]["mintemp_c"],
            "condition": day["day"]["condition"]["text"],
            "chance_of_rain": day["day"]["daily_chance_of_rain"]
        })
    return json.dumps({
        "location": data["location"]["name"],
        "forecast": forecast_data
    })

# Define the tools for the OpenAI API
weather_tools = [
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
                        "description": "The city and state, e.g., San Francisco, CA or country e.g., France",
                    }
                },
                "required": ["location"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "get_weather_forecast",
            "description": "Get the weather forecast for a location for a specific number of days",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "The city and state, e.g., San Francisco, CA or country e.g., France",
                    },
                    "days": {
                        "type": "integer",
                        "description": "The number of days to forecast (1-10)",
                        "minimum": 1,
                        "maximum": 10
                    }
                },
                "required": ["location"],
            },
        },
    }
]

available_functions = {
    "get_current_weather": get_current_weather,
    "get_weather_forecast": get_weather_forecast
}
# Process messages and invoke tools as needed
def process_messages(client, messages, tools=None, available_functions=None):
    """
    Process messages and invoke tools as needed.
    Args:
        client: The OpenAI client
        messages: The conversation history
        tools: The available tools
        available_functions: A dictionary mapping tool names to functions
    Returns:
        The list of messages with new additions
    """
    tools = tools or []
    available_functions = available_functions or {}

    # Step 1: Send the messages to the model with the tool definitions
    response = client.chat.completions.create(
        model=LLM_MODEL,
        messages=messages,
        tools=tools,
    )
    response_message = response.choices[0].message

    # Step 2: Append the model's response to the conversation
    messages.append(response_message)

    # Step 3: Check if the model wanted to use a tool
    if response_message.tool_calls:
        # Step 4: Extract tool invocation and make evaluation
        for tool_call in response_message.tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions[function_name]
            function_args = json.loads(tool_call.function.arguments)
            function_response = function_to_call(**function_args)

            # Step 5: Extend conversation with function response
            messages.append({
                "tool_call_id": tool_call.id,
                "role": "tool",
                "name": function_name,
                "content": function_response,
            })

    return messages


# Run a conversation with the user
def run_conversation(client, system_message="You are a helpful weather assistant."):
    """
    Run a conversation with the user, processing their messages and handling tool calls.
    Args:
        client: The OpenAI client
        system_message: The system message to initialize the conversation
    Returns:
        The final conversation history
    """
    messages = [
        {
            "role": "system",
            "content": system_message,
        }
    ]
    print("Weather Assistant: Hello! I can help you with weather information. Ask me about the weather anywhere!")
    print("(Type 'exit' to end the conversation)\n")

    while True:
        # Request user input and append to messages
        user_input = input("You: ")
        if user_input.lower() in ["exit", "quit", "bye"]:
            print("\nWeather Assistant: Goodbye! Have a great day!")
            break

        messages.append({
            "role": "user",
            "content": user_input,
        })

        # Process the messages and get tool calls if any
        messages = process_messages(client, messages, weather_tools, available_functions)

        # Check the last message to see if it's from the assistant
        last_message = messages[-1]
        if last_message["role"] == "assistant" and last_message.get("content"):
            print(f"\nWeather Assistant: {last_message['content']}\n")

    return messages

if __name__ == "__main__":
    run_conversation(client)
