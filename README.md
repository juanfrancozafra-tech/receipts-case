# Clara Receipts — a product case

> **Disclaimer:** This is an independent portfolio case study created by Juan Carlos Franco Zafra. It is not affiliated with, endorsed by, or sponsored by Clara. The Clara name and brand are used for illustrative purposes only, and all data shown is fictional.

A free, Clara-branded tool that turns a pile of trip receipts into a clean expense report (Excel) in seconds — designed as a **structured experiment** to test whether it can open a bottom-up acquisition channel for Clara.

**▶ Live prototype:** https://juanfrancozafra-tech.github.io/receipts-case/
**◆ Slide deck:** https://juanfrancozafra-tech.github.io/receipts-case/deck.html

---

## What this is

This repo is a product case, not a finished product. It's an end-to-end experiment design — **hypothesis → assumptions → working prototype → measurement plan** — for a free receipt-to-Excel tool aimed at people who consolidate trip receipts by hand inside companies that don't yet use Clara. The bet: a genuinely useful free tool turns those users into internal champions who put Clara on the table when their company evaluates spend management (bottom-up PLG).

The prototype is a working, clickable instrument — the AI extraction is intentionally simulated (a Wizard-of-Oz design; see the PRD). The goal of the case is to show product judgment and the modern build workflow, not to ship a backend.

## How it was built

Built by a senior product/business PM using an AI-first workflow — not by a career engineer. The prototype is a single self-contained HTML file (React + Babel + Tailwind via CDN, no build step), written and iterated with AI assistance, then published to GitHub Pages as a public link. Total effort: a focused multi-day sprint (~6–8 h) done together with an AI agent, with decisions logged as they were made.

This "how it was built" story is part of the point: the role prizes treating AI as a core building workflow and staying close to the work, and this case is the live proof of that.

## Repo map

| File | What it is |
|------|-----------|
| **`PRD.md`** | The core document: problem, target user, hypothesis, assumptions by risk, screens, metrics, **kill switch**, what's mocked vs. real, next steps. Start here for product judgment. |
| **`AI-STRATEGY.md`** | How the AI extraction would work in production: autonomy, data & model approach, **guardrails**, risks, evals, and cost. Forward-looking design for the currently-mocked engine. |
| **`ENGINEERING-HANDOFF.md`** | How the prototype is actually built: component inventory, data model (real vs. mocked), and the three biggest technical decisions. |
| **`index.html`** | The published prototype (single file). |
| **`deck.html`** | The slide deck (reveal.js, single file) — the case summarized in 4 slides. |

**Suggested reading order:** `PRD.md` → `AI-STRATEGY.md` → `ENGINEERING-HANDOFF.md`.

## How to run / view

It's a single static file — no install, no build.

- **Online:** open the live link above.
- **Locally:** open `index.html` in any modern browser, or serve the folder (`python3 -m http.server`) and visit `localhost:8000`.

## Project structure

```
.
├── index.html              # the prototype (React + Tailwind via CDN, single file)
├── deck.html               # the slide deck (reveal.js, single file)
├── PRD.md                  # product requirements + experiment design
├── AI-STRATEGY.md          # production AI design (extraction, guardrails, evals, cost)
├── ENGINEERING-HANDOFF.md  # technical handoff for an engineering team
└── README.md               # this file
```
