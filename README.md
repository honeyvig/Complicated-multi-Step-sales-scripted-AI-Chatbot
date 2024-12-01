# Complicated-multi-Step-sales-scripted-AI-Chatbot
Build an AI chatbot that follows multiple flows of a sales script, using both "free text" and predefined messages. Use a scoring system to make sure that the AI is responding to user inputs correctly. API integration.
==========
Below is Python code that creates a sales chatbot using OpenAI's GPT and includes a scoring mechanism to evaluate user interactions. The chatbot alternates between predefined script flows and free-text responses, adjusting dynamically based on user inputs.
Key Features:

    Predefined Sales Script Flows: Guided responses for specific steps.
    Free Text Responses: Handles unstructured inputs using GPT.
    Scoring System: Evaluates whether user responses align with expected inputs.
    API Integration: Utilizes OpenAI API for generating dynamic responses.

Code Implementation

import openai
from flask import Flask, request, jsonify

# Initialize Flask app
app = Flask(__name__)

# OpenAI API key
openai.api_key = "your_openai_api_key"

# Predefined sales script steps
sales_script = [
    {"step": 1, "prompt": "Welcome! Are you interested in learning about our product?", "expected": ["yes", "sure", "tell me more"]},
    {"step": 2, "prompt": "Our product helps you save time and money. Can I ask what challenges you’re facing?", "expected": ["time", "money", "efficiency"]},
    {"step": 3, "prompt": "Great! Our solution is designed specifically for those challenges. Would you like to see a demo?", "expected": ["yes", "demo", "show me"]},
]

# Scoring function
def score_response(user_input, expected_responses):
    """
    Scores the user's response based on how well it matches the expected responses.
    """
    for expected in expected_responses:
        if expected.lower() in user_input.lower():
            return 1  # Correct response
    return 0  # Incorrect response

# Chatbot logic
def chatbot_logic(user_input, current_step):
    """
    Handles chatbot flow based on user input and current step.
    """
    # If we are within the script flow
    if current_step < len(sales_script):
        script_step = sales_script[current_step]
        expected_responses = script_step["expected"]
        score = score_response(user_input, expected_responses)
        
        if score > 0:
            response = f"Thank you! Let's proceed. {sales_script[current_step]['prompt']}"
            next_step = current_step + 1
        else:
            response = f"I'm sorry, I didn't quite catch that. {sales_script[current_step]['prompt']}"
            next_step = current_step
        return response, next_step, score
    
    # If script is complete, switch to free text
    else:
        gpt_response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": "You are a helpful sales assistant."},
                {"role": "user", "content": user_input},
            ]
        )
        return gpt_response["choices"][0]["message"]["content"], current_step, 0

# Route for chatbot interaction
@app.route('/chat', methods=['POST'])
def chat():
    """
    Handles incoming chat requests.
    """
    data = request.json
    user_input = data.get("user_input", "")
    current_step = data.get("current_step", 0)

    # Process user input
    response, next_step, score = chatbot_logic(user_input, current_step)

    return jsonify({
        "response": response,
        "next_step": next_step,
        "score": score
    })

if __name__ == "__main__":
    app.run(debug=True)

How It Works:

    Flask Backend:
        Handles user interactions via a /chat endpoint.
        Receives user_input and the current_step of the sales script.
        Returns the bot's response, next step, and a score for the current interaction.

    Sales Script Flow:
        Guides users through predefined steps.
        Validates responses against expected answers and adjusts flow accordingly.

    Free Text Handling:
        When the script is complete, the chatbot uses OpenAI's GPT model for conversational responses.

    Scoring:
        Scores each user response (1 for correct, 0 for incorrect) to track performance.

API Requirements:

    OpenAI GPT API: For dynamic text generation.
    Flask: For creating the web interface.
    JSON Input/Output: For handling user data.

Example Request:
Input:

{
    "user_input": "Yes, I’d like to know more.",
    "current_step": 0
}

Response:

{
    "response": "Thank you! Let's proceed. Our product helps you save time and money. Can I ask what challenges you’re facing?",
    "next_step": 1,
    "score": 1
}

Deployment:

    Install required libraries:

pip install flask openai

Replace "your_openai_api_key" with your OpenAI API key.
Run the Flask app:

    python app.py

    Use tools like Postman or a frontend to interact with the chatbot.

This framework is modular and can be expanded with features like:

    Analytics Dashboard: To track scores and chatbot effectiveness.
    API Integrations: Add CRM or analytics tools for better insights.
