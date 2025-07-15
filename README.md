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
