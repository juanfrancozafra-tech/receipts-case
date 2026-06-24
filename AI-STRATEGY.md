# AI Strategy — Receipt Extraction

How the AI layer would work **in production**. In the current prototype, extraction is simulated (Wizard-of-Oz); this document is the forward-looking design for the real engine — what it does, how autonomous it is, how we'd build it, the guardrails around it, how we'd know it's good, and what it costs. It is design, not a claim that this is already running.

---

## 1. Problem & workflow

**Job to be done:** turn a photo or PDF of a receipt into a structured, correct expense line — merchant, date, category, subtotal, tax, total, currency — so the user gets a clean editable table instead of typing each field by hand.

**Where AI sits in the flow:** after upload (Screen 2), before review (Screen 3). The user drops N receipts; the model extracts each into a row; the user lands on a pre-filled, editable table and only corrects what's wrong. AI does the typing; the human approves. The "aha" is that most rows are already right.

**Why AI and not rules:** receipts are wildly heterogeneous — formats, languages, layouts, photo quality, handwriting. Template/regex approaches break on the long tail. Vision-capable models generalize across that variety, which is exactly the value the tool promises.

## 2. Autonomy level

**Human-in-the-loop by design — never auto-submit.** The model proposes; the user disposes. The review table (Screen 3) is a mandatory checkpoint: extracted values are editable, and low-confidence fields are visibly flagged for review.

This is a deliberate choice for a financial document: the cost of a silent wrong number (an expense report that doesn't reconcile) is high, and the user is right there anyway. We get the speed of automation without taking the human off the hook for correctness. Full autonomy (auto-file with no review) is explicitly **out of scope** until accuracy and trust are proven over time.

## 3. Data & model approach

**Start with a hosted model, not a trained one.** Two viable paths:

- **Receipt-OCR API** (e.g. Mindee, Veryfi) — purpose-built for receipts, returns structured fields out of the box. Fastest to a credible baseline.
- **LLM vision model** (e.g. a multimodal model) with a structured-output prompt — more flexible on categories, languages, and edge layouts; one vendor for extraction + categorization.

**Recommended:** prototype both on a small labeled set, pick on the accuracy/cost/latency trade-off; likely **LLM vision** for flexibility, with a receipt-OCR API as the fallback/comparison baseline.

**Data strategy:** we don't need a custom-trained model on day one. The valuable asset compounds over time — **every user edit is a labeled correction.** Logging (extracted value → user-corrected value) builds a high-quality evaluation and fine-tuning set for free, and tells us exactly where the model fails. No model training before launch; the correction log is the path to a tuned model later if accuracy demands it.

**Categorization:** map merchant + line items to an expense category. Start with the model's own classification constrained to a fixed category list; refine with a merchant→category lookup as data accumulates.

## 4. Guardrails

Because this touches money and corporate data, the AI runs inside hard constraints:

- **Computed fields, not generated.** The **Total is always `subtotal + tax`, computed in code — never produced by the model and never editable as a free field.** The model is not trusted to do arithmetic; the system does. (This is already true in the prototype.)
- **Confidence-flagged review.** Low-confidence fields are visibly flagged ("review") so the human looks where the model is unsure, rather than rubber-stamping everything.
- **No silent auto-submit.** The review checkpoint cannot be skipped (see §2).
- **Schema-constrained output.** The model must return a fixed schema (typed fields, currency from an allowed list, category from a fixed list). Anything off-schema is rejected and re-prompted, not shown to the user.
- **Graceful failure.** If extraction fails or confidence is very low for a receipt, surface an empty/partial row the user can fill in — never block the flow or fabricate plausible numbers. A wrong-but-confident number is worse than a blank.
- **PII & data handling.** Receipts contain names, card fragments, and locations. Minimize what's stored, don't send data to a model vendor that trains on it, and be explicit in the privacy copy about what happens to uploads. Define a retention window and deletion path before launch.
- **No hallucinated totals or merchants.** Prefer "unsure" (flagged for review) over a confident guess; the guardrail metric is the edit rate on flagged vs. unflagged fields.

## 5. Risks & mitigations

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Extraction accuracy too low on real-world photos (blur, glare, handwriting) | Edit rate spikes, no time saved, value hypothesis fails | Pre-launch evals on a realistic, messy image set; fallback OCR baseline; flag low-confidence rather than guess |
| Model arithmetic errors | Reports don't reconcile; trust lost | Totals computed in code, never by the model (guardrail §4) |
| Latency on multi-receipt batches | Slow time-to-value, drop-off before activation | Parallelize per-receipt calls; show per-file progress; cap batch size (15 in prototype) |
| Cost per extraction scales with free usage | Unit economics break for a *free* tool | Track cost per activated user; cheaper OCR baseline for simple receipts, vision model only when needed |
| PII / data-privacy exposure | Legal + trust risk | Data-handling guardrails (§4); no-training vendor terms; retention limits |
| Category misclassification | Annoying but low-stakes (user edits) | Fixed category list; merchant lookup; lowest-priority accuracy target |

## 6. Evals

### Definition of good

The engine is "good enough to launch" when, on a held-out set of realistic receipts: field-level accuracy on merchant/total/date is **≥ 90%**, the row-level clean rate (rows needing zero edits) is **≥ 70%**, and **zero** total-arithmetic errors occur (guaranteed by the computed-total guardrail). "Good" is defined by *what the user has to fix*, not by a model-internal score.

Two ongoing-health metrics track the engine in production — not launch gates, but drift signals:

- **Edit rate** — average fields edited per receipt; a rising edit rate is the early warning that extraction quality is degrading.
- **Time-to-value** — median time from upload to "report sent"; the product-level metric the AI most directly moves.

These ladder up to the PRD's activation metric: if extraction is bad, the edit rate climbs, time-to-value collapses, and activation never reaches the 40% target.

### Methodology

- **Golden set.** Assemble 100–300 real receipts spanning formats, languages, currencies, and photo quality (including deliberately bad images), each hand-labeled with ground-truth fields.
- **Automated scoring.** Run extraction over the set; compare field-by-field to ground truth; report per-field accuracy, row-clean rate, and a confusion view on categories.
- **Regression gate.** Re-run the golden set on every model or prompt change; block changes that lower accuracy on the high-value fields.
- **Production feedback loop.** Every user edit in the live tool is a real-world label; sample these into the golden set so evals track the actual input distribution over time.
- **Slice analysis.** Report accuracy by slice (currency, language, image quality) to find where it fails rather than hiding failures in an average.

## 7. Cost

For a **free** tool, AI cost is a direct hit to unit economics, so it's tracked from day one as **cost per activated user**, not per call.

- **Cost driver:** one extraction call per receipt; a trip averages a handful to ~15 receipts. So a single activation ≈ a handful of model calls plus a categorization step.
- **Order of magnitude:** receipt-OCR APIs price per page/document (cents per receipt); LLM vision prices per image + tokens (also cents-scale per receipt, varying with model). Either way, **single-digit cents per activated user** is the planning assumption to validate.
- **Levers:** route simple receipts to the cheaper OCR baseline and reserve the vision model for hard cases; cache by image hash to avoid re-extracting duplicates; cap batch size.
- **Why it's acceptable:** the tool is an acquisition channel. The relevant comparison isn't "cost per report" — it's **cost per Clara lead vs. paid-channel CAC** (the PRD's decisive metric). A few cents of extraction cost per user is rational only if the referral/lead-event rate clears the kill-switch threshold.
