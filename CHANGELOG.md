# AI Wiki Vault — Changelog

Personal Karpathy-method second brain. Single-file HTML app. No backend (yet).

## Files

| File | Status |
|------|--------|
| `ai-wiki-vault-prototype.html` | v1 — purple Obsidian theme, SVG graph |
| `2026-04-26-ai-wiki-vault-v2.html` | v2 — iterated design |
| `2026-04-27-ai-wiki-vault-v3.html` | v3 — IBM Plex + D3 graph + rust/lime palette. Compartment silos, review queue, research missions. **Deprecated** — design reference for graph polish only. |
| `2026-04-30-ai-wiki-vault-v4.html` | **Current.** Karpathy method, flat vault, type folders, multi-provider LLM. |
| `AI_Wiki_Vault_Product_Spec.md` | Original spec (pre-Karpathy pivot). Many decisions superseded. |
| `CLAUDE.md` | Repo guide for Claude Code. |
| `CHANGELOG.md` | This file. |

---

## v4 (2026-04-30) — Karpathy pivot

### Vision lock

Stripped-down Obsidian + Karpathy LLM-wiki method. Personal second brain. Multi-device sync deferred to phase 3.

User profile: solo researcher. Knowledge domains (trading, design, AI agents, fitness) treated as **emergent tags**, not enforced silos. Agents (Hermes, OpenClaw) write to same vault as the user.

### Killed from v3

- **Compartments** (Trading/Design/Agents/Fitness silos). Replaced with flat vault + tags.
- **Review queue ceremony.** AI writes direct. Karpathy method = LLM-as-author, user reverts via git/edit.
- **Research missions tab.** Defer.
- **Cross-compartment proposals, contradictions store, claims store, AI rules per compartment.** Out of scope.
- **5-tab shell** (Inbox/Wiki/Graph/Review/Research) → file tree + main + right rail.

### Architecture (deepened modules, single-file but seamed)

| Module | Responsibility |
|--------|---------------|
| `Vault` | Single seam over state. Reads/writes/persist/search. All mutations route here. |
| `LocalStorageAdapter` / `GitHubAdapter` (stub) | Storage seam. GitHub adapter unwired — phase 3. |
| `LLM` (transport) + `MockLLM` | Two adapters. `LLM.call()` routes to OpenAI-compatible or Anthropic shape via `Settings.resolvedShape()`. |
| `SourceFetcher` | Adapters: `fromUrl` (Jina Reader), `fromPdf` (pdf.js), `fromText`. |
| `IngestFlow` | State machine: `idle → fetching → preview → committing → done | error`. View subscribes. |
| `LinkResolver` | Single seam for explicit/backlink/tag-peer link derivation. |
| `GraphProjection` | Pure `vault → {nodes, edges}`. Renderer takes projection. |
| `Anchors` | Auto-generated `index.md` (catalog by type) + `log.md` (append-only ops history). |
| `Views` | Registry: `{page, drafts, graph, search, tag}`. |
| `Settings` + `PROVIDER_PRESETS` | Multi-provider config: OpenAI, OpenRouter, Anthropic, Ollama, Custom. |
| `Page / Source / Draft / LogEntry` factories | Domain types. |
| `PAGE_TYPES` enum | Single source of truth: `concept | person | source | clip | synthesis | note | anchor`. Drives folders, glyphs, tree grouping, index.md. |

### Karpathy-method ingest

One ingest produces 5-15 pages, not 1.

- Source/clip page (always)
- **Concept stubs** for new concepts (status: draft)
- **Person stubs** for new entities/handles (status: draft)
- **Auto-linked existing pages** get `## Mentioned in` appended

URL pattern → clip detection: `twitter.com / x.com / threads.net / linkedin.com/posts / mastodon / bsky.app / reddit.com/.../comments`. Clip titles get `YYYY-MM-DD handle` prefix.

### Three operations (Karpathy)

- **⊕ Ingest** (Cmd-I) — paste URL/PDF/text → fetch → preview → commit. Multi-stage UI.
- **? Query** (Cmd-/) — vault corpus cached on system prompt → answer with citations → optionally save as synthesis page.
- **⌖ Lint** — heuristic scan (broken wikilinks, orphans, missing summaries) → drafts.

### Multi-provider LLM (phase 2)

| Provider | Endpoint | Notes |
|----------|----------|-------|
| Mock | none | offline heuristic, free, dumb |
| OpenAI | api.openai.com/v1 | bearer auth, OpenAI-compatible shape |
| OpenRouter | openrouter.ai/api/v1 | Hermes-3, Llama, GPT, Claude — one key |
| Anthropic | api.anthropic.com/v1 | uses `cache_control` for repeat-query savings |
| Ollama | localhost:11434/v1 | local, needs `OLLAMA_ORIGINS=*` |
| Custom | user-typed | any OpenAI-compatible (Together, Groq, Fireworks, vLLM, LM Studio) |

OpenRouter sets `HTTP-Referer` + `X-Title` headers automatically. Anthropic uses prompt caching on vault corpus. JSON parsing tolerates code-fence wrapping.

### MockLLM extraction tightening

After v4 first release, MockLLM regex was producing junk concepts ("Before/Here/March") and false-positive people ("Cookie Use/Privacy Policy"). Fixed:

- Concepts require **2-4 word Title Case phrases**, length ≥8 chars.
- 40+ sentence-start stopwords blocked.
- 12 UI-noise phrases blocked.
- Person regex (capitalized "First Last") killed entirely. MockLLM now only extracts `@handles` + URL handle. Real LLM does proper name detection via prompt.

Real LLM prompt rewritten with **explicit GOOD/BAD examples** matching user's Obsidian vault: GOOD = `Swing Trading Pullback Entries / LLM Wiki Pattern / Claude.md Behavioral Guidelines`. BAD = sentence-starts, UI labels, products-as-people.

### UI

- IBM Plex Sans + IBM Plex Mono.
- Rust `#d97757` accent + lime `#CCFF00` highlights.
- Slim dark scrollbar (rust on hover).
- Top bar: brand · search/Cmd-K · Ingest/Query/Lint/Sync/Settings.
- Tree (240px) — anchors, drafts, type-folders (collapsible), tags, graph link.
- Main pane — markdown render with `[[wikilinks]]` + click-to-create on broken links.
- Right rail (280px) — backlinks · neighborhood mini-graph (D3) · tags.
- Status bar — provider/model · stats.
- Settings modal — 6 provider tabs in 3×2 grid, label/input grid layout.
- Ingest modal — 4-stage bar (source → fetch → preview → done), mode tabs (URL/PDF/Paste), drop-zone for PDF.

### Data shape

```js
state = {
  schemaVersion: 1,
  pages: [{ id, title, type, tags, sources, status, md, updated }],
  sources: [{ id, title, url, raw, summary, type, ingested }],
  drafts: [{ id, kind, target, content, reasoning, created }],
  log: [{ ts, op, summary }],
  ui: { selectedPageId, activeView, activeTag, editing, collapsedFolders }
}
```

Frontmatter on pages (soft, not enforced):
- `type`: from PAGE_TYPES
- `tags`: array
- `sources`: array of source ids
- `status`: `draft | curated | auto`

### Storage

LocalStorage only. GitHub adapter is a stub. Phase 3.

### Phase 3 (deferred)

- GitHub repo backing — agents (Hermes/OpenClaw) push markdown to same repo as user.
- Restore v3 graph polish — pulses, drift animation, zoom controls, legend, compartment-style hue tinting (per-tag instead of per-compartment).
- Live multi-device sync.

---

## v4.1 (2026-05-01) — Trust primitives (Council Review response)

External council review (`AI_Wiki_Vault_Council_Review_2026-05-01.md`) flagged: docs drift, list-render bug, silent localStorage failures, no rollback for AI writes, no export/import. Addressed:

### Doc truth
- `CLAUDE.md` rewritten — was still describing v3 compartments. Now reflects v4 architecture, Karpathy method, multi-provider LLM, GitHub sync, decisions worth not re-litigating.
- `README.md` rewritten — was pointing to v1 prototype. Now lists v4 + features + status.

### `renderMarkdown` rewrite
- Old: chained regex replacements with awkward block-vs-inline interaction → list bullets + paragraphs collided on Welcome page.
- New: split-by-blank-line block parser. Each block routed: heading / hr / blockquote / unordered list / ordered list / paragraph. Inline transforms (`code` `bold` `_italic_` `[[wikilink]]` `<url>`) applied per block. Code fences pre-extracted to placeholders so inner content survives transforms.

### Visible save failures
- `LocalStorageAdapter.save` was silently swallowing errors (try/catch with empty handler). Now toasts: `⚠ Save failed: quota exceeded — export + clear or move to GitHub sync` (or whatever error message). Counter `saveErrors` shown in Settings panel.

### Export / import vault.json
- Settings → "Vault Backup & Revert" section.
- Export: downloads `vault-YYYY-MM-DD.json` with full state (pages, sources, drafts, log, batches, ui).
- Import: file picker → confirm dialog asks REPLACE or MERGE. Replace = wholesale overwrite. Merge uses `GitHubSync._merge` (newer `.updated` wins on pages, union on sources/drafts/log).

### Reversible ingest batches
- Every ingest gets a `batch_id`. All pages it creates get tagged via `Page.batch` field (set automatically through `Vault._currentBatchId`).
- New `state.batches[]` array records `{id, ts, op, sourceId, sourceTitle, provider, created[], updated[]}`.
- New `Vault.revertBatch(id)`: deletes pages tagged with that batch (skips pages whose `.batch` no longer matches → user has edited/promoted), strips `## Mentioned in` lines this batch added to updated pages, drops orphan source record. Cascade delete still applies.
- "↩ Revert last ingest" button in Settings shows last batch metadata and triggers revert.

### Skipped from review
Items council flagged that we deferred:
- File-native markdown export (`pages/*.md` instead of single JSON). Trade-off: Obsidian compat vs simplicity. Defer until core loop trusted with real use.
- Chunked retrieval / hybrid search. Defer until vault size demands.
- Test harness. Defer until file split.
- Guided onboarding tour. Welcome page exists. Defer.
- Citation chip redesign. Current works.

---

## Pre-v4 history (summary)

| Date | Note |
|------|------|
| 2026-04-26 | v1 + spec written. Compartmented research wiki idea. |
| 2026-04-26/27 | v2/v3 design iterations. Built tab shell, compartments, review queue, research missions, contradictions, claims storage. |
| 2026-04-30 | Diagnosis session: v3 felt "wonky" vs Obsidian + Karpathy. Vision relock: kill compartments, kill review queue, flat vault, three Karpathy ops. v4 written from scratch. |
| 2026-04-30 | Phase 2 — added Anthropic, then refactored to multi-provider (OpenAI/OpenRouter/Anthropic/Ollama/Custom). |
| 2026-04-30 | Karpathy gap closed: ingest now produces N pages with concept + person stubs, not 1. |
| 2026-04-30 | MockLLM tightened. Settings modal aligned. Changelog written. |

---

## Decisions worth not re-litigating

- **Single-file HTML.** No build step. Will split when modules need separate testing.
- **Flat vault, not folders-as-silos.** Type-folders in tree are *descriptive* (concepts/people/clips) not *prescriptive* (compartments). Earlier compartment model rejected after diagnosis.
- **AI writes direct, no review queue.** Karpathy method. Drafts panel reserved for Lint output + future agent-pushed suggestions.
- **localStorage now, GitHub later.** Browser-direct API calls accept the security tradeoff (personal use, key in user's own browser only).
- **Mock provider stays.** Useful for testing vault flow without burning tokens. Heuristic is intentionally dumb — switch providers for real extraction.
- **Anchors (`index.md`, `log.md`) auto-generated.** Not stored in `pages[]` — rendered on demand from vault state.

---

## Open questions / known limits

- Vault corpus capped implicitly by model context. ~200 pages → context limit. Future: vector-search chunks.
- No streaming responses. 2-5s wait on real LLM.
- API keys exposed to running page (browser-direct). Personal-use only.
- Jina Reader (URL fetch) is single point of failure. Fallback to native `fetch` not yet wired.
- Conflict resolution for multi-device GitHub sync not designed yet.
