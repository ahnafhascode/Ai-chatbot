# Ai-chatbot
A chatbot like a openai chatgpt in python language.
import requests
import openai

# -----------------------------
# API KEYS (replace with yours)
# -----------------------------
OPENAI_API_KEY = "your-openai-key"
GOOGLE_API_KEY = "your-google-api-key"
GOOGLE_CX = "your-custom-search-engine-id"  # For Google Custom Search
GEMINI_API_KEY = "your-gemini-key"
DEEPSEEK_API_KEY = "your-deepseek-key"

openai.api_key = OPENAI_API_KEY

# -----------------------------
# FUNCTIONS TO GET ANSWERS
# -----------------------------

def search_google(query):
    url = f"https://www.googleapis.com/customsearch/v1?q={query}&key={GOOGLE_API_KEY}&cx={GOOGLE_CX}"
    res = requests.get(url).json()
    if "items" in res:
        return res["items"][0]["snippet"]
    return "No Google result."

def ask_chatgpt(query):
    response = openai.ChatCompletion.create(
        model="gpt-4o",
        messages=[{"role":"system","content":"You are a helpful assistant."},
                  {"role":"user","content": query}]
    )
    return response.choices[0].message["content"]

def ask_gemini(query):
    # Example only: Gemini API requires proper SDK
    url = "https://gemini-api.google.com/v1/query"
    headers = {"Authorization": f"Bearer {GEMINI_API_KEY}"}
    data = {"question": query}
    res = requests.post(url, headers=headers, json=data)
    return res.json().get("answer", "No Gemini result.")

def ask_deepseek(query):
    # Example only: check DeepSeek docs for real endpoint
    url = "https://api.deepseek.com/v1/query"
    headers = {"Authorization": f"Bearer {DEEPSEEK_API_KEY}"}
    data = {"prompt": query}
    res = requests.post(url, headers=headers, json=data)
    return res.json().get("answer", "No DeepSeek result.")

# -----------------------------
# MAIN CHATBOT
# -----------------------------

def ai_orchestrator(query):
    print("\nCollecting answers...\n")
    sources = {
        "Google": search_google(query),
        "ChatGPT": ask_chatgpt(query),
        "Gemini": ask_gemini(query),
        "DeepSeek": ask_deepseek(query),
    }

    # Summarize final answer (using ChatGPT to merge answers)
    combined_text = "\n".join([f"{k}: {v}" for k,v in sources.items()])
    final_answer = ask_chatgpt(f"Summarize the best answer from these sources:\n{combined_text}")
    
    return final_answer, sources

# -----------------------------
# RUN CHAT LOOP
# -----------------------------
if __name__ == "__main__":
    print("Meta-AI Chatbot 🤖 (type 'exit' to quit)")
    while True:
        user = input("You: ")
        if user.lower() == "exit":
            print("Chatbot: Goodbye! 👋")
            break

        answer, details = ai_orchestrator(user)
        print("\nChatbot:", answer)
        print("\n🔎 Sources used:")
        for src, ans in details.items():
            print(f" - {src}: {ans[:100]}...")
