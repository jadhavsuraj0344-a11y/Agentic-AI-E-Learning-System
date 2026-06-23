# LearnAI – AI-Powered Book Learning System

## Overview

LearnAI is an AI-powered eLearning platform that transforms PDF books into an interactive learning experience.

Instead of reading static PDFs, users can:

- Upload books
- Detect chapters automatically
- Generate chapter summaries
- Learn topic-by-topic
- Take AI-generated quizzes
- Track progress and performance
- Use built-in learning resources

The platform acts as a personalized AI teacher.

---

# Features

## PDF Upload

Upload any educational PDF and process it instantly.

Supported:
- Textbooks
- Research papers
- Study guides
- Notes

---

## Chapter Detection

The system automatically identifies chapters using:

### Regex-based detection

Detects patterns such as:

- Chapter 1
- CHAPTER 2
- Unit 3
- Section 4

### AI-based fallback

If regex detection fails, an LLM analyzes the document structure and extracts chapter titles.

---

## AI Summaries

For each chapter:

- AI generates concise summaries
- Important concepts are highlighted
- Complex content is simplified

---

## Topic Extraction

The AI extracts important topics from each chapter.

Example:

Chapter:
Discrete Probability

Topics:
- Sample Space
- Events
- Conditional Probability
- Bayes Theorem

---

## Topic Explanations

For every topic:

- AI-generated explanations
- Simplified learning
- Beginner-friendly format

---

## Dynamic MCQ Generation

The system generates:

- 4 MCQs per topic
- Content-based questions
- Real-time evaluation

---

## Progress Tracking

Stores:

- Chapter name
- Topic name
- Score
- Total questions
- Timestamp

Uses SQLite for persistence.

---

## Analytics Dashboard

Provides:

- Quiz count
- Average score
- Pass/Fail statistics
- Correct answers
- Wrong answers
- Topic-wise performance

---

## Built-in Books Library

Preloaded learning resources:

### Science
- Physics Basics
- Chemistry Fundamentals
- Biology Essentials

### Mathematics
- Discrete Mathematics
- Linear Algebra
- Calculus

### Medical
- Human Anatomy
- Physiology Basics
- Medical Terminology

### Sports
- Sports Science
- Football Tactics
- Sports Nutrition

### Economics
- Microeconomics
- Macroeconomics
- Business Economics

### Geography
- World Geography
- Physical Geography
- Environmental Geography

---

# Technology Stack

## Backend

- Python
- Flask

---

## Frontend

- HTML
- CSS
- JavaScript
- Jinja2

---

## AI Integration

- Groq API
- Llama 3.3 70B
- OpenRouter API

---

## Database

- SQLite

---

## PDF Processing

- PyPDF2 / pdfplumber

---

## Utilities

- Requests
- Regex
- JSON
- Python-dotenv

---

# Project Architecture

User
↓
Frontend (HTML/CSS/JS)
↓
Flask Backend
↓
PDF Processing
↓
Chapter Detection
↓
AI Processing
↓
Topic Generation
↓
MCQ Generation
↓
SQLite Storage
↓
Analytics Dashboard

---

# Folder Structure

```text
ai-elearning-system/

│
├── app.py
├── config.py
├── requirements.txt
│
├── models/
│   ├── llm.py
│   ├── chapter_detector.py
│   ├── mcq_generator.py
│   ├── topic_extractor.py
│   └── summarizer.py
│
├── database_utils/
│   ├── db.py
│   └── progress.py
│
├── templates/
│   ├── base.html
│   ├── index.html
│   ├── chapter.html
│   ├── quiz.html
│   ├── result.html
│   └── progress.html
│
├── static/
│   ├── style.css
│   └── script.js
│
├── uploads/
│
├── books/
│   ├── science/
│   ├── maths/
│   ├── medical/
│   ├── sports/
│   ├── economics/
│   └── geography/
│
└── database/
    └── elearning.db
```

---

# Installation

## Clone Repository

```bash
git clone <repository-url>
cd ai-elearning-system
```

## Create Virtual Environment

```bash
python -m venv venv
```

Activate:

Windows

```bash
venv\Scripts\activate
```

Mac/Linux

```bash
source venv/bin/activate
```

---

## Install Dependencies

```bash
pip install -r requirements.txt
```

---

# Environment Variables

Create a `.env` file:

```env
GROQ_API_KEY=your_groq_api_key
OPENROUTER_API_KEY=your_openrouter_api_key
```

---

# Running the Project

Start the Flask application:

```bash
python app.py
```

Open:

```text
http://127.0.0.1:5000
```

or

```text
http://localhost:5000
```

---

# Workflow

1. Upload PDF
2. Extract text
3. Detect chapters
4. Generate summary
5. Extract topics
6. Explain topic
7. Generate MCQs
8. Submit quiz
9. Store score
10. Display analytics

---

# Challenges Solved

## Chapter Over-Detection

Problem:
- Too many chapters detected

Solution:
- Improved regex filtering
- Added AI fallback

---

## MCQ Parsing Issues

Problem:
- Invalid JSON
- Incorrect answer mapping

Solution:
- Structured prompting
- JSON validation
- Regex extraction

---

## Database Schema Mismatch

Problem:

```text
sqlite3 OperationalError
```

Solution:
- Updated schema
- Added score tracking fields
- Recreated database

---

## PDF Formatting Issues

Problem:
- Inconsistent extraction

Solution:
- Text cleanup
- Whitespace normalization
- Chapter title sanitization

---

# Future Enhancements

- User Authentication
- Cloud Deployment
- Vector Database
- RAG-Based Learning
- Voice Tutor
- Personalized Difficulty
- Resume Learning
- AI Learning Analytics

---

# Author

AI/ML Internship Project

Built using:
- Flask
- Groq
- OpenRouter
- SQLite
- Llama 3.3
