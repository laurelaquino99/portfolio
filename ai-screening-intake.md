# AI Screening Intake — concept prototype

This is an **experimental concept**, not a hiring product and not a system meant to make (or assist with making) hiring decisions.

It exists to **validate an idea**: could AI sit in a first-round application flow in a way that stays useful *without* taking autonomy away from the applicant or the recruiter? The code is a working sketch of how that might feel end-to-end — a demonstration of a stance, not a recommendation to automate hiring.

AI in hiring is controversial for good reasons: opaque scoring, candidates who cannot correct the record, and recruiters who are asked to defer to a model. This prototype was built to explore the opposite shape.

**The model does not decide, rank, reject, or score anyone.** It gathers and organizes. The applicant controls what is submitted. The recruiter interprets it.

---

## The idea being tested

Early-stage hiring is a translation problem. Candidates compress years of work into a résumé. Recruiters scan hundreds of documents that do not share a structure. Many AI interview tools skip that problem and jump to evaluation.

This concept asks a narrower question: what if AI were only an **intake and structuring layer**?

- extract what is already on the résumé (facts, not a judgment)
- ask, out loud, for what a résumé usually under-specifies
- merge both into one object the **applicant can review and edit**
- hand a consistent packet to a **person**, who still does the reading

If that loop does not preserve agency on both sides, the concept fails — which is why the prototype is built around disclosure, edit-before-submit, and a recruiter inbox with no scores.

---

## Agency, by design

**Applicants** are told they are talking to AI, what it is for, and what it is not for. They see the structured profile assembled from their résumé and conversation, and they can change it before anything is sent. Nothing is submitted on their behalf.

**Recruiters** are not given a ranking, a fit score, or an auto-reject. They get comparable sections, plus the original résumé and the conversation recording. Interpretation stays with them.

The assistant is an honest voice + abstract orb — not a fake human recruiter, and not a silent judge.

---

## What the prototype walks through

### Candidate

1. **Job posting** — apply from a careers-style page.
2. **Résumé upload** — PDF / DOCX parsed into a structured draft (background, ownership, impact, role-specific experience, motivation). Extraction is factual, not evaluative.
3. **Transparency** — before any voice session, the candidate is told the assistant only gathers information, does not score them, and that a human will review the application.
4. **Realtime voice conversation** — a short spoken session. A canvas orb pulses with the AI’s speech so the interaction feels live without impersonating a person.
5. **Review & edit** — résumé facts and spoken answers are consolidated. The candidate can correct or add anything, then choose to submit.

### Recruiter

An inbox of applications with shared structure:

- comparable profile sections
- original résumé
- conversation recording
- star / triage

Same packet, still a human reading it.

---

## Architecture

Two services, one pipeline.

```
Browser (React + Vite)
   │  REST  /api/process-resume
   │  REST  /api/consolidate-profile
   │  WS    /ws/chat  ── mic PCM16 ──▶  FastAPI  ──▶  OpenAI Realtime
   │                  ◀── audio + events ──
   ▼
Structured profile  →  recruiter inbox
```

| Layer | What it does |
| --- | --- |
| **React 18, Vite, React Router** | Candidate and recruiter UIs |
| **FastAPI + Uvicorn** | Uploads, LLM orchestration, WebSocket bridge to the voice model |
| **Gemini** | Résumé structuring and merging spoken answers into the same schema |
| **OpenAI Realtime** | Bidirectional voice: turn-taking, short screening script, streaming speech |
| **pypdf / python-docx** | Text from PDF and DOCX |
| **Web Audio API + Canvas** | Mic capture, PCM playback, analyser-driven orb |

The interesting engineering is orchestration: streaming audio both ways, keeping the assistant on a short intake script, then folding unstructured speech into a structured, **editable** object — not a verdict.

---

## Setup

Requires **Python 3.12+**, **Node 18+**, and API keys for OpenAI and Gemini.

### Backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Create `backend/.env`:

```
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=...
```

```bash
uvicorn main:app --reload --port 8000
```

Health check: `http://localhost:8000/api/health`

### Frontend

```bash
cd frontend
npm install
npm run dev
```

App: `http://localhost:5173`  
Vite proxies `/api` and `/ws` to `http://127.0.0.1:8000`.  
Recruiter view: `http://localhost:5173/recruiter`

---

## Walking the concept locally

1. Open the job page, switch to **Application**, upload a résumé, proceed.
2. Read the disclosure, then start the voice session (allow microphone).
3. After the chat, review the merged profile, edit if needed, submit.
4. Open `/recruiter` to see the same packet on the hiring side.

This repo is for exploring the concept. It is not software for running a real hiring process.