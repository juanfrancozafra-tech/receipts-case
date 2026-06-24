# Engineering Handoff — Clara Receipts prototype

For an engineer picking this up. It explains how the prototype is **actually built**, what's real vs. faked at the data level, and the three decisions that most shape the code. For *why* the product is designed this way, see the PRD; for the production AI design, see the AI strategy doc.

---

## Start here

The entire prototype is **one file**: `index.html` (~900 lines). No build step, no dependencies to install, no backend.

- **Stack:** React 18 + Babel Standalone (in-browser JSX transform) + Tailwind, all via CDN.
- **Run it:** open `index.html` in a browser, or `python3 -m http.server` and visit `localhost:8000`.
- **Mental model:** a single `App` component holds all state and renders one of six screens based on a `step` counter. There is no router and no persistence — refreshing resets everything. State lives in React `useState` hooks for the session only.
- **What's faked:** AI extraction, email delivery, and analytics are all simulated. No network calls leave the page. See the data model below.

## Component inventory

All components live in `index.html`. Top to bottom:

| Component | Role |
|-----------|------|
| `App` | Root. Owns all shared state (`step`, `trip`, `files`, `rows`, `email`, `toFinance`, `financeEmail`, `processing`) and routes between screens. |
| `Header` / `Footer` | Static "by Clara" chrome. |
| `Stepper` | Progress indicator across the 6 steps. |
| `Landing` | Screen 0 — value prop + single CTA. |
| `TripForm` | Screen 1 — trip name, destination, start/end dates (with validation). |
| `Upload` | Screen 2 — drag & drop zone, per-file progress bars, "Load more" (9 at a time, cap 15), triggers simulated extraction. |
| `Review` | Screen 3 — the editable extraction table: sortable headers, load-more rows, summary cards (Total spent, Receipts). |
| `EditableCell` | Reusable inline-edit cell used by `Review`; supports numeric, date-picker, and "flagged for review" variants. |
| `Send` | Screen 4 — email capture (required), optional "also send to finance"; no download button. |
| `Done` | Screen 5 — conversion bridge: "share with finance" referral form + "see how Clara works" demo CTA. |

**Key constants** (top of file): `RECEIPT_POOL` (the 15 mock receipts), `MOCK_FILES` (mock filenames), `MAX_RECEIPTS = 15`, `PAGE_SIZE = 9`, and `rowTotal = subtotal + tax`.

## Data model — real vs. mocked

There is no schema/DB; "data" is in-memory React state shaped like this:

```
trip  = { name, destination, start, end }        // no "responsible" field
file  = { name, progress }                       // upload simulation
row   = { id, merchant, date, category, subtotal,// one per uploaded receipt
          tax, currency, flagDate?, flagTax? }
        // total is NOT stored — it's computed: rowTotal(row) = subtotal + tax
```

Review table columns (in order): **Receipt** (thumbnail) · Merchant · Date · Category · Subtotal · **Tax (IVA)** · Total (computed) · **Cur.** (currency).

| Field / behavior | Status | Notes |
|------------------|--------|-------|
| `trip.*` | **Real** (user input) | Captured on Screen 1, used in the report header. |
| Uploaded files | **Mocked** | Drag/drop is real UI, but filenames/thumbnails are drawn from `MOCK_FILES`; file bytes are never read. |
| Extracted rows | **Mocked** | Cloned from `RECEIPT_POOL`, one row per uploaded receipt. No OCR runs. |
| Field edits | **Real** | Edits to merchant/date/category/subtotal/tax/currency mutate state live. |
| **Total** | **Computed, real** | Always `subtotal + tax`, in code, not editable — the model/user never sets it directly. |
| `Total spent` summary | **Computed, real** | Reactive sum of all row totals. |
| Email send | **Mocked** | Validates the address, shows a success state; no email is sent. |
| Analytics events | **Instrumented, logs to console** | A `track(event, payload)` helper fires the full PRD taxonomy at every step (`visit`, `trip_created`, … `finance_referral_submitted`). It currently `console.log`s; swap the body for a real analytics call to wire it up. |

When this becomes real, the same `row` shape maps cleanly onto a `receipts` table (Postgres) and the extraction output schema described in the AI strategy doc.

## The 3 biggest technical decisions

1. **Single self-contained HTML file, React via CDN, no build step.**
   *What:* everything in one `index.html` using Babel Standalone to transform JSX in the browser.
   *Why:* the case is submitted as a **link**, the builder is a PM (not a build-tooling specialist), and the priority was iteration speed + trivial deployment (drop one file on GitHub Pages). *Trade-off:* in-browser Babel is fine for a prototype but not for production — a real build would move to Vite/Next with a proper bundle. This is a deliberate prototype-grade choice, not a recommendation for the shipped product.

2. **Simulated extraction via a mock receipt pool (Wizard-of-Oz).**
   *What:* `RECEIPT_POOL` provides realistic rows; uploading clones one row per file and flags 1–2 fields for "review" to make the AI feel real.
   *Why:* the hypotheses being tested (value + growth) don't require working OCR — only the *experience* of "drop receipts → get a clean editable table." This removes the biggest time-risk (OCR quality, secrets, hosting) from the critical path. The real engine is fully specified in the AI strategy doc and scoped in the PRD roadmap. *Trade-off:* not demoable on a user's own receipts; acceptable for a validation prototype.

3. **Total is computed in code, never stored or hand-editable.**
   *What:* `rowTotal(r) = subtotal + tax`; the Total column and the "Total spent" card are derived, reactive values.
   *Why:* it's a financial document — arithmetic must be guaranteed by the system, not trusted to a user keystroke or (later) a model output. This is the prototype-level expression of the computed-fields guardrail in the AI strategy doc, and it's the pattern the production extraction should preserve: **the model proposes line items; the system owns the math.**
