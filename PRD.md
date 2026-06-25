# PRD — Clara Receipts

*A free receipt-to-Excel tool, designed as a structured experiment to open a bottom-up acquisition channel for Clara.*

> **Status:** validation prototype (live link below). Discovery is built on well-reasoned assumptions, **not field work** — this is a designed case, seeded by one real anecdote (n=1). Numeric targets are reasoned benchmarks, declared as such.
> **Live prototype:** https://juanfrancozafra-tech.github.io/receipts-case/
> **Disclaimer:** This is just a personal concept I developed around a real company. It is **not affiliated with, endorsed by, or owned by Clara**.

---

## 1. What this product does

Clara Receipts turns a pile of trip receipts — photos and PDFs — into a clean, Clara-branded expense report (Excel) in seconds. The user creates a trip, drops in their receipts, reviews an auto-filled table, and gets the report delivered to their email. It is **free** and requires no Clara account.

In this prototype the AI extraction is **simulated** (realistic mock data). That is intentional — see §11. The point of the build is to test whether the *experience* drives adoption and pipeline, which does not require a production OCR engine.

## 2. Who it's for

The person who **suffers the receipt consolidation today** — not the buyer.

- Field sales reps who travel and pay with a corporate card.
- Admin assistants and junior analysts who collect and re-type colleagues' receipts.

They have **high pain frequency, no purchasing power, and high upward influence** (finance sees their work every month). This is deliberate: they are cheap to reach, they feel the pain weekly, and they sit one email away from the real buyer. The CFO is explicitly **not** the target user of the tool.

## 3. The problem it solves

Today this person photographs each receipt and manually types every line into Excel — merchant, date, amount, tax — then emails it to finance. The cost shows up on four axes:

- **Effort.** It's manual, low-value data entry: transcribing field by field from a photo into a spreadsheet.
- **Time.** A single trip's receipts can take an hour or more to capture and reconcile — time taken from the person's actual job.
- **Errors and rework.** Hand-typed amounts and taxes are easy to get wrong; mistakes bounce back from finance and have to be fixed, adding a second round of work.
- **Repetition.** It isn't a one-off — the same chore recurs on every trip, so the pain compounds.

**Seed insight (real, n=1):** Juan's wife uses a corporate card on business trips, then photographs every receipt and types each one into Excel for finance. That single observed case is the anecdote the whole experiment is built around — and it is labeled as such, not presented as validated demand.

## 4. The hypothesis being tested

One central, quantified, falsifiable hypothesis:

> **A free, Clara-branded receipt-to-Excel tool will drive ≥ 40% activation and convert ≥ 10% of activated users into a Clara lead event, opening a bottom-up acquisition channel at a lower CAC than paid channels.**

It decomposes into **two layers**, because a PLG product can fail for two independent reasons:

- **Value hypothesis.** If we let someone upload receipts and get a clean Excel in seconds, they adopt it recurrently because it saves hours. → proven by the **activation** metric.
- **Growth hypothesis.** If that tool is free, useful, and Clara-branded, those users become promoters who put Clara on the table when their company evaluates spend management — generating pipeline at lower CAC. → leading signal is the **referral/lead-event rate**; ultimately proven by lagging business metrics.

The prototype is built to attack **both** layers, not just the value one.

## 5. Key metrics

Four metrics only — one leading signal per hypothesis layer, plus the two lagging business outcomes.

**Leading (early, predictive):**

- **Activation rate** — % of users who complete and share their first report (upload → Excel). The "aha". Proves the value hypothesis. **Target ≥ 40%.**
- **Referral/lead-event rate** — % of activated users who trigger "share with finance" / "request demo". Leading indicator of growth. **Target ≥ 10%.**

**Lagging (downstream, only provable in a real rollout):**

- **Qualified leads sourced by the tool** — companies entering Clara's funnel via it.
- **CAC vs. other channels** — the decisive business comparison.

### Kill switch / decision rule

- **Kill:** if the referral/lead-event rate stays **below 5% once ≥ 500 activated users** have used the tool (≈ first 8 weeks). No conversion → no low-CAC channel → no business reason for Clara to run it.
- **Gate, not kill:** if activation is **< 25% after ~200 users** (≈ 4 weeks), pause and fix the value flow first — value risk is fixable; only conversion failure kills.
- **Long-term strategic kill:** if CAC stays ≥ paid channels even with leads flowing.

Thresholds (5% / 500 / 25% / 8 wks) are reasoned, adjustable benchmarks.

## 6. Assumptions, ordered by risk

Build priority follows risk: the prototype concentrates on #1 and #2.

| # | Risk | Assumption | How it's tested | In prototype? |
|---|------|-----------|-----------------|---------------|
| 1 | 🔴 **Kill switch** | Free-tool users convert into Clara pipeline (the **conversion bridge**). | Referral / lead-event rate. | ✅ Screen 5 |
| 2 | 🟠 High | AI extraction saves real time (if users must fix every field, no time is saved). | Extraction accuracy + time-to-value. | ✅ Screen 3 (simulated) |
| 3 | 🟡 Medium | Pain is acute enough to adopt a new tool vs. staying in their Excel. | Activation rate + recurrence. | ⚠️ Partially; seeded by n=1 |
| 4 | 🟡 Medium | Users are reachable cheaply (search intent / word-of-mouth inside the company). | Channel experiments in a real rollout. | ❌ Noted here, not built |
| 5 | 🟠 High | **The all-in cost stays below Clara's current CAC** — acquisition cost *plus* the cost of running the product, including AI extraction per user. A free tool only makes sense if the fully-loaded cost per sourced lead beats paid channels. | Fully-loaded cost per qualified lead vs. current CAC (the lagging metric). | ❌ Real rollout; AI cost modeled in the AI strategy doc |
| 6 | 🟡 Medium | **Legal/compliance is manageable, not a blocker.** Processing receipt data (PII + corporate financial documents) can be done lawfully with standard data-protection measures. | Legal review + a compliant data-handling setup before launch (see §12). | ❌ Not in prototype |
| 7 | 🟢 Low | No cannibalization of Clara's core (target = non-customers). | Holds by design; no test needed. | — |

Assumption #1 is load-bearing, unproven, and novel — so it is the kill switch (§5).

## 7. User flow (first screen → last)

`Landing` → `Create trip` → `Upload receipts` → `Review & edit` → `Send report` → `Conversion bridge`

Design principle: **deliver the value first, ask for conversion only at the peak-value moment** — never before.

## 8. Screens and their purpose

| # | Screen | Purpose | Assumption it tests | Activation event |
|---|--------|---------|---------------------|------------------|
| 0 | **Landing / hook** | Value prop + single CTA "Create your first report — free". Light "by Clara" branding. | Frames the acquisition entry. | `visit` |
| 1 | **Create trip ("correría")** | Minimal form: trip name, destination, start/end dates. | Low-friction start. | `trip_created` |
| 2 | **Upload receipts** | Drag & drop photos/PDF → triggers simulated AI extraction. | #2 (setup) | `receipts_uploaded` |
| 3 | **Review & edit** ← the "aha" | Auto-filled, editable table (receipt thumbnail, merchant, date, category, subtotal, Tax (IVA), total, currency); total is computed and not editable; 1–2 fields flagged "review" so the edit affordance is visible. Sortable headers, summary cards (Total spent, Receipts). | **#2** — does AI save real time? | `extraction_completed`, `field_edited` |
| 4 | **Send your report** | **No download button.** The only way to get the report is to enter the email we send it to (validated, likely corporate → identifies the company). Optional "also send to finance" = organic branded reach to the buyer. | Value delivery. **Activation counted here.** | `email_captured`, `report_generated`, `report_sent` |
| 5 | **Conversion bridge** ← the kill switch | "With Clara cards, this report builds itself." Two CTAs: "Share Clara with my finance team" (referral) and "See how Clara works" (demo). | **#1** — does the free tool create pipeline? | `convert_cta_clicked`, `finance_referral_submitted` |

**Why no download button** (key design call): forcing email capture as the *delivery mechanism* (not a toll) means users give a real, usually corporate address. That identifies the company and turns the friction into function.

## 9. Scope (in / out)

**In scope — built in this prototype:**

- The full **6-screen flow** (§7–§8): landing → create trip → upload → review & edit → send → conversion bridge.
- **Simulated AI extraction** with an editable, sortable review table (the "aha").
- **Email-gated delivery** (no download button) with optional "send to finance".
- **Conversion bridge** with referral capture and demo CTA — the growth test.
- **Instrumented event taxonomy** firing across the whole funnel (§10).

**Out of scope — deliberately deferred (not cut from the vision):**

- **Production backend:** real OCR/vision extraction, generated `.xlsx`, email delivery, storage, and an analytics backend (scoped in §13 and the AI strategy doc).
- **Excel column customization.**
- **"My trips" dashboard / recurrence loop** — belongs to the recurrence story once activation is proven.
- **Channel acquisition experiments** (assumption #4) — only testable in a real rollout.
- **The production AI engine** — autonomy, model approach, guardrails, evals, and cost live in the AI strategy doc.

## 10. Analytics design

The experiment has a defined event taxonomy so the funnel is measurable, and the prototype already **fires every event** (via a `track()` helper) so the instrumentation is real — it currently logs to the browser console rather than a real analytics backend:

`visit` → `trip_created` → `receipts_uploaded` → `extraction_completed` → `field_edited` → `email_captured` → `report_generated` → `report_sent` → `convert_cta_clicked` → `finance_referral_submitted` (plus `new_report_started` on restart).

Activation is measured at `report_sent`; the referral/lead-event rate at `finance_referral_submitted` + `convert_cta_clicked`.

## 11. What's mocked vs. what would need real data

*(This is the product-validity view. The technical data model — real vs. mocked fields — lives in the Engineering handoff.)*

**Mocked intentionally, for a validation prototype:**

- **AI extraction.** The review table is pre-filled from a realistic mock receipt pool instead of running OCR/vision. This is a deliberate **Wizard-of-Oz design**: to test whether the *value and growth hypotheses* hold, the user only needs to experience "I dropped receipts and got a clean editable table." Real OCR accuracy is an execution risk to de-risk *after* the hypotheses are validated, not a precondition for testing them.
- **Report delivery.** "Sent to your email" is simulated; no email actually goes out.
- **Analytics events.** The full taxonomy is *instrumented* — every step fires a `track()` event — but it currently logs to the browser console rather than being sent to a real analytics backend.
- **Persistence.** Nothing is stored; state lives in memory for the session.

**What would need to be real before launch:** OCR/vision extraction, the generated Excel file, email delivery, an analytics pipeline, and storage. These are scoped in §13 and in the AI strategy doc.

## 12. Legal & data handling

Receipts and invoices carry personal and corporate financial data, so compliance is a real precondition for a launch — not for this prototype, which stores nothing. *This is a product-level view by a PM, not legal advice; it must be validated with counsel before any rollout.*

- **Personal data / habeas data.** Receipts contain PII (names, locations, travel patterns, sometimes card fragments). A real version needs lawful basis and a clear privacy policy — in Colombia under Ley 1581 de 2012 and Decreto 1377/2013 (consent, purpose limitation, data-controller duties, holders' rights); equivalent regimes apply if the tool expands across LATAM (e.g. LGPD in Brazil).
- **Third-party and corporate data.** Two cases to handle explicitly: the optional "send to finance" processes a third party's email, and the user is uploading company expense documents they may not own. The flow must make the user's authorization to process that data clear.
- **Sub-processors & international transfer.** Sending a receipt image to an external OCR/LLM vendor is a data transfer to a sub-processor. This requires a data-processing agreement, a vendor that does **not train on the data**, and attention to where data is processed/stored.
- **Fiscal documents.** Invoices are tax documents (e.g. Colombia's DIAN e-invoicing). The tool only *reads* them to extract fields — it does not alter or replace any fiscal record; worth stating to avoid scope confusion.
- **Retention, deletion & security.** Define how long receipt images are kept, a deletion path (holders' right to erasure), and security/breach-handling appropriate to financial data.

Mitigation posture: minimize what's stored, get explicit consent at capture, use no-training vendor terms with a DPA, set a retention window, and run a legal review before launch. The AI-facing pieces (PII handling, sub-processors, no-training, graceful failure) are also covered as guardrails in the AI strategy doc.

## 13. Recommended next steps for engineering

A pragmatic path from validation prototype to a launchable experiment, in priority order.

1. **Real extraction (highest-risk, do first).** Replace the mock pool with OCR/vision (e.g. a receipt-OCR API such as Mindee/Veryfi, or an LLM vision model). See the AI strategy doc for model approach, guardrails, evals, and cost.
2. **Report generation.** Produce the actual Clara-branded `.xlsx`.
3. **Email delivery.** Send the report and the optional finance copy (e.g. Resend).
4. **Storage + persistence.** Trips, receipts, and extracted rows (e.g. Supabase: Postgres + Storage + Edge Functions).
5. **Analytics pipeline.** Wire the event taxonomy to a real product-analytics tool so the funnel and the kill-switch metrics are measurable.
6. **Channel experiments (assumption #4).** Test how to reach the target user cheaply — the one assumption a prototype can't address.

## 14. Constraints (time, budget, team)

These are deliberate guardrails, not just a plan. The whole point of a bottom-up wedge is that the bet stays **cheap, fast, and reversible** relative to the paid channel it's trying to beat — so the experiment is time-boxed and budget-boxed up front. *(Figures below are reasoned benchmarks, adjustable with Clara's real numbers.)*

**Time.**

- **Launch window: ship within ~3–4 weeks of green-light.** The build reuses managed APIs (OCR, email, DB) instead of standing up infrastructure, so the critical slice is small enough to ship in that window with one engineer + PM. If it can't ship in ~4 weeks, the scope has crept past the critical slice (§9) — cut, don't extend.
- **Read-out window: ~8 weeks to the keep/kill signal**, with a gate at ~4 weeks / 200 users (§5). If the tool can't reach ~500 activated users within 8 weeks, the read is inconclusive *and* the channel isn't scaling — that is itself a negative signal.
- **Total elapsed: ≤ one quarter** from green-light to decision. The experiment is not allowed to drift into an open-ended program.

**Budget.**

- **Build is low-cost by design:** no custom OCR/ML and no dedicated infra — managed APIs only. The build is measured in a few engineer-weeks, not a team-quarter.
- **Variable-cost ceiling:** cost per processed receipt (mostly AI extraction) must stay under a cap such that the fully-loaded cost per qualified lead still beats current CAC (assumption #5). If per-receipt cost breaches that cap, the free model is broken on economics *before* growth even matters — a reason to kill independent of the funnel.
- **No offsetting revenue:** the tool is free, so the experiment runs on a fixed budget envelope with nothing to recover mid-flight. That's intentional — the bet is sized to be killable with negligible sunk cost.
- **Acquisition-spend cap:** channel experiments (assumption #4) run under a capped budget; if target CAC isn't reachable within that cap, kill.

**Team & scope.**

- Built and run by a **small team (~1 PM + 1 engineer)**, leaning on Clara's existing brand, legal, and sales functions rather than standing up new ones.
- **Scope held to the critical slice (§9):** anything not needed to test assumptions #1–#2 is deferred — specifically to protect the time and budget box above, not because it lacks value.

**Reversibility.** Because it's free, API-based, stores little, and is time-boxed, killing it leaves almost no sunk cost or technical debt. The cost of being wrong is kept deliberately low — which is what makes running the experiment an easy yes.
