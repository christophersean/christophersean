<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AI Receptionist</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>

<header>
    <div class="logo">AI Receptionist</div>
</header>

<section class="hero">
    <h1>Welcome to Our AI Receptionist</h1>
    <p>Your smart assistant for answering questions, booking appointments, and helping visitors.</p>
</section>

<section class="chat-section">
    <h2>Chat With Our Receptionist</h2>

    <div id="chatbox">
        <div id="messages"></div>

        <div class="input-row">
            <input id="input" type="text" placeholder="Ask something...">
            <button onclick="sendMessage()">Send</button>
        </div>
    </div>
</section>

<footer>© 2025 AI Receptionist. All rights reserved.</footer>

<script src="script.js"></script>
</body>
</html>
body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f5f5f5;
}

header {
    background: #222;
    color: white;
    padding: 15px 20px;
    font-size: 20px;
}

.hero {
    text-align: center;
    padding: 60px 20px;
    background: #0077cc;
    color: white;
}

.chat-section {
    max-width: 700px;
    margin: 40px auto;
    background: white;
    padding: 20px;
    border-radius: 8px;
}

#chatbox {
    height: 450px;
    display: flex;
    flex-direction: column;
}

#messages {
    border: 1px solid #ccc;
    border-radius: 6px;
    flex-grow: 1;
    padding: 10px;
    overflow-y: auto;
    margin-bottom: 12px;
    background: #fafafa;
}

.msg {
    padding: 8px 12px;
    margin: 10px 0;
    border-radius: 6px;
    max-width: 80%;
}

.me {
    background: #d1e7ff;
    align-self: flex-end;
}

.ai {
    background: #eaeaea;
    align-self: flex-start;
}

.input-row {
    display: flex;
    gap: 10px;
}

input {
    flex: 1;
    padding: 10px;
    font-size: 16px;
    border-radius: 6px;
    border: 1px solid #aaa;
}

button {
    padding: 10px 16px;
    background: #0077cc;
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
}
async function sendMessage() {
    const input = document.getElementById("input");
    const message = input.value.trim();
    if (!message) return;

    addMessage(message, "me");

    const response = await fetch("http://localhost:8000/ask", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ message })
    });

    const data = await response.json();
    addMessage(data.reply, "ai");

    input.value = "";
}

function addMessage(text, type) {
    const messages = document.getElementById("messages");
    const div = document.createElement("div");
    div.className = `msg ${type}`;
    div.textContent = text;
    messages.appendChild(div);
    messages.scrollTop = messages.scrollHeight;
}
from fastapi import FastAPI
from pydantic import BaseModel
from fastapi.middleware.cors import CORSMiddleware
import openai

openai.api_key = "YOUR_OPENAI_API_KEY"

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

class UserMessage(BaseModel):
    message: str

@app.post("/ask")
async def ask(data: UserMessage):
    response = openai.ChatCompletion.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "You are a friendly, helpful AI receptionist."},
            {"role": "user",   "content": data.message}
        ]
    )
    return {"reply": response.choices[0].message["content"]}
uvicorn server:app --reload --port 8000

