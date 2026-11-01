# AI Interview Study Platform

A **research project** on how people experience AI in high-stakes, human-facing interactions — specifically first-round interviews, where a conversational system is standing in for a person.

We are experimenting with how to **improve acceptance of AI in that role**: what to disclose, how the system should look and speak, and what makes a conversation feel comfortable, trustworthy, fair, and like you were actually understood. The work is design research, measured in live voice sessions rather than assumed from product intuition.

---

## What we researched

Across a single session, participants completed several **within-subjects** comparisons: they experienced more than one version of a short realtime voice interview, then rated the experience (Likert after some parts; comparative scales after paired versions).

Taken together, the studies asked:

- **Disclosure** — Does it change the experience if people are told they are speaking with AI (and that a human will review the session) versus framed as talking to “a member of the team”?
- **Visible capture** — Does showing people a live, lightweight record of what the system took from their answers change whether they feel heard, comfortable, or fairly treated?
- **Presence** — How does an abstract voice interface compare with a more human-like visual interviewer for trust, naturalness, and engagement?
- **How the AI talks** — How does a generic, fixed script compare with questions that reference the participant’s background and what they just said?

The conversation medium stayed the same (live voice, short turns). What changed was framing, what was shown on screen, how the interviewer was represented, and how questions were worded — so the research isolates the *design* of the interaction.

People rated comfort, trust, fairness, understanding, naturalness, engagement, and (where relevant) relevance and overall preference. Free-text comments sat alongside the scales.

---

## How a session works

1. **Config** — participant identifier and the condition knobs for that run.
2. **A series of short live interviews** — each followed by feedback (per-part ratings or a comparison after two variants).
3. **Completion**

Interviews are voice, not forms. Each condition has its own backend path and instruction set so treatments do not leak into each other. Server-side voice activity detection handles turn-taking. The assistant is held to a tight script or a tightly scoped personalization policy so versions stay comparable.

---

## Architecture

```
React study UI
   │  REST   POST /api/feedback
   │  REST   POST /api/analyze-transcript
   │  WS     /ws/openai/...                 (one bridge per condition)
   ▼
FastAPI
   ├─ OpenAI Realtime  (bidirectional PCM16, instructions per condition)
   ├─ Lightweight LLM  (short phrases for the on-screen notes condition)
   └─ JSON session logs + result dashboards
```

| Piece | Role |
| --- | --- |
| **React 18, Vite, React Router** | Study machine: config, intros, variants, scales, progress |
| **FastAPI** | Feedback store, transcript analysis, WebSocket bridges, dashboards |
| **OpenAI Realtime** | Voice-to-voice interviews; separate handlers so conditions stay isolated |
| **Web Audio + Canvas** | Mic capture, playback, volume-reactive orb |
| **On-screen notes** | Typewriter rendering of extracted phrases (one of the visual conditions) |
| **Visual interviewer** | Video/avatar condition for the representation comparison |
| **Analysis** | Per-participant JSON logs; scripts and Chart.js dashboards |

Condition isolation is deliberate. Different WebSocket routes and instruction sets are what make the comparisons a study rather than two slightly different chat UIs.

---

## Setup

**Python 3.12+**, **Node 18+**, OpenAI API key with Realtime access. Microphone permission in the browser.

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
```

Run (port **8011**):

```bash
python main.py
```

Or:

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8011
```

Health: `http://localhost:8011/api/health`

### Frontend

```bash
cd frontend
npm install
npm run dev
```

App: `http://localhost:5173`

Optional `frontend/.env`:

```
VITE_API_URL=http://localhost:8011
```

(Default is already that URL.) Run backend and frontend together; voice sessions need the Realtime bridge.

---

## Data

Feedback is written under `backend/data/` as `Name__YYYY-MM-DD.json` (one file per participant per day).

In-app dashboards (backend running):

- `http://localhost:8011/study1-visualizations`
- `http://localhost:8011/study2-visualizations`
- `http://localhost:8011/study3-visualizations`

Offline scripts in `backend/` aggregate the same logs.

Treat session files as **research data**. They can contain names and free-text comments; do not commit or publish them unless you have consent and a redaction plan.