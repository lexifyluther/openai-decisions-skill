---
name: openai-decisions
description: Use the OpenAI Decisions API (model gpt-6-luna) for fast typed classification, routing, triage, guardrails, and rubric scoring. Trigger when the user needs to (a) get a probability that a condition is true, (b) pick one option from a fixed set, or (c) score an input against ordered levels — especially when they ask about the OpenAI Decisions API, gpt-6-luna, or want a faster/cheaper alternative to the Responses API for decision-type answers.
---

# OpenAI Decisions API

A dedicated endpoint for **typed decision answers** — probabilities, choices, and
scores — instead of prose. ~10x faster than the Responses API and priced at input
tokens only. Public beta.

## Quick facts

| Item | Value |
| --- | --- |
| Model | `gpt-6-luna` (only model available) |
| Endpoint | `POST https://api.openai.com/v1/decisions` |
| Price | `$0.10 / 1M` input tokens — **no** output or cache charges |
| SDK min | Python ≥3.26.0, JS ≥7.30.0, Go ≥3.73.0, Ruby ≥0.101.0, Java ≥4.78.0 |
| Compliance | ZDR + HIPAA eligible; data residency US & EU (EEA + CH) |
| SDK method | `client.decisions.create(...)` (Python/JS/Ruby), `client.Decisions.New` (Go), `client.decisions().create` (Java) |

## When to use it

- **Decisions API** → need one of these typed answers: `predicate`, `choice`, `score`.
- **Structured Outputs** (Responses API) → need a custom JSON object / extracted fields.
- **Function calling** → need the model to request a tool call with arguments.

## Request shape

Three top-level fields:

- `model` — always `"gpt-6-luna"`.
- `input` — shared evidence: a plain string, or a list of user messages with text/images.
- `questions` — array of questions, each with a unique `name`.

```json
{
  "model": "gpt-6-luna",
  "input": "I was charged twice for my order.",
  "questions": [ { "...": "..." } ]
}
```

## Question types

### `predicate` — is a condition true?

```json
{ "type": "predicate", "name": "visible_damage",
  "instructions": "Does the product have visible damage? Ignore packaging." }
```

→ returns `probability` (0–1).

### `choice` — pick one from an unordered set

```json
{ "type": "choice", "name": "department",
  "instructions": "Which department handles this?",
  "choices": [
    { "value": "billing",   "description": "Payments, invoices, refunds." },
    { "value": "other",     "description": "Requests outside these categories." }
  ] }
```

→ returns `choice`, `probabilities[]`, `confidence`. **Always add an `"other"` fallback.**

### `score` — rate against ordered levels (rubric)

```json
{ "type": "score", "name": "severity",
  "instructions": "How severe is this issue?",
  "levels": [
    { "label": "Cosmetic",            "description": "No lost functionality." },
    { "label": "Workaround available","description": "Task fails but another way works." },
    { "label": "Fully blocked",       "description": "Task fails, no workaround." }
  ] }
```

→ returns `score` = probability-weighted average of **level indices (start at 0)**,
plus `probabilities[]` and `confidence`. `score` can fall between levels (e.g. 1.1).

## Response shape & refusal

Response is `{ "answers": [ ... ] }` in question order; each answer echoes the
question `name`. **Every SDK/consumer must handle `type: "refusal"`.**

```python
answer = decision.answers[0]
if answer.type == "refusal":
    pass                       # couldn't answer; log or escalate
elif answer.type == "predicate":
    use(answer.probability)    # threshold
elif answer.type == "choice":
    use(answer.choice, answer.confidence, answer.probabilities)
elif answer.type == "score":
    use(answer.score, answer.confidence, answer.probabilities)
```

## Constraints (easy to trip on)

- Images must be **inline base64 data URLs**. Hosted `http(s)` URLs and `file_id` are **rejected**.
- Max **128 image parts** per request.
- Only **user messages** with `input_text` / `input_image` parts. No system/assistant, no tool calls, no audio, no files, no item refs.
- Dependent questions (answer A feeds question B) need **separate requests**; only independent questions go in one `questions` array.

## Thresholds

Set routing/filtering/review thresholds from **labeled examples**, balancing false
positives vs false negatives. Prefer human-review fallbacks for borderline cases
(low `confidence` or mid-range `score`).

## Minimal examples

**Python (choice):**
```python
from openai import OpenAI
client = OpenAI()
d = client.decisions.create(
    model="gpt-6-luna",
    input="I was charged twice for my order.",
    questions=[{
        "type": "choice", "name": "department",
        "instructions": "Which department should handle this?",
        "choices": [
            {"value": "billing", "description": "Payments, invoices, and refunds."},
            {"value": "other",   "description": "Anything else."},
        ],
    }],
)
a = d.answers[0]
if a.type != "refusal":
    print(a.choice, a.confidence)
```

**curl:**
```bash
curl https://api.openai.com/v1/decisions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-6-luna",
    "input": "I was charged twice for my order.",
    "questions": [{
      "type": "choice", "name": "department",
      "instructions": "Which department should handle this?",
      "choices": [
        {"value": "billing", "description": "Payments, invoices, and refunds."},
        {"value": "other",   "description": "Anything else."}
      ]
    }]
  }'
```

## Full reference

- Complete guide with every SDK example (Python, JS, Go, Java, Ruby, curl):
  `reference/guide.md`
- Live docs: https://developers.openai.com/api/docs/guides/decisions
- Playground: https://platform.openai.com/decisions
