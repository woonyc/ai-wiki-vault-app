# AI Wiki Vault

Personal Karpathy-method second brain. Single-file HTML app. No build step, no server, no npm.

## Run

```bash
start 2026-05-13-ai-wiki-vault-v5.2.html
```

Open the HTML file in a browser. That is the app.

## Current workflow (v5.2)

### Primary path
1. **Capture** — paste URL(s), PDF, or text into the **Shared Inbox**
2. **↻ Inbox** — manually refresh shared intake from GitHub when you want the latest cross-device state
3. **Claim + Prepare** — lock an inbox item and normalize it into reviewable drafts
4. **Review** — inspect title, type, summary, warnings, and suggested action
5. **Commit local** — write curated vault changes with a pre-commit backup snapshot
6. **↻ Sync** — manually push the curated `vault.json` state to GitHub

### Key separation
- **Shared Inbox** = cross-device intake coordination layer
- **Vault** = curated knowledge layer

This is the main v5.2 shift: capture no longer depends on a full-vault sync before another device or Hermes can see it.

## What v5.2 changes

v5.1 made capture safer, but it was still local-first. v5.2 adds a **Shared Sandbox / Layer 0** so intake is visible across devices before canonical vault sync.

### Shared inbox model
Captures live as per-item JSON files in the GitHub repo:

```text
inbox/
  captured/
  claimed/
  processed/
  discarded/
```

- **captured** — newly submitted intake items
- **claimed** — locked by a browser session or Hermes processor
- **processed** — successfully committed/audited
- **discarded** — intentionally rejected, retained for audit

State transitions use atomic moves (`PUT` new path + `DELETE` old path).

## What it does

- **Shared capture workflow** — cross-device inbox backed by GitHub `inbox/`
- **Manual low-cost refresh** — `↻ Inbox`, no auto-poll
- **Review-gated ingest** — claim, prepare, review, commit
- **Backups** — pre-commit snapshots with restore support
- **Finance quarantine** — finance-like content is flagged `review_required`
- **Query** — ask vault questions; AI answers with `[[wikilink]]` citations
- **Lint** — scan for broken links, orphans, and missing summaries
- **Graph** — D3 force-directed graph with Obsidian-style control panel
- **Canonical vault sync** — optional GitHub backing via `vault.json`

## Data model

The vault is organized as layered page types:

- **Wiki layer** — `concept`, `person`, `synthesis`, `note`
- **Raw layer** — `source`, `clip`
- **Atoms layer** — `claim`, `fact`, `decision`, `hypothesis`, `question`, `playbook`, `entity`
- **Contexts layer** — `context-pack`
- **Meta** — `anchor`

v5 added the type system and review-state model. v5.1 added the staged ingest workflow. v5.2 adds the shared cross-device inbox seam in front of that workflow.

## Configure

Click **⚙ Settings**.

- **LLM Provider** — Mock, OpenAI, OpenRouter, Anthropic, Ollama, Custom
- **GitHub Sync** — owner, repo, branch, path, PAT
- **GitHub Inbox folder** — defaults to `inbox`
- **Graph config** — groups, filters, colors, display, force tuning

Notes:
- **Mock** works offline and is useful for flow testing, but extraction quality is intentionally limited.
- Real providers give better concept/entity extraction.
- **↻ Inbox** is manual by design to save API cost.
- **↻ Sync** is for curated vault state, not raw intake coordination.

## Files

- `2026-05-13-ai-wiki-vault-v5.2.html` — current app build
- `index.html` — deployed entry file / site root; should mirror the promoted live version
- `CHANGELOG.md` — authoritative decision record
- `CLAUDE.md` — maintainer guide for Claude Code
- `HERMES_PROTOCOL.md` — contract for the always-on Hermes processor
- `docs/superpowers/specs/2026-05-12-shared-sandbox-design.md` — v5.2 shared sandbox design spec
- `AI_First_Second_Brain_Hybrid_Claude_Brief_2026-05-07.md` — architecture brief behind v5
- `AI_Wiki_Vault_Workflow_Rework_Claude_Instructions_2026-05-07.md` — workflow rework brief behind v5.1
- earlier `*.html` — deprecated historical versions

## Status

**Latest repo version:** v5.2 — shared sandbox / cross-device inbox implemented in `2026-05-13-ai-wiki-vault-v5.2.html`.

**Deployed site entry:** `index.html` should serve the promoted v5.2 build when the repo is up to date on GitHub Pages.

Implemented now:
- Shared Inbox / Layer 0
- manual `↻ Inbox` refresh
- claim / process / discard flow
- local commit with backup snapshots
- manual vault sync gate
- finance review-required handling
- atoms/context page types
- graph control panel
- Hermes processor contract

Still deferred:
- LLM-driven atom extraction into `claim` / `fact` / `decision` pages
- context-pack generation from promoted atoms
- typed relationships like `derived_from`, `supports`, `contradicts`
- full review-state lifecycle UI (`promote`, `deprecate`, `mark reviewed`)
- file-native markdown export beyond the single `vault.json` sync model

## Rule of thumb

If you want the truth about product state, read in this order:
1. `CHANGELOG.md`
2. `docs/superpowers/specs/2026-05-12-shared-sandbox-design.md`
3. `2026-05-13-ai-wiki-vault-v5.2.html`
4. `README.md`
