# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Single-file HTML prototype for AI Wiki Vault — an Obsidian-inspired, AI-assisted research wiki built around the **Karpathy LLM-wiki method**. Flat vault, type-folders, AI writes direct, multi-provider LLM, GitHub sync. No build system, no backend, no npm. Everything runs by opening one HTML file in a browser.

**Read `CHANGELOG.md` first.** It is the authoritative decision record. `CLAUDE.md` (this file) is a quick orientation.

## How to run

```
start 2026-04-30-ai-wiki-vault-v4.html
```

Persistence is `localStorage`. Optional GitHub sync via single `vault.json` blob (configure in ⚙ Settings).

## File history

| File | Status |
|------|--------|
| `ai-wiki-vault-prototype.html` | v1 — Tailwind CDN, SVG graph. Deprecated. |
| `2026-04-26-ai-wiki-vault-v2.html` | v2 — iteration. Deprecated. |
| `2026-04-27-ai-wiki-vault-v3.html` | v3 — IBM Plex + D3 + rust/lime palette. Compartment silos, review queue, research missions. **Deprecated**, kept only as graph polish reference. |
| `2026-04-30-ai-wiki-vault-v4.html` | **Current.** Karpathy method. Flat vault. Multi-provider LLM. GitHub sync. |

Earlier specs (`AI_Wiki_Vault_Product_Spec.md`) describe pre-v4 model with compartments, review queue, claims, contradictions — **partially superseded**. Check `CHANGELOG.md` for what was killed and why.

## Architecture (v4)

All logic in one `<script>` tag. Modules are conventional, single-file but seamed:

| Module | Role |
|--------|------|
| `Vault` | Single seam over state. All reads/writes/persist route here. |
| `LocalStorageAdapter` | Primary storage. Always synchronous. |
| `GitHubSync` | Sync layer on top. Push debounced 1.8s. Pull on boot + manual ↻. SHA-tracked, 409 → pull+merge+retry. |
| `LLM` | Transport with two shapes: `_callOpenAI` (bearer + chat/completions) and `_callAnthropic` (x-api-key + messages, with cache_control). |
| `MockLLM` | Offline heuristic provider. Regex extraction, keyword query. |
| `PROVIDER_PRESETS` | Mock / OpenAI / OpenRouter / Anthropic / Ollama / Custom. |
| `SourceFetcher` | URL (Jina Reader), PDF (pdf.js), text adapters. |
| `IngestFlow` | State machine: idle → fetching → preview → committing → done/error. |
| `LinkResolver` | explicit/backlink/tag-peer derivation. |
| `GraphProjection` | pure vault → {nodes, edges}. |
| `Anchors` | Auto-renders `index.md` (catalog by type) + `log.md` (append-only ops history). |
| `PAGE_TYPES` | Enum: concept · person · source · clip · synthesis · note · anchor. Drives tree folders, glyphs, graph color. |
| `Views` | Registry: `{page, drafts, graph, search, tag}`. |
| `Settings` | Multi-provider + GitHub config persisted to localStorage. |

## Domain model

**Karpathy three operations:**
- **Ingest** — paste URL/PDF/text → fetch → preview → commit. Writes one source/clip page + N concept stubs + M person stubs + auto-links existing pages.
- **Query** — vault corpus in cached system prompt → answer with citations → optionally save as synthesis page.
- **Lint** — heuristic scan for broken wikilinks, orphans, missing summaries → drafts.

**Page types** drive everything (folders, graph color, glyphs):
- `concept` rust — primary ideas
- `person` violet — entities
- `source` teal — long-form content
- `clip` dim ink — short social posts (twitter/x/threads/reddit/linkedin)
- `synthesis` amber — derived insight
- `note` gray — generic
- `anchor` ink — meta (index.md, log.md)

URL pattern detects clip vs source. Clips get `YYYY-MM-DD handle` title prefix.

**Cascade delete** — deleting a page also deletes draft stubs created from it (body matches `(Created from|First mention via) [[deleted-title]]`). Other references degrade to plain text.

## Design system

CSS variables in `:root`. IBM Plex Sans + IBM Plex Mono. Rust `#d97757` accent + lime `#CCFF00` highlights. Slim dark scrollbar.

Layout: `240px tree | 1fr main | 220px rail` grid. 48px topbar + 24px statusbar.

Graph: D3 force sim with murmuration `flow` force (per-node sin/cos phase + swarm-avg drift). Hub nodes (top-3 by degree) get lime ring. Type-clustered with `forceX/Y` cohesion. Random neuron pulse every 1400ms.

When editing: use CSS variables, not hardcoded hex.

## What is mocked

Heuristic extraction in `MockLLM`. Real LLM calls require user-configured key in ⚙ Settings (OpenAI/OpenRouter/Anthropic/Ollama/Custom). Real URL fetch via Jina Reader (no auth). PDF parsing via pdf.js. GitHub sync requires user PAT.

## Decisions worth not re-litigating

- Single-file HTML. No build step.
- Flat vault with **type-folders** (descriptive). Compartments (prescriptive silos) rejected — see CHANGELOG.
- AI writes direct, no review queue ceremony. Drafts panel reserved for Lint output.
- localStorage primary, GitHub as sync layer (not replacement).
- Browser-direct API calls — keys in localStorage, personal-use only. No backend.
- Single `vault.json` blob on GitHub. File-native markdown export deferred until core loop trusted.

## Things to maintain when editing

- Update `CHANGELOG.md` for any architectural decision.
- New file format / schema bump → migration in `Vault.migrate()`.
- New page type → add to `PAGE_TYPES` enum (single source of truth).
- New provider → row in `PROVIDER_PRESETS`. `LLM._callOpenAI` covers OpenAI-compatible shapes.
- New view → entry in `Views` registry.
