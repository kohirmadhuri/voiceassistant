# voiceassistant
import speech_recognition as sr
import pyttsx3
import datetime
import webbrowser
import os
from pathlib import Path

# Initialize the text-to-speech engine
engine = pyttsx3.init()

def speak(text):
    """Convert text to speech."""
    engine.say(text)
    engine.runAndWait()

def greet_user():
    """Greet the user based on the time of day."""
    hour = datetime.datetime.now().hour
    if 0 <= hour < 12:
        speak("Good morning!")
    elif 12 <= hour < 18:
        speak("Good afternoon!")
    else:
        speak("Good evening!")
    speak("How can I assist you?")

def listen_command():
    """Listen for user commands and convert speech to text."""
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        recognizer.adjust_for_ambient_noise(source)  # Adjust for background noise
        try:
            audio = recognizer.listen(source, timeout=5)
            command = recognizer.recognize_google(audio).lower()
            print(f"User said: {command}")
            return command
        except sr.UnknownValueError:
            speak("Sorry, I didn't catch that. Could you repeat?")
        except sr.RequestError:
            speak("There seems to be an issue with the speech recognition service.")
        except sr.WaitTimeoutError:
            speak("You didn't say anything.")
        return ""

def process_command(command):
    """Process the command and perform tasks."""
    if "time" in command:
        current_time = datetime.datetime.now().strftime("%I:%M %p")
        speak(f"The current time is {current_time}")
    elif "search" in command or "google" in command:
        speak("What should I search for?")
        search_query = listen_command()
        if search_query:
            url = f"https://www.google.com/search?q={search_query}"
            webbrowser.open(url)
            speak(f"Here are the search results for {search_query}")
    elif "open" in command:
        if "youtube" in command:
            webbrowser.open("https://www.youtube.com")
            speak("Opening YouTube.")
        elif "google" in command:
            webbrowser.open("https://www.google.com")
            speak("Opening Google.")
    elif "play music" in command:
        music_dir = Path.home() / "Music"  # Dynamically get the user's Music directory
        if music_dir.exists():
            songs = os.listdir(music_dir)
            if songs:
                os.startfile(os.path.join(music_dir, songs[0]))  # Play the first song
                speak("Playing music.")
            else:
                speak("No music files found in the Music directory.")
        else:
            speak("Music directory not found.")
    elif "exit" in command or "quit" in command:
        speak("Goodbye!")
        exit()
    else:
        speak("I can’t help with that right now, but I'm learning!")

if __name__ == "__main__":
    greet_user()
    while True:
        command = listen_command()
        if command:
            process_command(command)
