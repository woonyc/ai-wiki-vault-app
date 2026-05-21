# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Single-file HTML prototype for AI Wiki Vault — an Obsidian-inspired, AI-assisted research wiki built around the **Karpathy LLM-wiki method**, extended in v5 with a derived **atomic layer** (claims/facts/decisions/etc.) and a **specialist context-pack layer**. Flat vault, type-folders, multi-provider LLM, optional GitHub sync. No build system, no backend, no npm. Everything runs by opening one HTML file in a browser.

**Read `CHANGELOG.md` first.** It is the authoritative decision record. `CLAUDE.md` (this file) is a quick orientation for maintainers.

## How to run

```bash
start 2026-05-13-ai-wiki-vault-v5.2.html
```

Persistence is `localStorage` (keys still `vault-v4-*` for backward compatibility — v5/v5.1 read v4 user data without migration friction). Optional GitHub sync uses a single `vault.json` blob configured in **⚙ Settings**.

## File history

| File | Status |
|------|--------|
| `ai-wiki-vault-prototype.html` | v1 — Tailwind CDN, SVG graph. Deprecated. |
| `2026-04-26-ai-wiki-vault-v2.html` | v2 — iteration. Deprecated. |
| `2026-04-27-ai-wiki-vault-v3.html` | v3 — IBM Plex + D3 + rust/lime palette. Compartment silos, review queue, research missions. **Deprecated**, kept only as graph polish reference. |
| `2026-04-30-ai-wiki-vault-v4.html` | v4 — Karpathy method, flat vault, multi-provider LLM, GitHub sync. Reference until v5 stabilizes. |
| `2026-05-11-ai-wiki-vault-v5.html` | v5 — atoms/contexts + Obsidian graph panel + review-state. Reference until v5.1 stabilizes. |
| `2026-05-12-ai-wiki-vault-v5.1.html` | v5.1 — Workflow Rework: Temp Inbox / Prepare / Review Gate / transactional Commit / manual sync gate. |
| `2026-05-13-ai-wiki-vault-v5.2.html` | **Current.** v5.1 + Shared Sandbox (Layer 0): cross-device intake via GitHub `inbox/` folder, atomic claim/process/discard, Hermes protocol. |
| `index.html` | Site entry file currently served at the custom domain. It should mirror the currently promoted live app build for `vault.woon-yc.com`. |

Earlier specs (`AI_Wiki_Vault_Product_Spec.md`) describe a pre-v4 model with compartments, review queue, claims, contradictions — **partially superseded**. Check `CHANGELOG.md` for what was killed and why.

The two 05-07 briefs (`AI_First_Second_Brain_Hybrid_Claude_Brief_2026-05-07.md` and `AI_Wiki_Vault_Workflow_Rework_Claude_Instructions_2026-05-07.md`) are the source of v5/v5.1 design.

## Architecture (v5.2)

All logic lives in one `<script>` tag. Modules are conventional: single-file but intentionally seamed.

| Module | Role |
|--------|------|
| `Vault` | Single seam over state. All reads/writes/persist route here. |
| `LocalStorageAdapter` | Primary storage. Always synchronous. |
| `GitHubSync` | Sync layer on top. Manual ↻ sync by default in v5.1; optional debounced auto-push only when `Settings.data.autoSync === true`. Pull on boot, SHA-tracked, 409 → pull+merge+retry. |
| `LLM` | Transport with two shapes: `_callOpenAI` (bearer + chat/completions) and `_callAnthropic` (x-api-key + messages, with cache_control). |
| `MockLLM` | Offline heuristic provider. Regex extraction, keyword query. |
| `PROVIDER_PRESETS` | Mock / OpenAI / OpenRouter / Anthropic / Ollama / Custom. |
| `SourceFetcher` | URL (Jina Reader), PDF (pdf.js), text adapters. |
| `IngestFlow` | v4 single-shot ingest modal. Still wired to `⊕ Direct` topbar button + Cmd-I. State machine: idle → fetching → preview → committing → done/error. |
| `IngestPipeline` | v5.1 staged workflow. v5.2 keeps `normalize()` / `prepare()` / `commit(action)` / `archiveDraft()` / `discardDraft()`. `capture()` is no longer the primary entry — `SharedInbox.capture()` is. `commit()` now also moves the originating shared-inbox file to `processed/` when the draft has `shared_capture_id`. |
| **`SharedInbox`** | **v5.2 primary intake seam.** Cross-device sandbox backed by GitHub `inbox/{captured,claimed,processed,discarded}/cap-*.json`. Public API: `capture()`, `list(state)`, `refresh()`, `claim()`, `process()`, `discard()`, `releaseStale()`, `flushPendingPush()`, `status()`. |
| `GitHubInboxAdapter` | v5.2 GitHub Contents API adapter for the inbox folder. `listFolder()` (with `If-None-Match` ETag), `getFile()`, `putFile()`, `deleteFile()`. |
| `LocalInboxCache` | v5.2 offline cache + pending-push queue, in own localStorage key `vault-v4-inbox-cache`. Independent persist from `Vault`. |
| `TempItem`, `IngestDraft`, `IngestLogEntry` | v5.1 factories backing the staged pipeline. |
| `detectFinance(text)`, `canonicalizeUrl(u)` | Deterministic helpers used by `IngestPipeline.normalize` to flag `review_required` and strip tracking params. |
| `LinkResolver` | Explicit/backlink/tag-peer derivation. |
| `GraphProjection` | Pure vault → `{nodes, edges}`. |
| `Anchors` | Auto-renders `index.md` (catalog by type) + `log.md` (append-only ops history). |
| `PAGE_TYPES` | 15 types across 5 layers (raw / wiki / atoms / contexts / meta). Each carries `{folder, glyph, label, order, layer}`. Drives tree folders, glyphs, graph color, anchor index ordering. |
| `REVIEW_STATES` | `raw → extracted → reviewed → promoted → deprecated`. Stored on `Page.review_state`. Default per layer (raw=raw, atoms=extracted, others=reviewed). |
| `Views` | Registry includes workflow and content surfaces: `page`, `drafts`, `graph`, `search`, `tag`, `inbox`, `review`, `ingestlog`, `backups`. |
| `Settings` | Multi-provider + GitHub config + `graphCfg` (Groups/Filters/Colors/Display/Forces) persisted to localStorage. |
| `getGraphCfg() / saveGraphCfg()` | Graph view panel state. Defaults in `GRAPH_DEFAULTS`. |
| `nodeGroupColor(p, groups)` | Per-query color override for graph nodes. Query syntax: `tag:` `type:` `layer:` `path:` `status:` `review:` or bare title regex. |
| `filterPagesForGraph(pages, cfg)` | Layer / draft / orphan / tag-substring filtering before graph build. |

## Domain model

**Karpathy operations still matter, but the default workflow changed in v5.1.**

### Primary path (v5.2)
- **Capture** — paste URL/PDF/text. Writes to **Shared Inbox** (GitHub `inbox/captured/`) so PC + Mac + Hermes all see it. No vault mutation.
- **Refresh (manual)** — `↻ Inbox` pulls new captured/claimed items from GitHub. No auto-poll.
- **Claim + Prepare** — atomic move to `inbox/claimed/` (other devices see lock), then deterministic normalization into ingest drafts.
- **Review** — inspect title, type, warnings, summary, suggested action.
- **Commit local** — write to vault with pre-commit backup snapshot. Originating inbox file atomically moves to `inbox/processed/`.
- **Sync remote** — manual GitHub push via `↻ Sync` for the curated `vault.json`. Inbox state was already on GitHub all along (separate seam).
- **Discard** — atomic move to `inbox/discarded/` (kept for audit).

### Legacy fast path
- **`⊕ Direct` / Cmd-I** — keeps the old single-shot ingest modal for explicit bypass of the staged workflow.

### Other operations
- **Query** — vault corpus in cached system prompt → answer with citations → optionally save as synthesis page.
- **Lint** — heuristic scan for broken wikilinks, orphans, missing summaries → drafts.

**Page types** drive folders, glyphs, graph color, and ordering. v5 adds atoms + contexts:

Wiki layer (human-maintained, default `review_state: reviewed`):
- `concept` rust · `person` violet · `synthesis` amber · `note` gray

Raw layer (immutable source evidence, default `review_state: raw`):
- `source` teal · `clip` dim ink

Atoms layer (derived retrieval substrate, default `review_state: extracted`):
- `claim` orange · `fact` gold · `decision` red · `hypothesis` pale orange · `question` pink · `playbook` purple · `entity` teal

Contexts layer (generated agent working memory, default `review_state: reviewed`):
- `context-pack` mint green

Meta:
- `anchor` ink — `index.md` + `log.md`

URL pattern detects clip vs source. Clips get `YYYY-MM-DD handle` title prefix.

**Cascade delete** — deleting a page also deletes draft stubs created from it (body matches `(Created from|First mention via) [[deleted-title]]`). Other references degrade to plain text.

## Design system

CSS variables live in `:root`. IBM Plex Sans + IBM Plex Mono. Rust `#d97757` accent + lime `#CCFF00` highlights. Slim dark scrollbar.

Layout: `240px tree | 1fr main | 220px rail` grid. 48px topbar + 24px statusbar.

Graph: D3 force sim with murmuration `flow` force (per-node sin/cos phase + swarm-avg drift). Hub nodes (top-3 by degree) get lime ring. Type-clustered with `forceX/Y` cohesion. Random neuron pulse every 1400ms. v5 adds an Obsidian-style top-left control panel — Groups (per-query color overrides) / Filters (layer + draft + orphan + tag) / Colors / Display / Forces — persisted on `Settings.data.graphCfg`. Background defaults to pure black, links to rust-brown `#6b3a2a`.

When editing: use CSS variables, not hardcoded hex.

## What is mocked

Heuristic extraction in `MockLLM`. Real LLM calls require a user-configured key in **⚙ Settings** (OpenAI/OpenRouter/Anthropic/Ollama/Custom). Real URL fetch uses Jina Reader (no auth). PDF parsing uses pdf.js. GitHub sync requires a user PAT.

## Decisions worth not re-litigating

- Single-file HTML. No build step.
- Flat vault with **type-folders** (descriptive), not compartment silos.
- **Primary path is staged review**, not auto-mutate-on-capture.
- `⊕ Direct` remains only as an explicit fast-path escape hatch.
- localStorage remains primary for active canonical-vault editing; the Shared Inbox is GitHub-backed and is the cross-device intake source of truth.
- Browser-direct API calls — keys in localStorage, personal-use only. No backend.
- Single `vault.json` blob on GitHub. File-native markdown export is deferred until the core loop is trusted.

## Things to maintain when editing

- Update `CHANGELOG.md` for any architectural decision.
- New file format / schema bump → migration in `Vault.migrate()`. Bump `schemaVersion` and gate the upgrade block on it.
- New page type → add to `PAGE_TYPES` **with a `layer` value** so graph filters and tree ordering work. Add a `TYPE_COLORS` entry too.
- New provider → add a row in `PROVIDER_PRESETS`. `LLM._callOpenAI` covers OpenAI-compatible shapes.
- New view → add it to the `Views` registry and the tree/topbar wiring if user-facing.
- LocalStorage keys are still `vault-v4-*`. Do not rename without writing a migration that reads the old keys.
- If promoting a new HTML file live, update `index.html` (or intentionally redirect) so the deployed site stops serving the old version.

## v5.2 next (deferred from v5.1)

Atom extraction + context-pack generation per the Hybrid Brief. Specifically:
- `LLM.extractAtoms(text)` → produce `claim` / `fact` / `decision` / `hypothesis` / `question` pages from a source, with `derived_from` provenance + `confidence`.
- Wire into `IngestPipeline.commit` as a third action: `'commit_with_atoms'`.
- Context-pack generator: read promoted atoms by `specialist`, produce `state.contextPacks[domain].{overview, active-beliefs, open-questions, key-sources, decisions, playbooks, contradictions, agent-context}`.
- Promote / deprecate / mark-reviewed UI for the `review_state` lifecycle.
- Two-layer Kimi worker + reviewer agent routing (Brief §Model routing).
- Typed relationships (`derived_from`, `supports`, `contradicts`).
