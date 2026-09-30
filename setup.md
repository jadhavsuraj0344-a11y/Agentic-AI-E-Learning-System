# LearnAI — Setup Guide

## Prerequisites

- Python 3.10+
- pip (Python package manager)

## Setup

```bash
# 1. Clone the repository
git clone <repo-url>
cd LearnAI

# 2. Create a virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate   # Linux/macOS
# venv\Scripts\activate    # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment variables
cp .env.example .env        # (or create .env manually — see below)
```

Edit `.env` and add your API keys:

```env
GROQ_API_KEY='gsk_...'
OPENROUTER_API_KEY='sk-or-v1-...'
```

> **Get API keys:**
> - **Groq** (primary LLM — chapter detection, summaries, MCQs): [console.groq.com](https://console.groq.com)
> - **OpenRouter** (fallback + evaluations): [openrouter.ai/keys](https://openrouter.ai/keys)

## Run

```bash
python app.py
```

The app starts at **http://127.0.0.1:5000**.

## How It Works

1. **Upload a PDF** or pick a built-in book (place PDFs in `books/`)
2. The app **detects chapters**, generates **summaries**, and extracts **topics**
3. For each topic, it creates **4 MCQs** — answer them and get evaluated
4. Track your **progress** on the analytics dashboard

## Project Structure

```
LearnAI/
├── app.py                   # Flask entry point (routes, session, STORE)
├── config.py                # API keys, model names, paths
├── requirements.txt         # Python dependencies
├── .env                     # API keys (not committed)
│
├── models/                  # Core AI modules
│   ├── pdf_reader.py        # PDF text extraction
│   ├── chapter_detector.py  # Regex → AI → chunk fallback
│   ├── llm.py               # Groq (primary) → OpenRouter (fallback)
│   ├── summarizer.py        # Chapter summaries
│   ├── topic_extractor.py   # Key topics per chapter
│   ├── mcq_generator.py     # 4 MCQs per topic
│   ├── evaluator.py         # Weak-topic detection
│   └── performance_analyzer.py  # SQL analytics
│
├── database_utils/          # Database layer
│   ├── db.py                # SQLite init
│   └── progress.py          # Save / query progress
│
├── utils/                   # Shared utilities
│   ├── helpers.py           # File validation, score display
│   └── prompts.py           # LLM prompt templates
│
├── templates/               # Jinja2 HTML pages
│   ├── base.html            # Layout + nav + bubble canvas
│   ├── index.html           # Landing (upload / chapters / progress / books / about)
│   ├── chapters.html        # Chapter grid
│   ├── summary.html         # Chapter summary
│   ├── quiz.html            # Topic explanation + MCQs
│   ├── result.html          # Score + answer review
│   └── progress.html        # Chart.js analytics dashboard
│
├── static/                  # Frontend assets
│   ├── style.css            # Dark neon theme
│   └── script.js            # Animations, tabs, quiz, Chart.js
│
├── uploads/                 # Uploaded PDFs (auto-created)
├── books/                   # Built-in PDFs (add your own)
├── database/                # SQLite DB location (auto-created)
└── elearning.db             # Runtime SQLite file
```
