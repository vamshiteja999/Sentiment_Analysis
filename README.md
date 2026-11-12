# Speech & Text Sentiment Analysis (FastAPI + Google Cloud AI)

A web app that analyzes the sentiment of **typed text or recorded speech**. Speech is recorded in the browser, transcribed with Google Cloud Speech-to-Text, scored with the Google Cloud Natural Language API, and saved so past results and recordings can be revisited. It can also read text aloud with Text-to-Speech.

![Python](https://img.shields.io/badge/Python-3.10+-blue) ![FastAPI](https://img.shields.io/badge/FastAPI-backend-009688) ![Google Cloud](https://img.shields.io/badge/Google%20Cloud-AI%20APIs-4285F4)

## Features

| Feature | How it works |
|---|---|
| Text sentiment | Natural Language API → score (−1 to 1) + magnitude → Positive / Neutral / Negative |
| Speech sentiment | Browser `MediaRecorder` (WebM/Opus) → Speech-to-Text → sentiment |
| Text-to-speech | Text-to-Speech API (MP3, en-US neutral voice), returned as base64 |
| History | Every result is stored as JSON with a UUID and timestamp, newest first |
| Audio playback | Recordings are stored and can be replayed from the history view |

## Architecture

```
Browser (static/index2.html, Bootstrap + axios)
   │  text / recorded audio
   ▼
FastAPI (app.py)
   ├── /analyze-text ────────► Cloud Natural Language API
   ├── /speech-to-text ──────► Cloud Speech-to-Text API
   ├── /analyze-speech ──────► Cloud Natural Language API
   ├── /text-to-speech ──────► Cloud Text-to-Speech API
   └── /get-history, /get-result/{id}, /get-audio/{file}
          │
          ▼
   sentiment_results/*.json   audio_files/*.webm
```

**Sentiment thresholds:** score > 0.25 → Positive, score < −0.25 → Negative, otherwise Neutral.

## Setup

**Prerequisites:** Python 3.10+, a Google Cloud project with the **Natural Language**, **Speech-to-Text** and **Text-to-Speech** APIs enabled.

```bash
git clone https://github.com/vamshiteja999/Sentiment_Analysis.git
cd Sentiment_Analysis
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

**Credentials.** Use either option. Keep key files outside the repo; `.gitignore` blocks `*.json` as a safety net.

```bash
# Option A — your own user credentials (easiest for local dev)
gcloud auth application-default login

# Option B — a service-account key stored outside the repo
export GOOGLE_APPLICATION_CREDENTIALS="$HOME/.gcp/sentiment-sa.json"
```

**Run:**

```bash
python app.py            # or: uvicorn app:app --reload
```

Open http://127.0.0.1:8000.

## API

| Method | Endpoint | Body | Returns |
|---|---|---|---|
| POST | `/analyze-text` | form: `text` | `{sentiment, score, magnitude, result_id}` |
| POST | `/text-to-speech` | form: `text` | `{audio}` (base64 MP3) |
| POST | `/speech-to-text` | multipart: `file` (WebM/Opus, 48 kHz) | `{transcript, audio_filename}` |
| POST | `/analyze-speech` | JSON: `{transcript, audio_filename}` | `{sentiment, score, magnitude, result_id}` |
| GET | `/get-history` | – | list of saved results |
| GET | `/get-result/{result_id}` | – | one saved result |
| GET | `/get-audio/{filename}` | – | the WebM recording |

Interactive API docs are available at http://127.0.0.1:8000/docs.

## Project structure

```
app.py              FastAPI backend
static/index2.html  Single-page frontend
requirements.txt
```

## Security notes

- Credentials come from the environment and are never stored in code.
- File endpoints reject path-traversal attempts such as `../`.
- There is no user authentication. Add it before deploying publicly.

## Possible improvements

- Per-user history with authentication
- Multi-language support (the Natural Language API supports many languages)
- Store results in a database (e.g. Firestore) instead of local JSON files
- Entity-level sentiment and charts of sentiment over time
