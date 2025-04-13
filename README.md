# VidyAI++
🔷 Project Title:
VidyAI++ - Multilingual AI Tutoring & Mentorship Platform for BPL Government School Students
🔷 Selected Domain:
AIML
🔷 Problem Statement / Use Case:
Build an AI-powered, multilingual, and inclusive education platform that offers adaptive tutoring and real-time mentorship for underprivileged (BPL) students in Indian government schools, addressing barriers like language diversity, low literacy, lack of teachers, and limited internet connectivity.
VidyAI++ is an innovative AI-powered tutoring and mentorship platform tailored for underprivileged students in government schools across India. This initiative aligns with the National Education Policy (NEP) and aims to provide personalized, accessible education in regional languages through a dynamic web platform.

The platform functions as a multilingual academic assistant, employing Generative AI (GPT/Gemini) to create localized content and quizzes in various Indian languages, available in both text and voice formats. It utilizes reinforcement learning to adapt lesson difficulty based on individual student performance, offering personalized interventions such as explainer videos and alternative teaching methods.

To enhance personalization further, the system incorporates webcam-based emotion and fatigue detection using OpenCV and DeepFace, which assesses the student’s cognitive load and adjusts pacing in real time. Additionally, an AI-powered mentor matchmaking module connects students with suitable mentors (volunteers/NGOs), taking into account emotional states, subject requirements, and mentor availability.

VidyAI++ also features a Persona Intelligence Engine that classifies students as visual, auditory, or kinesthetic learners, adapting content formats to suit their learning styles. The platform gamifies the learning experience through streaks, badges, skill heatmaps, and micro-certifications, fostering engagement and motivation.

Moreover, the offline-first Progressive Web App (PWA) architecture ensures functionality in low or no internet zones, while a zero-literacy interface allows access via voice commands and visual cues, making it inclusive for non-readers. This comprehensive approach aims to empower underprivileged students, providing them with the tools and support needed to succeed academically and beyond.
🔷 Tech Stack Used:
Layer	and Technologies:
Frontend --	React (PWA), Tailwind CSS, React Router
Backend --	Node.js / FastAPI (Python)
Database --	MongoDB
AI/ML Models	OpenAI, TensorFlow
Vision AI -- OpenCV, DeepFace
Offline Capability --	PWA + IndexedDB + TensorFlow Lite
Speech & Language --	Whisper, Google TTS
Deployment	Vercel / Render (Frontend), Heroku (Backend)
..........................................................
**VidyAI++** is a multilingual AI-powered tutoring and mentorship platform designed for underprivileged government school students across India.

## 🌟 Features

- ✅ Multilingual AI Tutoring (GPT/Gemini)
- ✅ Personalized Learning with Reinforcement Learning
- ✅ Emotion & Fatigue Detection (OpenCV + DeepFace)
- ✅ AI Mentor Matching with ML
- ✅ Persona-Based Learning Style Adaptation
- ✅ Gamification: Streaks, Badges, Heatmaps
- ✅ Offline Support via PWA & TensorFlow Lite
- ✅ Zero Literacy Interface: Voice-based UI + Visual Cues
#code for backend
app.py
from flask import Flask, request, jsonify
from flask_cors import CORS
from models import db, User, QuizProgress
from quiz_generator import QuizGenerator
import json
import bcrypt

app = Flask(__name__)
CORS(app)
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///vidyai.db"
db.init_app(app)

quiz_generator = QuizGenerator()

@app.route("/login", methods=["POST"])
def login():
    data = request.json
    username = data.get("username")
    password = data.get("password")
    
    user = User.query.filter_by(username=username).first()
    if user and bcrypt.checkpw(password.encode("utf-8"), user.password.encode("utf-8")):
        return jsonify({"user_id": user.id})
    else:
        return jsonify({"error": "Invalid credentials"}), 401

@app.route("/generate_quiz", methods=["POST"])
def generate_quiz():
    data = request.json
    quiz = quiz_generator.generate_quiz(
        data["subject"], data["class_level"], data["language"]
    )
    return jsonify({"quiz": quiz, "difficulty": 1.0})

@app.route("/submit_answer", methods=["POST"])
def submit_answer():
    data = request.json
    user = User.query.get(data["user_id"])
    user.points += int(data["score"] * 0.1)
    progress = QuizProgress(
        user_id=data["user_id"],
        subject=data["subject"],
        class_level=data["class_level"],
        language=data["language"],
        score=data["score"],
        difficulty=1.0
    )
    db.session.add(progress)
    db.session.commit()
    return jsonify({"points": user.points, "badges": json.loads(user.badges)})

@app.route("/get_progress", methods=["GET"])
def get_progress():
    user_id = request.args.get("user_id")
    user = User.query.get(user_id)
    progress = QuizProgress.query.filter_by(user_id=user_id).all()
    return jsonify({
        "points": user.points,
        "badges": json.loads(user.badges),
        "history": [
            {
                "subject": p.subject,
                "class_level": p.class_level,
                "language": p.language,
                "score": p.score
            } for p in progress
        ]
    })

if __name__ == "__main__":
    with app.app_context():
        db.create_all()
        # Seed a demo user (for hackathon)
        if not User.query.filter_by(username="student1").first():
            hashed = bcrypt.hashpw("password123".encode("utf-8"), bcrypt.gensalt())
            demo_user = User(username="student1", password=hashed.decode("utf-8"))
            db.session.add(demo_user)
            db.session.commit()
    app.run(debug=True)
#models.py
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True, nullable=False)
    password = db.Column(db.String(100), nullable=False)  # Store hashed password
    points = db.Column(db.Integer, default=0)
    badges = db.Column(db.String(200), default="[]")

class QuizProgress(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey("user.id"), nullable=False)
    subject = db.Column(db.String(50), nullable=False)
    class_level = db.Column(db.Integer, nullable=False)
    language = db.Column(db.String(20), nullable=False)
    score = db.Column(db.Integer, nullable=False)
    difficulty = db.Column(db.Float, default=1.0)
#generator.py
from transformers import pipeline

def generate_quiz(prompt, lang='en'):
    generator = pipeline("text-generation", model="gpt2")
    output = generator(prompt, max_length=100, num_return_sequences=1)
    return output[0]['generated_text']
  #requirements.txt
  flask==2.3.2
flask-cors==4.0.0
transformers==4.35.0
torch
opencv-python
numpy
deepface
bcrypt==4.0.1
#code for backend
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#138d75" />
    <meta name="description" content="VidyAI++ Tutoring Platform" />
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <title>VidyAI++</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>

{
  "short_name": "VidyAI",
  "name": "VidyAI++ Tutoring Platform",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#4f46e5",
  "icons": [
    {
      "src": "favicon.ico",
      "sizes": "64x64",
      "type": "image/x-icon"
    },
    {
      "src": "icon.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
#quiz.js
import React, { useState, useEffect } from "react";
import axios from "axios";

function Quiz({ userId }) {
  const [quiz, setQuiz] = useState(null);
  const [subject, setSubject] = useState("Math");
  const [classLevel, setClassLevel] = useState(5);
  const [language, setLanguage] = useState("Hindi");
  const [answers, setAnswers] = useState([]);
  const [score, setScore] = useState(null);
  const [error, setError] = useState(null);

  const fetchQuiz = async () => {
    try {
      const response = await axios.post("http://localhost:5000/generate_quiz", {
        user_id: userId,
        subject,
        class_level: classLevel,
        language,
      });
      setQuiz(response.data.quiz);
      setAnswers([]);
      setScore(null);
      setError(null);
    } catch (err) {
      setError("Failed to load quiz. Please try again.");
      console.error(err);
    }
  };

  const submitAnswers = async () => {
    try {
      const correctAnswers = quiz.questions.map((q) => q.answer);
      const correct = answers.reduce(
        (acc, ans, i) => acc + (ans === correctAnswers[i] ? 1 : 0),
        0
      );
      const score = (correct / quiz.questions.length) * 100;
      await axios.post("http://localhost:5000/submit_answer", {
        user_id: userId,
        subject,
        class_level: classLevel,
        language,
        score,
      });
      setScore(score);
      setError(null);
    } catch (err) {
      setError("Failed to submit answers. Please try again.");
      console.error(err);
    }
  };

  useEffect(() => {
    fetchQuiz();
  }, [subject, classLevel, language]);

  return (
    <div className="quiz-container">
      <h2>Take a Quiz</h2>
      <div>
        <select onChange={(e) => setSubject(e.target.value)} value={subject}>
          <option>Math</option>
          <option>Science</option>
        </select>
        <select onChange={(e) => setClassLevel(Number(e.target.value))} value={classLevel}>
          <option value={5}>5</option>
          <option value={6}>6</option>
        </select>
        <select onChange={(e) => setLanguage(e.target.value)} value={language}>
          <option>Hindi</option>
          <option>English</option>
        </select>
      </div>
      <button onClick={fetchQuiz}>New Quiz</button>
      {error && <p style={{ color: "red" }}>{error}</p>}
      {quiz && (
        <div>
          {quiz.questions.map((q, i) => (
            <div key={i} className="quiz-question">
              <p>{q.question}</p>
              {q.options.map((opt, j) => (
                <button
                  key={j}
                  className="option-button"
                  onClick={() => {
                    const newAnswers = [...answers];
                    newAnswers[i] = opt;
                    setAnswers(newAnswers);
                  }}
                  style={{
                    backgroundColor: answers[i] === opt ? "#28a745" : "#007bff",
                  }}
                >
                  {opt}
                </button>
              ))}
            </div>
          ))}
          <button
            onClick={submitAnswers}
            disabled={answers.length !== quiz.questions.length}
          >
            Submit
          </button>
        </div>
      )}
      {score !== null && <p>Your score: {score}%</p>}
    </div>
  );
}

export default Quiz;
#progress.js
import React, { useState, useEffect } from "react";
import axios from "axios";
import { Line } from "react-chartjs-2";
import Chart from "chart.js/auto";

function Progress({ userId }) {
  const [progress, setProgress] = useState({ points: 0, badges: [], history: [] });

  useEffect(() => {
    axios
      .get(`http://localhost:5000/get_progress?user_id=${userId}`)
      .then((response) => setProgress(response.data))
      .catch((error) => console.error("Error fetching progress:", error));
  }, [userId]);

  const chartData = {
    labels: progress.history.map((p) => `${p.subject} ${p.class_level}`),
    datasets: [
      {
        label: "Scores",
        data: progress.history.map((p) => p.score),
        fill: false,
        borderColor: "rgb(75, 192, 192)",
        tension: 0.2,
      },
    ],
  };

  return (
    <div className="progress-container">
      <h2>Progress</h2>
      <p>Points: {progress.points}</p>
      <p>Badges: {progress.badges.join(", ")}</p>
      {progress.history.length > 0 ? (
        <Line data={chartData} />
      ) : (
        <p>No progress data available.</p>
      )}
    </div>
  );
}

export default Progress;
#voicenavigation
import React, { useEffect } from "react";

function VoiceNavigation({ setView }) {
  useEffect(() => {
    const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
    recognition.lang = "hi-IN"; // Default language set to Hindi

    recognition.onresult = (event) => {
      const command = event.results[0][0].transcript.toLowerCase();
      if (command.includes("quiz")) setView("quiz");
      if (command.includes("progress")) setView("progress");
    };

    recognition.start();

    return () => recognition.stop();
  }, [setView]);

  return <div>Listening for voice commands...</div>;
}

export default VoiceNavigation;
#App.js
import EmotionDetection from './components/EmotionDetection';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import StudentDashboard from './components/StudentDashboard';
import MentorMatch from './components/MentorMatch';
// ... other imports

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<StudentDashboard />} />
        <Route path="/emotion-detection" element={<EmotionDetection />} />
        <Route path="/mentor-match" element={<MentorMatch />} />
        {/* Add your other routes like /quiz, /progress etc. */}
      </Routes>
    </Router>
  );
}

export default App;
#index.js
import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";
import * as serviceWorker from "./serviceWorker";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// Enable service worker for offline support
serviceWorker.register();
#serviceworker.js
/* eslint-disable no-restricted-globals */

const CACHE_NAME = "vidyai-cache-v1";
const urlsToCache = ["/", "/index.html", "/static/js/main.chunk.js"];

globalThis.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(urlsToCache))
  );
});

globalThis.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});

globalThis.addEventListener("activate", (event) => {
  const cacheWhitelist = [CACHE_NAME];
  event.waitUntil(
    caches.keys().then((cacheNames) =>
      Promise.all(
        cacheNames.map((cacheName) => {
          if (!cacheWhitelist.includes(cacheName)) {
            return caches.delete(cacheName);
          }
          return null;
        })
      )
    )
  );
})
#package.json

  "name": "client",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "axios": "^1.8.4",
    "chart.js": "^4.4.8",
    "react": "^18.0.0",
    "react-chartjs-2": "^5.3.0",
    "react-dom": "^18.0.0",
    "react-router-dom": "^6.14.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}

