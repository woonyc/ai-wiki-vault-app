# AI Wiki Vault

Personal Karpathy-method second brain. Single-file HTML app. No build, no server, no npm.

## Run

```bash
start 2026-05-12-ai-wiki-vault-v5.1.html
```

Open the HTML file in a browser. That is the app.

## Current workflow (v5.1)

Primary path:

1. **Capture** — paste URL(s), PDF, or text into **Temp Inbox**
2. **Prepare** — deterministic normalization and draft creation
3. **Review** — inspect title, type, summary, warnings, and suggested action
4. **Commit local** — write to vault with a pre-commit backup snapshot
5. **Sync remote** — optional manual GitHub sync via `vault.json`

This is a deliberate shift away from the old “capture immediately mutates vault and auto-pushes” model.

There is still a **`⊕ Direct`** escape hatch for the legacy fast path, but the default v5.1 workflow is staged and review-gated.

## What it does

- **Capture / Review workflow** — Temp Inbox, Prepare, Review Gate, local commit, manual sync
- **Ingest** — commit a source/clip page plus concept/person stubs and auto-links
- **Finance quarantine** — finance-like content is flagged `review_required` and defaults toward safer handling
- **Query** — ask vault questions; AI answers with `[[wikilink]]` citations
- **Lint** — scan for broken links, orphans, and missing summaries
- **Graph** — D3 force-directed graph with Obsidian-style control panel
- **Backups** — pre-commit snapshots with restore support
- **Sync** — optional GitHub backing via a single `vault.json` blob

## Data model

The vault is organized as layered page types:

- **Wiki layer** — `concept`, `person`, `synthesis`, `note`
- **Raw layer** — `source`, `clip`
- **Atoms layer** — `claim`, `fact`, `decision`, `hypothesis`, `question`, `playbook`, `entity`
- **Contexts layer** — `context-pack`
- **Meta** — `anchor`

v5 adds the type system and review-state model. v5.1 adds the staged ingest workflow around them.

## Configure

Click **⚙ Settings**.

- **LLM Provider** — Mock, OpenAI, OpenRouter, Anthropic, Ollama, Custom
- **GitHub Sync** — owner, repo, branch, path, PAT
- **Graph config** — groups, filters, colors, display, force tuning

Notes:
- **Mock** works offline and is useful for testing flow, but extraction quality is intentionally limited.
- Real providers give better concept/entity extraction.
- GitHub sync is manual by default in v5.1.

## Files

- `2026-05-12-ai-wiki-vault-v5.1.html` — current app
- `index.html` — deployed entry file / site root; currently still serving the older v4 build until you promote v5.1 into it
- `CHANGELOG.md` — authoritative decision record
- `CLAUDE.md` — maintainer guide for Claude Code
- `AI_First_Second_Brain_Hybrid_Claude_Brief_2026-05-07.md` — architecture brief behind v5
- `AI_Wiki_Vault_Workflow_Rework_Claude_Instructions_2026-05-07.md` — workflow rework brief behind v5.1
- earlier `*.html` — deprecated historical versions

## Status

**Latest repo version:** v5.1 — workflow rework is implemented in `2026-05-12-ai-wiki-vault-v5.1.html`.

**Currently deployed site entry:** `index.html` still serves the older v4 build until you replace or redirect it.

Implemented now:
- Temp Inbox / Prepare / Review Gate
- local commit with backup snapshots
- manual sync gate
- finance review-required handling
- atoms/context page types
- graph control panel

Not in v5.1 yet:
- LLM-driven atom extraction into `claim` / `fact` / `decision` pages
- context-pack generation from promoted atoms
- typed relationships like `derived_from`, `supports`, `contradicts`
- full review-state lifecycle UI (`promote`, `deprecate`, `mark reviewed`)
- file-native markdown export beyond the single `vault.json` sync model

## Rule of thumb

If you want the truth about product state, read:
1. `CHANGELOG.md`
2. `2026-05-12-ai-wiki-vault-v5.1.html`
3. `README.md`
