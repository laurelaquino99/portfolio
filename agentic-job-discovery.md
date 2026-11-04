# Job Search Assistant

A personal **agentic job search and application assistant**. It helps you discover roles and companies, scrape and score job postings, draft application answers from your own history, and prepare for interviews — all from a local UI, with no multi-user deployment.

See [PROJECT_PLAN.md](./PROJECT_PLAN.md) for the full design.

## What it does (intended end state)

The product has three main surfaces in the UI, backed by five agents and a personal knowledge base (resume, projects, past applications, preferences).

### 1. Discover + job search

Move from exploring career directions → defining search targets → running scrapers → reviewing results.

- **Discovery workspace:** Describe interests, skills, industries, and things to avoid. An LLM suggests roles and companies with short reasoning. Approve or reject each suggestion (rejects are remembered so regeneration does not repeat them).
- **Target lists:** Persistent roles and companies you actually want to search. Add or remove items by hand. Companies can store an optional job-board URL (Greenhouse, Lever, Ashby, etc.) for direct scraping.
- **Job search (two modes):**
  - **Role-based:** Pick target roles, location, and max jobs per run, then scrape matching postings.
  - **Company boards:** Select companies (using stored board URLs) or paste a board URL and scrape that board.
- **Scraped jobs:** Table of results with experience fit, interest fit, application effort, star, and applied. Open a job for the full description, score reasoning, and application link.

### 2. Application assistant

For a selected job and a pasted application question:

- Find similar past answers (retrieval over your stored Q&A).
- Generate a draft in your tone, grounded in resume + project chunks.
- Edit in a text editor.
- Evaluate answer strength (qualitative: weak → great) with reasoning and suggestions.

### 3. Interview prep

For a selected job:

- Generate technical, behavioral, and company-specific questions.
- Expand each question for personalized talking points: a recommended example from your experience, why it is strong, and key points to cover.

Mock interview mode is out of scope for now.

### Personalization and storage

Agents read a **local knowledge base**: resume, project writeups, past cover letters and application answers, plus structured files:

| File | Purpose |
| --- | --- |
| `data/jobs.json` | Scraped jobs, scores, star/applied, notes |
| `data/preferences.json` | Accepted/rejected roles and companies, board URLs, interest inputs |
| `data/application_history.json` | Submitted answers and apply dates |
| `data/documents/` | Markdown/text sources |
| `vector_store/` | Local embeddings (FAISS or Chroma) for retrieval |

All LLM outputs are structured (JSON) with transparent reasoning stored on the job or answer.

## Tech stack

| Layer | Choice |
| --- | --- |
| Frontend | React 18, Vite, React Router |
| Backend | FastAPI, Uvicorn |
| Agents | LLM-powered modules behind a single orchestrator (LangChain optional, retrieval + structured output only) |
| Scraping | `requests` + BeautifulSoup (planned) |
| Persistence | JSON + markdown files on disk |
| Retrieval | Local vector index (FAISS or Chroma) + embeddings (Gemini API or SentenceTransformers) |

The UI uses a **Soft Slate** theme: white background, slate-blue primary (`#334155`), cool-gray secondary text, muted sage for success.


## Setup and run

### Backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

API: http://localhost:8000 — Docs: http://localhost:8000/docs

### Frontend

```bash
cd frontend
npm install
npm run dev
```
