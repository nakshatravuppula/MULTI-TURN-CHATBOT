# MULTI-TURN-CHATBOT
import gradio as gr
from google import genai

# ====================================================
# Gemini API
# ====================================================

API_KEY = "YOUR_GEMINI_API_KEY"

client = genai.Client(api_key=API_KEY)

MODEL = "gemini-2.5-flash"

SYSTEM_PROMPT = """
You are an AI Cooking Assistant.

You help users with:

- Recipes
- Healthy food suggestions
- Cooking tips
- Ingredient substitutions
- Nutrition
- Cooking time
- Meal planning

Always answer politely.

If user asks unrelated questions,
politely remind them that you specialize
in cooking assistance.
"""

# ====================================================
# Chat Function
# ====================================================

def chatbot(message, history):

    history = history or []

    contents = []

    # System prompt
    contents.append(
        {
            "role": "user",
            "parts": [{"text": SYSTEM_PROMPT}]
        }
    )

    contents.append(
        {
            "role": "model",
            "parts": [{"text": "Understood. I am ready."}]
        }
    )

    # Previous conversation
    for user_msg, bot_msg in history:

        contents.append(
            {
                "role": "user",
                "parts": [{"text": user_msg}]
            }
        )

        contents.append(
            {
                "role": "model",
                "parts": [{"text": bot_msg}]
            }
        )

    # Current message
    contents.append(
        {
            "role": "user",
            "parts": [{"text": message}]
        }
    )

    response = client.models.generate_content(
        model=MODEL,
        contents=contents
    )

    answer = response.text

    history.append((message, answer))

    return history, history

# ====================================================
# Clear Chat
# ====================================================

def clear():

    return [], []

# ====================================================
# Gradio UI
# ====================================================

with gr.Blocks(title="AI Cooking Chatbot") as demo:

    gr.Markdown(
        """
# 🍳 AI Cooking Chatbot

Ask anything about cooking!

Examples:

- Chicken biryani recipe
- Healthy breakfast
- Calories in dosa
- Pasta recipe
- Cake without eggs
"""
    )

    chatbot_ui = gr.Chatbot(height=500)

    state = gr.State([])

    txt = gr.Textbox(
        placeholder="Ask a cooking question...",
        label="Message"
    )

    send = gr.Button("Send")

    clear_btn = gr.Button("Clear Chat")

    send.click(
        chatbot,
        inputs=[txt, state],
        outputs=[chatbot_ui, state]
    )

    txt.submit(
        chatbot,
        inputs=[txt, state],
        outputs=[chatbot_ui, state]
    )

    clear_btn.click(
        clear,
        outputs=[chatbot_ui, state]
    )

demo.launch()
