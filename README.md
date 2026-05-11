# API Schema Comparator

> Upload two URL files. Get a complete JSON diff report. No tokens, no LLM, no rate limits.

A deterministic, pure-Python tool built on FastAPI that compares old and new API endpoints side by side — field by field, type by type — and hands you a clean, downloadable report showing exactly what changed, what broke, and what passed.

---

## The Problem It Solves

When you migrate an API from v1 to v2, or deploy a new backend build, you need confidence that the response structure hasn't silently changed. Doing this manually across 40+ endpoints is slow, error-prone, and easy to get wrong.

This tool automates the entire comparison in seconds — no guessing, no hallucinations, just deterministic code telling you exactly what is different.

---

## How It Works

```
Old_URLs.txt ──┐
               ├──── Fetch both concurrently ────► Compare JSON responses ────► Download report
New_URLs.txt ──┘
```

Each URL in your old file is paired line-by-line with the corresponding URL in your new file. Both are fetched at the same time using async HTTP, and then the comparator walks through the entire JSON tree — checking keys, data types, empty values, array counts, and key ordering — and writes the results into a single timestamped `.txt` file.

---

## What the Comparator Checks

The comparison engine runs seven distinct checks on every JSON response:

| Check | What It Catches |
|---|---|
| Missing key | A field exists in OLD but is absent from NEW |
| Added key | NEW introduces a field that OLD did not have |
| Type mismatch | Same field, different type — e.g. `int` became `str` |
| Empty value mismatch | One side has data, the other has `""`, `null`, `[]`, or `{}` |
| Array count mismatch | Different number of items, with labels for which ones are missing or extra |
| Key order mismatch | Same keys, different sequence inside a dict |
| Array order mismatch | Same items, shuffled order inside a list |

It handles deeply nested JSON recursively, builds representative schemas from arrays to avoid false positives from different records, and deduplicates path-level issues so you only see each problem once.

---

## Getting Started

Clone the repo and set up your environment.

```bash
git clone https://github.com/your-username/api-schema-comparator.git
cd api-schema-comparator

python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate     # Mac / Linux

pip install -r requirements.txt
uvicorn app:app --reload
```

Then open `http://localhost:8000/docs` in your browser.

---

## Usage

Create two plain text files with your API endpoints — one per line, same count, same order.

**Old_URLs.txt**
```
https://api.example.com/v1/seasons/?token=xxx
https://api.example.com/v1/competitions?token=xxx
https://api.example.com/v1/competitions/123/matches?token=xxx
```

**New_URLs.txt**
```
https://api-new.example.com/v2/seasons/?token=xxx
https://api-new.example.com/v2/competitions?token=xxx
https://api-new.example.com/v2/competitions/123/matches?token=xxx
```

Lines starting with `#` are treated as comments and skipped. Both files must have the same number of URLs.

Go to `/docs`, upload both files under the `/compare` endpoint, and hit Execute. The comparison report downloads automatically.

---

## Sample Output

```
====================================================================
API SCHEMA COMPARISON REPORT
Generated   : 2026-05-11 14:30:00
Endpoints   : 42 OK  2 Errors
====================================================================

ENDPOINT 1/42
OLD  : https://api.example.com/v1/seasons/
NEW  : https://api-new.example.com/v2/seasons/
Status : PASS
====================================================================

ENDPOINT 2/42
OLD  : https://api.example.com/v1/competitions/
NEW  : https://api-new.example.com/v2/competitions/
Status : NEEDS FIX: 3 issue(s)
====================================================================

ADDED IN NEW (remove from NEW):
  - response.teams_win_percentage (type: array)

DATATYPE CHANGED:
  - response.total_items
    OLD: str  ->  NEW: int

EMPTY VALUE MISMATCH:
  - response.items[*].venue.timezone
    OLD: ''  ->  NEW: 'Local'

====================================================================
SUMMARY
====================================================================
Total endpoints  : 42
PASS             : 39
NEEDS FIX        : 3
Total issues     : 7
====================================================================
```

---

## Project Structure

```
api-schema-comparator/
│
├── app.py           FastAPI server — routes, file parsing, report delivery
├── scraper.py       Async HTTP client — fetches all URLs concurrently with httpx
├── comparator.py    JSON diff engine — the core comparison and report formatting logic
├── url.py           URL file reader — pairs old and new URLs line by line
├── main.py          CLI pipeline — run the full comparison without FastAPI
└── requirements.txt Python dependencies
```

---

## Running Without FastAPI

If you prefer running from the command line directly:

```bash
python main.py
```

Place your `Old_URL.txt` and `New_URL.txt` in the project root. The report is saved to `output/groqsummaries.txt`.

---

## Tech Stack

| Layer | Library |
|---|---|
| API Framework | FastAPI |
| HTTP Client | httpx (async) |
| ASGI Server | uvicorn |
| Language | Python 3.11 |

---

## Deploying on Render

This project is ready to deploy. Connect your GitHub repo to Render and use these settings:

```
Build Command  :  pip install -r requirements.txt
Start Command  :  uvicorn app:app --host 0.0.0.0 --port $PORT
Runtime        :  Python 3
```

---

## Security

Your URL files contain API tokens. Never push them to GitHub. Add this to your `.gitignore` before the first commit:

```
venv/
__pycache__/
*.pyc
.env
output/
Old_URL.txt
New_URL.txt
```

---

## Why Not Use an LLM?

This was an intentional choice. LLMs hallucinate, hit token limits, cost money per call, and give inconsistent results across runs. For a task like API schema comparison — which is purely structural and deterministic — a rule-based Python engine is faster, cheaper, 100% accurate, and infinitely scalable.

The comparator runs 42 endpoints in under a second. An LLM would take 12 minutes and burn through a daily token quota.

---

## Author

**Govind Singh** — Python Developer building tools that make API testing less painful.

If this saved you time, give it a star.
