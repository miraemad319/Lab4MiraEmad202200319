Read Me (Conversational Agent)
This project implements a conversational agent that can fetch weather information and respond to queries using reasoning techniques. 

The agent supports:
1. Weather Retrieval: Fetches current weather and multi-day forecasts.
2. Chain of Thought (CoT): Breaks down complex weather-related queries into logical steps.
3. ReAct Framework: Uses an iterative approach of thinking, acting, and observing to improve responses.

Installation:
Clone this repository: git clone https://github.com/your-username/your-repo-name.git

Install the necessary dependencies: 
pip install openai python-dotenv requests

Create a .env file with API keys:
OPENAI_API_KEY=your_openai_api_key
WEATHER_API_KEY=your_weatherapi_key

Run the script using:
python conversational_agent.py

Example Interaction: Travel Planning Assistant
User: I’m planning a trip to Italy next week. What’s the weather like in Rome for the next three days?

Agent:
Thought: The user wants a weather forecast for Rome over the next three days.
Action: [Calls get_weather_forecast(location="Rome", days=3)]
Observation:

Day 1: 22°C, sunny, 10% chance of rain
Day 2: 20°C, partly cloudy, 15% chance of rain
Day 3: 18°C, rainy, 70% chance of rain
Final Answer: The weather in Rome for the next three days:

Day 1: 22°C, sunny (10% chance of rain)
Day 2: 20°C, partly cloudy (15% chance of rain)
Day 3: 18°C, rainy (70% chance of rain)
User: That’s helpful! Can you compare it with the weather in Madrid?

Agent:
Thought: The user wants a weather comparison between Rome and Madrid for the next three days.
Action: [Calls get_weather_forecast(location="Madrid", days=3)]
Observation:

Day 1: 25°C, sunny, 5% chance of rain
Day 2: 24°C, clear skies, 0% chance of rain
Day 3: 22°C, partly cloudy, 10% chance of rain
Final Answer: Comparing Rome and Madrid:

Rome: Cooler overall, with rain expected on the third day.
Madrid: Warmer and sunnier, with no rain expected.
User: I guess Madrid is a better choice for a sunny vacation! Thanks!

Agent: You’re welcome! Have a fantastic trip!

Challenges Faced:
1.. Handling Errors: Implemented messages to handle failed API responses.
2. Reasoning Steps Debugging: Ensured the agent follows a structured Thought → Action → Observation process.
3.Maintaining Context in Multi-Turn Conversations: Users may reference previous queries (e.g., "What about tomorrow?" after asking about today’s weather). Keeping track of context while dynamically updating responses was an important challenge to address.

References
OpenAI API Documentation
WeatherAPI Documentation
