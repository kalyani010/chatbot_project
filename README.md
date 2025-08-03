# chatbot_project
from flask import Flask, render_template, request
import speech_recognition as sr
from gtts import gTTS
import playsound
import os
import uuid

app = Flask(__name__)

def get_response(user_input):
    user_input = user_input.lower()
    if user_input in ["hi", "hello", "hey"]:
        return "Hi! 😊"
    elif user_input in ["how are you", "how are you?"]:
        return "I'm fine, thanks!"
    elif user_input in ["bye", "exit", "goodbye"]:
        return "Goodbye! 👋"
    elif user_input in ["what is your name", "who are you"]:
        return "I'm a voice-enabled Python chatbot!"
    else:
        return "Sorry, I didn't understand that."

def speak(text):
    tts = gTTS(text=text, lang='en')
    filename = f"voice_{uuid.uuid4()}.mp3"
    tts.save(filename)
    playsound.playsound(filename)
    os.remove(filename)

@app.route("/", methods=["GET", "POST"])
def chat():
    response = ""
    if request.method == "POST":
        user_input = request.form["message"]
        response = get_response(user_input)
        speak(response)
    return render_template("index.html", response=response)

@app.route("/voice", methods=["POST"])
def voice_input():
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        audio = recognizer.listen(source)

    try:
        user_input = recognizer.recognize_google(audio)
        print("You said:", user_input)
        response = get_response(user_input)
        speak(response)
        return render_template("index.html", response=response)
    except:
        return render_template("index.html", response="Sorry, I didn't catch that.")

if __name__ == "__main__":
    app.run(debug=True)
