# DockMaster Service Intelligence

**An AI-assisted operations console for marine service technicians.**

DockMaster Service Intelligence is an interactive product prototype that shows how AI can sit inside a real service workflow—not as a chatbot on the side, but as a working layer across diagnosis, repair planning, parts, labor, and technician collaboration.

The goal is simple: get a technician from *reported symptom* to *confident next action* in one screen, with the record updating as new evidence comes in.

---

## The problem

Marine service work is high-context and high-cost. A technician arrives with a vague customer complaint (“it clicks but won’t start”), a boat with years of service history, incomplete notes, and a parts cage that may or may not have what they need.

Most shop software stores the job. It does not help the technician think.

That gap shows up as:

- Repeated diagnosis of issues the shop has already seen on the same model
- Repair plans that don’t update when field tests change the picture
- Parts ordered too late because inventory status lives in a different tool
- Knowledge trapped in one senior tech’s head instead of the work order

This prototype is designed around that job, not around a generic dashboard.

---

## What this product demonstrates

A full **Service Intelligence** workspace for a live job:

| Area | What the technician sees |
| --- | --- |
| **Job queue** | Color-coded service requests with status and priority filters |
| **Case record** | Boat, customer, reported issue, and past service history in one place |
| **AI diagnosis** | Suggested root cause, reasoning, and confidence—with confirm / reject |
| **Technician notes** | Field observations that can trigger a fresh diagnosis pass |
| **Repair plan** | Step-by-step work instructions that stay tied to the current diagnosis |
| **Parts & inventory** | Needed vs. on-hand stock, with inline order flow for shortages |
| **Labor** | Estimated hours and technician assignment |
| **Job assistant** | A collapsible chat panel that answers in the context of the open request |

The layout is built for a shop floor, not a slide deck: queue on the left, case in the center, assistant on the right. Chat can be minimized so the work order stays full-width.

---

## The demo story (one job, three AI moments)

The prototype is designed as a **live walkthrough**, not a static mock.

### 1. Notes change the diagnosis

A job opens with a high-confidence electrical diagnosis (voltage drop / battery path). The technician records what they actually tested. **Rerun Diagnosis** re-analyzes the case and rewrites:

- diagnosis and reasoning
- repair plan
- required parts and stock posture
- estimated labor

The UI shows the analysis in progress, then updates the full record—not just a text box.

### 2. The assistant uses shop history

The technician asks whether the shop has seen this issue before. The assistant answers against **this boat, this model, and similar past failures**, not generic engine advice.

### 3. New evidence can rewrite the work order

When the technician reports a new test result in chat (for example, relay behavior that points away from the current root cause), the assistant:

- interprets the finding in plain language
- flags that the official diagnosis is now stale
- asks whether to update the record
- on confirmation, runs the same structured refresh across diagnosis, plan, parts, and labor

That last step is the product thesis: **AI that can propose a change to the system of record, with the technician still in control.**

---

## Product principles

- **Technician-in-the-loop.** Confirm, reject, and update are first-class actions. The model never silently owns the job.
- **One case, one truth.** Diagnosis, plan, parts, and hours move together. A new finding is not a chat footnote.
- **Operational, not ornamental.** Inventory shortages, purchase-queue orders, and technician assignment live next to the AI output.
- **Readable under pressure.** Compact typography, status color coding, collapsible sections, and a chat panel that can get out of the way.
- **Demo-deterministic.** The walkthrough is scripted so a live presentation is reliable. The interaction design is the point; the prototype does not depend on a live model call.

---

## Interface at a glance

**Queue** — Filterable list of service requests with request number, vessel, schedule, priority, and status.

**Case** — Stacked, collapsible sections:

1. Request overview and history  
2. Technician notes  
3. Diagnosis (with confidence, confirm / reject, and rerun)  
4. Repair plan  
5. Parts and inventory  
6. Labor / scheduling  

**Assistant** — ChatGPT-style conversation for the selected job, including streamed responses and in-thread decisions when the record should be updated.

---

## Technical snapshot

Built as a modern single-page application:

- **React 19** + **TypeScript**
- **Vite**
- Componentized UI (queue, case detail, chat, inventory order flow)
- Domain model for boats, customers, technicians, inventory, and service requests
- Client-side state for a fully interactive, offline-friendly product demo

This is a **frontend product prototype**: realistic data, production-quality interaction, and a complete technician workflow—without requiring backend or model infrastructure to present.

---

## Why this project exists

DockMaster Service Intelligence was created for a Case Study. It turns a messy field workflow into a coherent AI-assisted console.