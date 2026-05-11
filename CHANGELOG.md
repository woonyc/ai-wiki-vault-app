# AI Wiki Vault — Changelog

Personal Karpathy-method second brain. Single-file HTML app. No backend (yet).

## Files

| File | Status |
|------|--------|
| `ai-wiki-vault-prototype.html` | v1 — purple Obsidian theme, SVG graph |
| `2026-04-26-ai-wiki-vault-v2.html` | v2 — iterated design |
| `2026-04-27-ai-wiki-vault-v3.html` | v3 — IBM Plex + D3 graph + rust/lime palette. Compartment silos, review queue, research missions. **Deprecated** — design reference for graph polish only. |
| `2026-04-30-ai-wiki-vault-v4.html` | v4 — Karpathy method, flat vault, type folders, multi-provider LLM, GitHub sync. Reference until v5 stabilizes. |
| `2026-05-11-ai-wiki-vault-v5.html` | v5 — atoms/contexts layers + Obsidian graph panel + review-state. Reference until v5.1 stabilizes. |
| `2026-05-12-ai-wiki-vault-v5.1.html` | **Current.** v5 + Workflow Rework: Temp Inbox · Prepare · Review Gate · transactional Commit with backup · manual GitHub sync gate. |
| `AI_Wiki_Vault_Product_Spec.md` | Original spec (pre-Karpathy pivot). Many decisions superseded. |
| `AI_First_Second_Brain_Hybrid_Claude_Brief_2026-05-07.md` | Hybrid architecture brief — Karpathy intake + Openclaw synthesis + atoms + contexts. Partial implementation in v5; pilot extraction deferred. |
| `AI_Wiki_Vault_Workflow_Rework_Claude_Instructions_2026-05-07.md` | Workflow rework brief — Temp Inbox / Prepare / Drafts / Review Gate / Local Commit / Manual Sync. Deferred to v5.1. |
| `CLAUDE.md` | Repo guide for Claude Code. |
| `CHANGELOG.md` | This file. |

---

## v5.1 (2026-05-12) — Workflow Rework: Temp Inbox + Review Gate

Source brief: `AI_Wiki_Vault_Workflow_Rework_Claude_Instructions_2026-05-07.md`.

### Mental model shift
v4: `capture → ingest → sync` (one button mutates vault + auto-pushes).
v5.1: `capture → normalize → draft → review gate → commit local → sync remote` (each step reversible; remote sync is a manual button with a pre-flight gate).

### Schema bump (2 → 3)
- `state.tempItems[]` — captured raw input (Brief §Step 1).
- `state.ingestDrafts[]` — normalized draft packets awaiting review (Brief §Step 3).
- `state.ingestLog[]` — append-only state-transition log per draft.
- `state.backups[]` — pre-commit snapshots (`pages`, `sources`, `contextPacks`); cap last 5.
- `state.sync = { last_synced_remote_sha, last_synced_at, sync_status }` — explicit sync metadata.
- `state.revision` — bumped on every commit (so a future device knows about local-only changes).
- `state.device_id` — random per browser (Brief §Required vault metadata additions).
- LocalStorage keys still `vault-v4-*` — v5.1 reads v4 user data and migrates in place.

### New module: `IngestPipeline`
Replaces v4's single-shot ingest path with a staged state machine. Public API:

| Method | Brief step | What |
|--------|-----------|------|
| `capture(rawInput, opts)` | §Step 1 | Push to Temp Inbox. Stores raw + hash + timestamp. No LLM, no vault write. |
| `normalize(tempItemId)` | §Step 2 | Deterministic. Splits multi-URL paste. `canonicalizeUrl` strips utm/fbclid/etc. Detects clip vs source. Runs `detectFinance()` to set `review_required` + `specialist:'finance-trading'`. Creates one `IngestDraft` per normalized item. |
| `prepare(draftId)` | §Step 3 | Marks `needs_review`. (LLM enrichment hook for v5.2.) |
| `commit(draftId, action)` | §Step 4-5 | Pre-flight backup snapshot. Two paths: `'commit'` runs full Karpathy ingest (source page + concept stubs + person stubs + auto-links). `'dated_source_only'` writes a single dated page with no extraction (Brief §6 finance default). Bumps revision, marks `sync_status='pending'`. |
| `archiveDraft(id)` / `discardDraft(id)` | §Step 4 | Soft delete with state log. |

### Auto-sync killed
`Vault.persist()` previously called `GitHubSync.schedulePush()` on every state change. v5.1 gates this on `Settings.data.autoSync === true` (default false). Manual ↻ Sync button is the only push path. Per Brief §1: "do not auto-sync after every ingest".

### Manual sync gate
↻ Sync button now:
- Warns if drafts in `draft_ready` / `needs_review` (won't push, but flags they exist).
- On success: writes `state.sync.last_synced_at` + `last_synced_remote_sha` + `sync_status='synced'`.
- On failure: `sync_status='failed'` (state stays committed locally — Brief §Step 6).

### Finance quarantine wired (Brief §6)
`detectFinance()` (added in v5) now invoked at normalize time. Hits set:
- `draft.review_required = true` (UI: pulsing red REVIEW REQUIRED pill)
- `draft.specialist = 'finance-trading'` (green pill)
- `draft.suggested_action = 'dated_source_only'` (won't auto-promote to durable concepts)
- `draft.reasons_review_required = [...]` shown in the review card warning box

`TICKER_RE` extracts `$AAPL` style tickers and stores them on the draft.

### New views (added to `Views` registry)
- `inbox` — Temp Inbox cards. Bulk "Prepare all". Stale warnings at >10 (orange) / >25 (red).
- `review` — Draft cards with all Brief §Step 4 fields: title, source URL/domain, type, category, summary, tickers, warnings, review_required + reasons, provenance snippet, edit-in-place title.
- `ingestlog` — Append-only state transitions per draft.
- `backups` — List of last 5 snapshots with restore action.

### Tree chrome (left sidebar)
New top-level **Workflow** section with `⊞ Temp Inbox`, `↯ Review`, `⌚ Ingest log`, `⎙ Backups`. Inbox + Review show pending counts (rust at >0, red if any `review_required`). "SYNC PENDING" label appears next to section header when local commits await push.

### Topbar
- `⊕ CAPTURE` (primary) — opens Capture modal → Temp Inbox.
- `⊕ Direct` — kept as escape hatch to v4 single-shot ingest modal (Cmd-I).
- ↻ Sync, ⚙ Settings unchanged in placement, gate-aware behavior.

### Backward compat
- v5 graph aesthetic, atoms/contexts types, review_state pills — all preserved.
- v5 `getGraphCfg`, `Settings.save` flow — preserved.
- v4/v5 vault.json structure — readable. New fields are added on migration; old fields untouched.

### What is NOT in v5.1 (deferred to v5.2)
- LLM-driven atom extraction (claim/fact/decision pages) — IngestPipeline.commit still uses v4-shaped output (source page + concept/person stubs).
- Context-pack generation per specialist (overview / active-beliefs / open-questions / etc).
- Two-layer Kimi worker + reviewer agent routing (Brief §Model routing).
- Typed relationships (`derived_from`, `supports`, `contradicts`, etc).
- Promote / deprecate / mark-reviewed UI for review_state lifecycle.
- File-native markdown export (still single vault.json blob).

---

## v5 (2026-05-11) — Atoms + Contexts + Obsidian graph aesthetic

Source briefs:
- `AI_First_Second_Brain_Hybrid_Claude_Brief_2026-05-07.md`
- `AI_Wiki_Vault_Workflow_Rework_Claude_Instructions_2026-05-07.md`

### Vision lock
Karpathy intake stays as the front door. The vault now models four layers (Brief §5): raw → maintained → atoms (derived retrieval substrate) → contexts (generated specialist agent memory). v5 adds the type system, review-state, and graph affordances. Workflow rework (Temp Inbox / Prepare / Review Gate / Local Commit) is split into v5.1 to keep the diff auditable.

### Schema bump (1 → 2)
- `Page.review_state` field — `raw | extracted | reviewed | promoted | deprecated` (Brief §9). Backfilled per layer at migration time.
- Optional `Page.specialist` (`ai-research | finance-trading | hermes-ops | business-product | personal`).
- Optional `Page.confidence` (`low | medium | high`) and `Page.review_required` (boolean).
- `state.contextPacks` — generated specialist memory blobs (Brief §10). Empty by default.
- LocalStorage keys (`vault-v4`, `vault-v4-settings`, `vault-v4-github`) **kept unchanged** so v5 reads existing v4 data without migration friction.

### `PAGE_TYPES` extension (Brief §3.1, §7)
Added eight types in two new layers:

| Type | Layer | Folder | Glyph |
|------|-------|--------|-------|
| `claim` | atoms | `atoms/claims` | ❝ |
| `fact` | atoms | `atoms/facts` | ✓ |
| `decision` | atoms | `atoms/decisions` | ⊞ |
| `hypothesis` | atoms | `atoms/hypotheses` | ? |
| `question` | atoms | `atoms/questions` | ¿ |
| `playbook` | atoms | `atoms/playbooks` | ▷ |
| `entity` | atoms | `atoms/entities` | ◉ |
| `context-pack` | contexts | `contexts` | ▣ |

Each type carries `layer: 'wiki' | 'atoms' | 'contexts' | 'raw' | 'meta'`. Tree, anchor index, and graph all order by `PAGE_TYPES[*].order` so atoms cluster after wiki and contexts after atoms.

### Finance quarantine helpers (Workflow Brief §6)
- `FINANCE_KEYWORDS` array + `TICKER_RE`.
- `detectFinance(text)` returns `{ isFinance, hits, tickers }`. Used by future Prepare step to set `review_required = true` automatically. Plumbed into the Page factory via `review_required` argument; UI surface = pulsing red pill in page meta.

### Graph view rewrite — Obsidian aesthetic
Reference: Obsidian forum thread on multiple-graph-view filters and color groups. Animation behavior preserved (murmuration `flow` force, neuron pulse every ~1.4s, hub lime ring, type clustering, hover isolation, drag, zoom).

Replaced the bottom-right legend with a 5-section collapsible panel pinned top-left:

| Section | Controls |
|---------|----------|
| **Groups** | Per-query color overrides. Query syntax: `tag:x` · `type:x` · `layer:x` · `path:x` · `status:x` · `review:x` · or bare regex on title. Color picker per row. |
| **Filters** | Toggle wiki / atoms / contexts / raw / drafts / orphans. Tag substring filter. Filters mutate the node set, not just visibility (so simulation also gets faster). |
| **Colors** | Background / link / link-hover / label / hub. Defaults: pure black bg, rust-brown link `#6b3a2a`, rust hover. |
| **Display** | Toggle labels, node scale, link opacity, label-min-degree threshold. |
| **Forces** | Repel / link length / center pull / type cluster / murmuration flow intensity. All wired to the existing simulation. |

Persisted on `Settings.data.graphCfg`. Reset button restores defaults.

### TYPE_COLORS extended for new types
- atoms layer: warm orange/red family (claim `#ff8c5a`, fact `#ffb347`, decision `#e74c3c`, hypothesis `#ffc870`, question `#ff6b9d`, playbook `#a78bfa`, entity `#5fb8c9`).
- contexts layer: mint green `#7ee787` — visually signals "agent working memory" vs human-maintained synthesis.

### What is NOT in v5 (deferred to v5.1)
Workflow Rework brief explicitly. Specifically:
- Temp Inbox capture surface
- `Prepare for ingest` deterministic normalizer
- Ingest Drafts / Review Cards
- Transactional local commit with backup snapshot
- Finance review-required enforced at Prepare time (helper exists, not yet wired into IngestFlow)
- Two-layer Kimi/reviewer routing
- Dated-source-only / archive / discard actions

### Things v5 deliberately did not change
- LocalStorage keys (backward compat with v4 user data).
- Existing seed pages (Welcome + Karpathy LLM Wiki) — still curated.
- Anthropic / OpenAI / OpenRouter / Ollama transports — unchanged.
- Murmuration animation, hub detection, neuron pulse, link cross-cluster lime accent — preserved.

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
