# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A documentation/reference repo for researching OpenAI's **Decisions API** (model
`gpt-6-luna`). There is no application code, build system, or test suite here —
do not look for one. The repo holds downloaded OpenAI docs plus a reusable Claude
Code skill that distills them.

## Layout

- `docs/` — raw docs pulled from `developers.openai.com`:
  - `decisions.md` — the full Decisions API guide (862 lines; all SDK examples).
  - `llms.txt`, `api-llms.txt`, `reference-llms.txt` — doc index files.
- `~/.claude/skills/openai-decisions/` — the reusable skill (see below).

## The reusable skill

The actual deliverable is a user-level skill installed outside this repo:

```
~/.claude/skills/openai-decisions/
├── SKILL.md                 # distilled usage guide + minimal examples
└── reference/guide.md       # copy of docs/decisions.md
```

Because it lives under `~/.claude/skills/`, it is available to every project, not
just this one. Invoke it with `/openai-decisions`, or it auto-loads when a task
mentions classification, routing, triage, rubric scoring, `gpt-6-luna`, or the
Decisions API. Keep `SKILL.md` and `reference/guide.md` in sync with `docs/decisions.md`
when the upstream guide changes.

## Key facts (for quick orientation)

- Model: `gpt-6-luna` (only model). Endpoint: `POST https://api.openai.com/v1/decisions`.
- Pricing: `$0.10 / 1M` input tokens, no output/cache charges. Public beta.
- Three question types: `predicate` (→ probability), `choice` (→ choice + distribution),
  `score` (→ probability-weighted average of level indices).
- Consumers must handle `type: "refusal"` in every answer.
- Images are inline base64 data URLs only (no hosted URLs or `file_id`); max 128 image
  parts; user messages only.

The canonical source of truth is `https://developers.openai.com/api/docs/guides/decisions`.
