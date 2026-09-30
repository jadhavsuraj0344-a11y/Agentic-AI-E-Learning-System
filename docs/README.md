# LearnAI — AI-Powered Book Learning System

## Overview

LearnAI is an AI-powered eLearning platform that transforms static PDF books into an interactive, personalized learning experience. Instead of passively reading a PDF, a user uploads a book and the system automatically extracts text, detects chapters, generates AI summaries, extracts key topics, explains each topic in beginner-friendly terms, creates content-grounded quizzes, evaluates answers, and tracks progress on an analytics dashboard.

It acts as a **personal AI teacher** — no human instructor required.

---

## Features

### Core Learning Pipeline
- **PDF Upload** — Drag-and-drop or click-to-select; supports textbooks, research papers, study guides, and notes.
- **Automatic Chapter Detection** — Three-tier strategy:
  1. **Regex detection** — matches `Chapter N`, `CHAPTER N`, `Unit N`, `Section N`, and numbered headings.
  2. **AI fallback** — if regex finds fewer than 2 chapters, an LLM analyzes the first 4,000 characters and returns a JSON array of chapter titles.
  3. **Chunk fallback** — if AI also fails, the text is split into 500-word sections.
- **AI Chapter Summaries** — each chapter is truncated to 5,000 chars and summarized in 3 short paragraphs.
- **Topic Extraction** — 4–6 key topics extracted from each summary as a JSON array, with a plain-text fallback.
- **Topic Explanations** — beginner-friendly explanation with real-world analogies and a "Key takeaway" sentence.
- **Dynamic MCQ Generation** — 4 MCQs per topic, grounded in the actual chapter text, with randomized focus hints (definitions, applications, comparisons, causes/effects, processes, pros/cons) so questions vary each call.
- **Quiz Evaluation** — server-side scoring with per-question answer review (correct/wrong highlighting).
- **Progress Tracking** — every quiz result (chapter, topic, score, total, timestamp) is persisted in SQLite.
- **Analytics Dashboard** — stats cards (quizzes done, average score, pass count, chapters covered), doughnut charts (pass/fail, score bands), a horizontal bar chart (per-topic scores), and a detailed results table.

### Built-in Books Library
Preloaded PDFs across 6 categories (Science, Mathematics, Medical, Sports, Economics, Geography) accessible via the `/books/<path>` route.

### UI/UX
- Dark neon-themed design (cyan `#00d4ff` + purple `#7b5ff5` accents on black).
- Animated bubble canvas background (floating neon/purple orbs with radial gradients).
- Glassmorphism cards with backdrop blur.
- Tabbed landing page (Upload, Chapters, Progress, Default Books, About).
- Loading overlay with spinner during AI processing.
- Staggered card entrance animations.
- Fully responsive (mobile breakpoint at 720px).

---

## Technology Stack

| Layer | Technology |
|---|---|
| **Backend** | Python 3, Flask |
| **Frontend** | HTML, CSS (custom dark/neon theme), Vanilla JavaScript, Jinja2 templating |
| **Charts** | Chart.js 4.4.0 (via CDN) |
| **Primary AI** | Groq API — Llama 3.3 70B (`llama-3.3-70b-versatile`) |
| **Fallback AI** | OpenRouter API — Mistral 7B Instruct (`mistralai/mistral-7b-instruct`) |
| **Database** | SQLite |
| **PDF Processing** | PyPDF2 |
| **Utilities** | `requests`, `python-dotenv`, `re`, `json` |

**Dependencies** (`requirements.txt`):
```
flask
PyPDF2
requests
python-dotenv
```

---

## Architecture & Data Flow

```
User → Frontend (HTML/CSS/JS) → Flask Backend → PDF Processing →
Chapter Detection → AI Summary → Topic Extraction → Topic Explanation →
MCQ Generation → Quiz Submission → SQLite Storage → Analytics Dashboard
```

### In-Memory State
The app uses a global `STORE` dictionary to hold session state (no per-user isolation):
```python
STORE = {
    "book_text":      "",   # full extracted PDF text
    "chapters":       {},   # {chapter_title: chapter_text}
    "chapter_topics": {},   # {chapter_title: [topic1, topic2, ...]}
}
```
Flask `session` is used for `book_name`, `chapter_name`, and `topics` list.

### Two-Tier LLM Routing (`llm.py`)
```
ask_groq()  ──→ Groq API (Llama 3.3 70B, ~1-2s)
                 │
                 ├─ if GROQ_API_KEY missing or request fails
                 ↓
              ask_llm()  ──→ OpenRouter API (Mistral 7B, ~8-15s)
```
- `ask_groq_json()` / `ask_llm_json()` wrappers append "Return ONLY valid JSON" to the system prompt and parse the response through `_clean_json()`, which strips markdown fences and uses regex extraction as a fallback.
- All AI modules call `ask_groq` or `ask_groq_json` as their primary path.
- Timeouts: 30s for Groq, 90s for OpenRouter.

---

## Flask Routes

| Route | Method | Purpose |
|---|---|---|
| `/` | GET | Landing page with tabbed UI (upload, chapters, progress, books, about) |
| `/upload` | POST | Accept PDF upload, extract text, detect chapters, store in `STORE` |
| `/books/<path:book_path>` | GET | Load a preloaded book from the `books/` directory |
| `/chapters` | GET | Standalone chapter listing page |
| `/chapter/<chapter_name>` | GET | Generate summary + extract topics for a chapter |
| `/topic/<topic_name>` | GET | Generate explanation + 4 MCQs for a topic |
| `/submit_quiz` | POST | Evaluate answers, save progress to SQLite, show results |
| `/progress` | GET | Standalone progress dashboard |

---

## Database Schema (`db.py`)

```sql
CREATE TABLE IF NOT EXISTS progress (
    id        INTEGER PRIMARY KEY AUTOINCREMENT,
    chapter   TEXT,
    topic     TEXT,
    score     INTEGER,
    total     INTEGER,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
)
```

### Scoring Logic (`helpers.py`)
- **≥80%**: `success` (green) + 🎉
- **≥60%**: `warning` (amber) + 👍
- **<60%**: `danger` (red) + 💪
- Pass threshold: 60%

---

## Project Structure

The code is organized into packages as referenced by the imports in `app.py`:

```text
LearnAI/
├── app.py                      # Flask app — routes, session, in-memory STORE
├── config.py                   # API keys, model names, URLs, paths
├── requirements.txt            # Python dependencies
├── .env                        # API keys (not committed)
│
├── models/                     # AI processing modules
│   ├── llm.py                  # Two-tier LLM routing (Groq → OpenRouter)
│   ├── pdf_reader.py           # PyPDF2 text extraction + cleanup
│   ├── chapter_detector.py     # Regex + AI + chunk fallback chapter detection
│   ├── summarizer.py           # Chapter summary generation
│   ├── topic_extractor.py      # Topic extraction from summaries
│   ├── mcq_generator.py        # MCQ generation with 3-tier fallback
│   ├── evaluator.py            # Weak-topic detection + re-explanation
│   └── performance_analyzer.py # Chapter/all-performance DB queries
│
├── database_utils/             # Database layer
│   ├── db.py                   # init_db() — creates progress table
│   └── progress.py             # save_progress(), get_all_progress()
│
├── utils/                      # Shared utilities
│   ├── helpers.py              # allowed_file(), score_color(), score_emoji()
│   └── prompts.py              # System/user prompt templates
│
├── templates/                  # Jinja2 HTML templates
│   ├── base.html               # Layout: nav, loading overlay, bubble canvas
│   ├── index.html              # Landing page with 5 tabs
│   ├── chapters.html           # Chapter grid listing
│   ├── summary.html            # Chapter summary + topic learning path
│   ├── quiz.html               # Topic explanation + MCQ form
│   ├── result.html             # Score circle + answer review
│   └── progress.html           # Analytics dashboard with charts
│
├── static/                     # Frontend assets
│   ├── style.css               # Dark neon theme
│   └── script.js               # Bubbles, tabs, upload, quiz, charts
│
├── uploads/                    # User-uploaded PDFs (created at runtime)
├── books/                      # Preloaded PDFs by category
│   ├── science/
│   ├── maths/
│   ├── medicine/
│   ├── sports/
│   ├── economics/
│   └── geography/
└── database/
    └── elearning.db            # SQLite database (created at runtime)
```

> **Note:** `app.py` imports from `models.*`, `database_utils.*`, and `utils.*`, and Flask resolves templates from `templates/` and static files from `static/`. Ensure these directories exist (with `__init__.py` in each package) for the app to run.

---

## Key Modules

### `llm.py` — Two-Tier LLM Router
- `ask_groq(prompt, system, temperature, max_tokens)` → Groq API (primary). Falls back to `ask_llm()` if the key is missing or the request fails.
- `ask_llm(prompt, system, temperature, max_tokens)` → OpenRouter API (fallback).
- `ask_groq_json()` / `ask_llm_json()` → wrappers that enforce JSON-only output and parse via `_clean_json()`.

### `chapter_detector.py` — Chapter Detection
- `detect_chapters(text)` → tries regex first; if <2 chapters found, falls back to AI; if AI fails, splits into 500-word chunks.

### `mcq_generator.py` — MCQ Generation
- `generate_mcqs(topic_name, chapter_context, count=4)` → generates content-grounded MCQs with 6 randomized focus hints.
- `_validate()` → normalizes options to A/B/C/D format.
- `_text_fallback()` → plain-text MCQ format with regex parsing.
- `_emergency_mcqs()` → last-resort generic but structurally valid questions.

### `evaluator.py` — Weak Topic Re-explanation
- `get_weak_topics(topic_scores)` → returns topics scoring below 60%.
- `generate_reexplanation(topic_name, wrong_questions)` → re-explains a weak topic in simpler terms using the fallback LLM.

### `prompts.py` — Prompt Templates
- `TOPIC_EXPLANATION_SYSTEM` — "Brilliant, enthusiastic teacher" persona.
- `TOPIC_EXPLANATION_PROMPT` — structured explanation with core idea, examples, and key takeaway.
- `FINAL_TEST_PROMPT` — 20-question chapter test template (defined but not yet wired to a route).

---

## Setup & Installation

### Prerequisites
- Python 3.8+
- A Groq API key (free tier available) — [console.groq.com](https://console.groq.com)
- An OpenRouter API key (fallback) — [openrouter.ai](https://openrouter.ai)

### Installation

```bash
# 1. Clone the repository
git clone <repository-url>
cd LearnAI

# 2. Create and activate a virtual environment
python -m venv venv
# Windows:
venv\Scripts\activate
# Mac/Linux:
source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment variables (see below)

# 5. Run the application
python app.py
```

### Access
Open `http://127.0.0.1:5000` or `http://localhost:5000` in your browser.

---

## Environment Variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key
OPENROUTER_API_KEY=your_openrouter_api_key
```

Both keys are loaded via `python-dotenv` in `config.py`. If `GROQ_API_KEY` is empty, the system automatically routes all requests to OpenRouter.

---

## Workflow

1. Upload PDF (or select a built-in book)
2. Extract text via PyPDF2
3. Detect chapters (regex → AI → chunk fallback)
4. Generate AI summary per chapter
5. Extract 4–6 key topics per chapter
6. Explain each topic in beginner-friendly terms
7. Generate 4 content-grounded MCQs per topic
8. Submit quiz for server-side evaluation
9. Store score in SQLite
10. Display analytics dashboard

---

## Challenges Solved

| Challenge | Solution |
|---|---|
| Chapter over-detection | Improved regex filtering + AI fallback |
| MCQ parsing issues (invalid JSON, wrong answer mapping) | Structured prompting, JSON validation, regex extraction |
| Database schema mismatch (`sqlite3 OperationalError`) | Updated schema with score tracking fields, recreated database |
| PDF formatting issues (inconsistent extraction) | Text cleanup, whitespace normalization, chapter title sanitization |

---

## Future Enhancements

- User Authentication
- Cloud Deployment
- Vector Database + RAG-Based Learning
- Voice Tutor
- Personalized Difficulty
- Resume Learning
- AI Learning Analytics

---

## Author

AI/ML Internship Project — built with Flask, Groq, OpenRouter, SQLite, and Llama 3.3.
