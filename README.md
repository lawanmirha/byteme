# byteme
from flask import Flask, render_template, request, jsonify, redirect, session
from flask_cors import CORS
from dotenv import load_dotenv
import openai
import os
import jwt
from datetime import datetime, timedelta

from google.oauth2 import id_token
from google.auth.transport import requests as grequests

load_dotenv()

app = Flask(__name__)
CORS(app)
app.secret_key = os.getenv("JWT_SECRET_KEY")

openai.api_key = os.getenv("OPENAI_API_KEY")
GOOGLE_CLIENT_ID = os.getenv("GOOGLE_CLIENT_ID")

# Custom replies
def custom_response(msg):
    msg = msg.lower().strip()
    if msg in ["hi", "hey", "hello"]:
        return "yoii 😋"
    elif msg == "no":
        return "naurrr 😭"
    elif msg == "go":
        return "gaurrrrrrr 🐍"
    return None

@app.route('/')
def home():
    return render_template("index.html")

@app.route('/login', methods=["POST"])
def login():
    token = request.json.get("credential")
    try:
        idinfo = id_token.verify_oauth2_token(token, grequests.Request(), GOOGLE_CLIENT_ID)
        session["user"] = {
            "name": idinfo["name"],
            "email": idinfo["email"],
            "picture": idinfo["picture"]
        }
flask
flask_cors
openai
python-dotenv
google-auth
<!DOCTYPE html>
<html>
<head>
  <title>ByteMe AI</title>
  <style>
    body {
      background: #121212;
      color: #fff;
      font-family: 'Segoe UI', sans-serif;
      padding: 0;
      margin: 0;
    }
    .chat-container {
      width: 90%;
      max-width: 700px;
      margin: 60px auto;
    }
    .bubble {
      padding: 12px 16px;
      border-radius: 20px;
      margin: 10px;
      display: inline-block;
      max-width: 75%;
    }
    .user {
      background-color: #4a90e2;
      float: right;
      clear: both;
    }
    .bot {
      background-color: #e94e77;
      float: left;
      clear: both;
    }
    .input-area {
      width: 90%;
      max-width: 700px;
      margin: 20px auto;
      display: flex;
    }
    input {
      flex: 1;
      padding: 12px;
      border-radius: 20px;
      border: none;
      font-size: 16px;
    }
    button {
      margin-left: 10px;
      padding: 12px 16px;
      border-radius: 20px;
      border: none;
      background: #e94e77;
      color: #fff;
      font-size: 16px;
      cursor: pointer;
    }
    .genre {
      margin-top: 10px;
    }
    .logout-btn {
      position: absolute;
      right: 20px;
      top: 20px;
      background: #fff;
      color: #e94e77;
      border: 2px solid #e94e77;
      border-radius: 16px;
      padding: 6px 12px;
      cursor: pointer;
    }
  </style>
  <script src="https://accounts.google.com/gsi/client" async defer></script>
</head>
<body>
  <button class="logout-btn" onclick="logout()">Logout</button>
  <div class="chat-container" id="chat"></div>

  <div class="input-area">
    <input id="msg" type="text" placeholder="come over and byte me 😘">
    <button onclick="send()">Send</button>
  </div>

  <div class="chat-container genre">
    <label for="genre">Pick your vibe:</label>
    <select id="genre">
      <option value="romantic">Romantic 💖</option>
      <option value="sassy">Sassy 💅</option>
      <option value="teasing">Teasing 😈</option>
      <option value="friend">Friend 🧸</option>
      <option value="anime">Anime 🌸</option>
    </select>
  </div>

  <div id="g_id_onload"
    data-client_id="YOUR_GOOGLE_CLIENT_ID"
    data-context="signin"
    data-ux_mode="popup"
    data-callback="onSignIn">
  </div>
  <div class="g_id_signin" data-type="standard"></div>

  <script>
    let user = null;

    function addMessage(msg, sender) {
      const div = document.createElement("div");
      div.className = "bubble " + sender;
      div.innerText = msg;
      document.getElementById("chat").appendChild(div);
      div.scrollIntoView({ behavior: 'smooth' });
    }

    function send() {
      const msg = document.getElementById("msg").value;
      const genre = document.getElementById("genre").value;
      if (!msg) return;
      addMessage(msg, "user");
      document.getElementById("msg").value = "";
      fetch("/chat", {
        method: "POST",
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({ message: msg, genre: genre })
      }).then(res => res.json()).then(data => {
        if (data.response) addMessage(data.response, "bot");
        else addMessage("Something went wrong!", "bot");
      });
    }

    function onSignIn(response) {
      fetch("/login", {
        method: "POST",
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({ credential: response.credential })
      }).then(res => res.json()).then(data => {
        if (data.success) {
          document.querySelector(".g_id_signin").style.display = "none";
        } else {
          alert("Login failed!");
        }
      });
    }

    function logout() {
      fetch("/logout", { method: "POST" }).then(() => {
        location.reload();
      });
    }
  </script>
</body>
</html>
