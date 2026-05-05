# AI Wiki Vault

Personal Karpathy-method second brain. Single-file HTML. Open in browser, use.

## Run

```
start 2026-04-30-ai-wiki-vault-v4.html
```

That's it. No npm, no build, no server.

## What it does

- **Ingest** — paste a URL, PDF, or text. AI creates a source/clip page + concept stubs + person stubs, auto-links existing pages. (Tweet URLs become date-stamped `clip` pages.)
- **Query** — ask vault questions. AI answers with `[[wikilink]]` citations from your pages.
- **Lint** — periodic scan for broken links, orphans, missing summaries.
- **Graph** — D3 force-directed murmuration view. Hub nodes glow lime. Drag, zoom, click.
- **Sync** — optional GitHub backing (single `vault.json` blob, multi-device).

## Configure

Click ⚙ Settings:

| Section | What |
|---------|------|
| LLM Provider | Mock (offline) · OpenAI · OpenRouter (Hermes etc) · Anthropic · Ollama (local) · Custom |
| GitHub Sync | Owner / repo / branch / path / PAT |

Mock works offline with regex extraction (cheap, dumb). Real provider gives Karpathy-quality concept/person extraction.

## Files

- `2026-04-30-ai-wiki-vault-v4.html` — the app
- `CHANGELOG.md` — decision record + architecture notes
- `CLAUDE.md` — guide for Claude Code agents
- `AI_Wiki_Vault_Product_Spec.md` — pre-v4 spec, partially superseded
- `AI_Wiki_Vault_Council_Review_2026-05-01.md` — external review
- earlier `*.html` — deprecated versions

## Status

Phase 3 done (graph polish + GitHub sync + multi-provider LLM). Next iteration focuses on trust primitives: export/import, reversible AI batches, visible save-failure surfacing.
