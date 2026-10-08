# OpenAI Decisions API — Claude Code Skill

A reusable [Claude Code](https://claude.com/claude-code) skill for OpenAI's
**Decisions API** (model `gpt-6-luna`). The Decisions API returns typed decision
answers — probabilities, choices, and rubric scores — about 10x faster than the
Responses API, priced at input tokens only.

## What's inside

```
.claude/skills/openai-decisions/
├── SKILL.md                 # distilled usage guide + minimal Python/curl examples
└── reference/guide.md       # full upstream guide (all SDK examples)
docs/
├── decisions.md             # raw upstream guide (source of truth for guide.md)
├── llms.txt                 # doc index files
└── ...
```

## Install (as a user-level skill, available in every project)

```bash
mkdir -p ~/.claude/skills
cp -r .claude/skills/openai-decisions ~/.claude/skills/
```

Or symlink to stay in sync with this repo:

```bash
ln -sfn "$(pwd)/.claude/skills/openai-decisions" ~/.claude/skills/openai-decisions
```

## Usage

- **Auto-loads** when a task mentions classification, routing, triage, rubric
  scoring, `gpt-6-luna`, or the Decisions API.
- **Manual:** `/openai-decisions <your request>`.

## Key facts

- Model: `gpt-6-luna` (only model). Endpoint: `POST https://api.openai.com/v1/decisions`.
- Pricing: `$0.10 / 1M` input tokens, no output/cache charges. Public beta.
- Three question types: `predicate` (→ probability), `choice` (→ choice + distribution),
  `score` (→ probability-weighted average of level indices).
- Always handle `type: "refusal"`; images are inline base64 data URLs only.

Canonical source: https://developers.openai.com/api/docs/guides/decisions
