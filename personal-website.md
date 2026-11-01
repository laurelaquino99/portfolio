# Laurel Aquino — personal site

A custom personal site for [Laurel Aquino](https://www.linkedin.com/in/laurel-aquino) — education, career, interests, and skills in one place. It’s also a sandbox for interactive web design, and (coming soon) an on-site chatbot that answers from Laurel’s own profile rather than making things up.

Built as a production-style Next.js app with a custom layout, motion, and interaction model: part portfolio, part playground.

## What it is

A single-page experience that walks through who I am, how I got here, roles, education, projects, and how to get in touch. The interesting part is the craft:

- **Section-based navigation** with snap scrolling, so the site reads like a sequence of screens rather than a long document
- **A custom journey timeline** (layout and data owned in-app — no third-party timeline library)
- **Interactive experience cards** that flip on click or keyboard, with copy written for both desktop and mobile
- **Scroll-triggered reveals** that respect `prefers-reduced-motion`
- **A grounded chat API** scaffolded for an on-site assistant that answers from my profile instead of freewheeling

The visual system (type, color, spacing, motion) is custom CSS — not a component kit dropped on a default Next.js starter.

## Stack

| | |
| --- | --- |
| Framework | Next.js 15 (App Router) |
| UI | React 19, TypeScript |
| Styling | Custom CSS, Google fonts |
| APIs | Route handlers for a future in-site chatbot |

## Status

Actively in progress. Core structure, design, and content are in place; I am still iterating on motion, content, and wiring the chatbot to a model with a tight persona so it stays factual.

## Contact

- [LinkedIn](https://www.linkedin.com/in/laurel-aquino)