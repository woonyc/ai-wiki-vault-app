# AI Wiki Vault v4 — Five-Advisor Council Review

**Date:** 2026-05-01  
**Prototype reviewed:** `/Users/woonyencheng/Downloads/Telegram Desktop/AI_Wiki_Vault/2026-04-30-ai-wiki-vault-v4.html`  
**Repo/folder:** `/Users/woonyencheng/Downloads/Telegram Desktop/AI_Wiki_Vault`  
**Purpose:** Evaluate whether AI Wiki Vault v4 is good as a working prototype for replacing or augmenting Woon's Obsidian second brain with a custom Karpathy-style LLM wiki + web sync.

---

## Executive Verdict

**Yes — AI Wiki Vault v4 is good as a product/interface prototype.** It clearly communicates a credible AI-native second-brain direction: source ingestion, AI-maintained wiki pages, backlinks, tags, graph navigation, query, linting, and sync.

**No — it is not yet safe to treat as a real Obsidian replacement.** The prototype currently proves the shape of the experience, not the durability/trust layer needed for a personal knowledge system.

The best framing is:

> AI Wiki Vault should become an AI-native wiki compiler and query layer that can eventually make Obsidian optional — not a forced Obsidian replacement yet.

The winning next move is to keep the v4 UI direction, but harden the substrate: real file/export safety, rollback/diff, citation integrity, source immutability, and a clearer first-run ingest → query → lint loop.

---

## Prototype Context

### Current v4 direction

v4 is a single-file HTML prototype titled **AI Wiki Vault v4 — Karpathy Method**.

It uses:

- Plain HTML/CSS/vanilla JS
- D3.js for graph visualization
- PDF.js for PDF text extraction
- Browser `localStorage` for persistence
- Jina Reader for URL extraction
- Multi-provider LLM adapter:
  - Mock
  - OpenAI
  - OpenRouter
  - Anthropic
  - Ollama
  - Custom OpenAI-compatible endpoint
- GitHub sync concept via a single `vault.json` blob

### v4 product pivot

According to `CHANGELOG.md`, v4 intentionally killed much of the earlier v3 scope:

- Removed compartments as enforced silos
- Removed heavy review-queue ceremony
- Deferred research missions
- Deferred contradictions store / claims store / per-compartment AI rules
- Replaced five-tab shell with:
  - file tree
  - main page area
  - right rail
  - top-level actions: Ingest, Query, Lint, Sync, Settings

The new mental model is:

- Flat vault
- Emergent tags
- AI writes directly
- User reverts/edits later
- Anchors are auto-generated `index.md` and `log.md`
- Karpathy-style operations:
  - **Ingest**
  - **Query**
  - **Lint**

### Important repo inconsistency

There is documentation drift:

- `CHANGELOG.md` says `2026-04-30-ai-wiki-vault-v4.html` is current.
- `README.md` still points to `ai-wiki-vault-prototype.html`.
- `CLAUDE.md` still says v3 is current and describes compartments/review queue.

This should be fixed before giving the folder to another coding agent.

---

# Quadrant 1 — Where the Council Agrees

## 1. v4 is a strong prototype, but mainly at the interface/product-shape level

All advisors agree the prototype is credible as a working *experience* prototype.

It already communicates:

- A knowledge vault
- Source ingestion
- Wiki pages
- Tags
- Backlinks
- Graph
- Query over the vault
- Linting / hygiene
- Sync settings
- AI-provider flexibility

The visual design feels like a real tool for a technical knowledge worker, not a toy mockup.

## 2. The v4 simplification was likely the right move

The council broadly agrees that the v4 pivot away from the heavier v3 model was healthy.

Removed v3 concepts that were probably too much too early:

- Rigid compartments
- Review queue as a mandatory ceremony
- Research missions
- Claims/contradictions store
- Cross-compartment proposals
- Heavy multi-tab shell

The flat-vault + emergent-tags approach better matches the Karpathy LLM-wiki idea: let structure emerge from accumulated source-backed synthesis rather than over-modeling upfront.

## 3. The core promise is not “better Obsidian”

The council agrees the best promise is:

> Paste a source → AI turns it into durable linked wiki knowledge → query it later with citations → lint/refactor the vault over time.

This is meaningfully different from simply recreating Obsidian in the browser.

## 4. Trust primitives are the next bottleneck

Every advisor independently converged on trust/safety as the key issue.

Required trust primitives:

- Source preservation
- Citation traceability
- Diff / undo / rollback
- Per-page history
- Export / backup
- Linting for broken structure
- Clear AI-change visibility
- Safer sync/conflict semantics

Without these, direct AI writes can turn the vault into an AI-generated note landfill.

## 5. Do not pitch it as an Obsidian replacement yet

Council agreement:

- It can become a replacement later.
- But right now, it should be positioned as an **AI-native wiki compiler/query layer**.
- The safest path is to preserve Obsidian/Markdown compatibility so replacement remains optional.

---

# Quadrant 2 — Where the Council Clashes

## 1. Direct AI writes: breakthrough or fatal flaw?

### Contrarian view

Direct AI writes are dangerous because the current storage layer lacks the rollback/versioning that the method depends on.

Current issue:

- AI writes directly.
- Primary storage is `localStorage`.
- GitHub sync is not yet a true file-native versioning layer.
- No per-page diffs, undo stack, or rollback UI.

So the safety story says “user can revert via git/edit,” but that infrastructure is not actually present yet.

### Expansionist view

Direct AI writes are the strategic unlock. Review queues create inbox debt. If the AI can maintain the wiki directly, the vault becomes an accumulating asset rather than another chore.

### Synthesis

Direct AI writes are worth preserving as the north star, but only after adding lightweight versioning:

- Every AI write should create a change record.
- User should see affected pages.
- User should be able to revert an ingest batch.
- Git/file history should become the durable safety layer.

Recommended compromise:

> Keep direct writes, but implement AI changes as reversible commits/batches.

## 2. Single-file HTML: clever constraint or architecture ceiling?

### Expansionist view

Single-file HTML is strategically underrated:

- No install
- No build
- Easy to share
- Easy to archive
- Great for proving the loop cheaply

### Contrarian / Executor view

Single-file HTML is fine for the prototype, but it will hit limits quickly:

- Hard to test modules
- Hard to handle secrets safely
- Hard to manage large source data
- Hard to implement robust sync/conflict resolution

### Synthesis

Keep the single-file prototype while validating UX, but define an exit criterion:

- Once you need automated tests, real file storage, or robust sync, split into modules.

## 3. Flat vault vs compartments

### First Principle / Expansionist view

Flat vault is better early. It avoids premature taxonomy and lets tags/wikilinks reveal the real structure.

### Contrarian view

Flat vault may recreate the mess compartments were trying to prevent, especially across Woon’s heterogeneous domains:

- Trading
- Design
- AI agents
- Fitness/life

Same term can mean different things in different domains.

### Synthesis

Keep flat vault, but add soft scoping tools:

- Tags
- Type folders
- Optional namespaces later
- Aliases later
- Graph filters by tag/type/source

Do not reintroduce hard compartments yet.

## 4. GitHub sync: good bridge or wrong foundation?

### Outsider / Expansionist view

GitHub sync is a good personal-use bridge and can become distribution later.

### Contrarian view

One `vault.json` blob is structurally wrong for a multi-agent markdown wiki.

Problems:

- Agents cannot safely co-author individual files.
- Conflict resolution is too weak.
- Same-day edits collide because `updated` is date-level.
- “Newer wins” can silently lose work.

### Synthesis

For prototype:

- One `vault.json` is acceptable as backup/sync experiment.

For real second brain:

- Move toward file-native storage:
  - `pages/*.md`
  - `sources/*.md` or `sources/*.json`
  - `log.md`
  - `index.md`
  - Git commits per ingest/query/lint action

---

# Quadrant 3 — Blind Spots the Council Caught

## 1. The prototype does not yet prove the hardest part

It currently proves:

- UI shell
- local state
- source page creation
- concept/person stubs
- graph rendering
- provider settings
- basic query/lint flow

It does **not** yet prove:

- Durable source-grounded synthesis
- Correct updating of existing pages
- Citation precision
- Rollback after bad AI writes
- Multi-device conflict handling
- Long-term vault hygiene
- Real markdown interoperability

The key test is not “can it create pages?”

The key test is:

> After 20–50 messy real sources, is the vault easier to trust and query than Obsidian/search?

## 2. `localStorage` is a bigger risk than it looks

`localStorage` is okay for demo persistence, but risky for actual notes.

Risks:

- Browser quota limits
- Silent save failures in current adapter
- Data loss if browser storage is cleared
- Large PDFs/transcripts can overwhelm it
- No visible backup/export guarantee

This needs explicit UI warnings and export/import soon.

## 3. The GitHub/API key security model is personal-use only

The app stores provider keys and GitHub tokens in browser `localStorage`.

This is acceptable for a private prototype but not a broadly safe architecture.

Risks:

- Browser extensions
- XSS from rendered content if escaping ever fails
- CDN compromise
- Shared machine/browser profile
- Accidental screen/share leakage

If this becomes more than personal use, secrets need a backend or local key-management bridge.

## 4. The rendered Welcome page has a visible markdown/list bug

In the browser snapshot, the “Three operations” list appears partially empty/broken, with only a `log` link visible in one bullet.

This hurts first impression because onboarding is the first trust-building surface.

Likely area to inspect:

- `renderMarkdown(md)` list/paragraph transformation around lines ~1381–1403 in v4 HTML.

## 5. `CLAUDE.md` is stale and will mislead Claude Code

This is important because Woon explicitly wants a Markdown handoff file for Claude.

`CLAUDE.md` currently says:

- v3 is current
- compartments are core
- review queue is core
- AI never writes directly

But `CHANGELOG.md` says v4 killed these and is now current.

Before using Claude Code heavily, update `CLAUDE.md` or attach this review file as overriding context.

## 6. The query system will not scale as currently designed

Current query approach:

- Pack every page into the prompt.
- Slice each page to ~2500 chars.
- Ask LLM to cite page IDs.

This is fine for prototype, but will hit limits around hundreds of pages.

Future needed:

- Hybrid retrieval
- Chunking
- Page-level and source-span citations
- Cached summaries
- Query-time source selection

## 7. The graph may become a dopamine trap

Graph edges are currently derived from:

- Explicit wikilinks
- Shared tags

Shared tags can create dense, semantically weak edges.

A graph can look alive while adding little navigational value.

Needed later:

- Filters
- Local neighborhood mode
- Edge-type semantics
- Hide weak tag-only edges by default
- Source-backed edge confidence

## 8. The app needs a guided first-run loop

A new user needs to know exactly what to do first.

Recommended onboarding loop:

1. Paste URL/text.
2. Preview fetched source.
3. AI creates source page + concept stubs.
4. Show affected pages.
5. Ask a question over the new content.
6. Show citations.
7. Open graph/backlinks.
8. Export/backup reminder.

Right now, the loop is inferable but not guided.

---

# Quadrant 4 — Recommendation

## Main recommendation

Continue with v4. It is the right direction.

But do **not** expand features yet. Do **not** jump to agents, swarms, full research missions, mobile, collaboration, or complex graph intelligence.

The next phase should be:

> Turn v4 from a convincing UI prototype into a trustworthy local-first knowledge prototype.

## Recommended framing

Avoid:

> Obsidian replacement.

Use:

> AI-native wiki compiler/query layer for a Markdown-compatible second brain.

This keeps the ambition high while avoiding premature migration risk.

## Fastest path to a real working prototype

### Phase 0 — Repo truth cleanup

Fix docs before further coding:

- Update `README.md` to point to v4.
- Update `CLAUDE.md` to say v4 is current.
- Mark `AI_Wiki_Vault_Product_Spec.md` as pre-v4 / partially superseded.
- Keep `CHANGELOG.md` as the decision record.

### Phase 1 — Trustable local prototype

Must-have improvements:

1. **Fix markdown rendering bug**
   - Especially lists on the Welcome page.

2. **Add export/import now**
   - Export `vault.json`.
   - Import `vault.json`.
   - Display backup warning while using localStorage.

3. **Add reversible AI write batches**
   - Every ingest creates a batch ID.
   - Every created/updated page records that batch.
   - User can revert an ingest batch.

4. **Add visible AI-change trace**
   - “Last changed by ingest X from source Y.”
   - Show affected pages after ingest.

5. **Improve citations**
   - Source pages should clearly show source provenance.
   - Query answers should make citations visually obvious.

6. **Guided first-run flow**
   - Seed the vault with 5–10 realistic example pages/sources.
   - Add “Try this first” flow.

7. **Make save failures visible**
   - Do not silently ignore `localStorage` save errors.
   - Show warning/toast if persistence fails.

### Phase 2 — Markdown/file-native transition

Once the core loop feels good:

- Move away from one `vault.json` as the main mental model.
- Add file-native export or storage:
  - `pages/*.md`
  - `sources/*.md` or `sources/*.json`
  - `index.md`
  - `log.md`
- Preserve Obsidian compatibility.
- Consider Git as sync/versioning layer.

### Phase 3 — Retrieval and sync

Only after Phase 1 and 2:

- Chunked source retrieval
- Hybrid search
- Vector search optional
- GitHub sync with file-level commits
- Conflict resolution by file and timestamp, not whole-vault blob
- Optional backend only if absolutely necessary

## What not to build yet

Do not build yet:

- Multi-user collaboration
- Full plugin system
- Agent swarms
- Research missions
- Full video multimodal understanding
- Complex permissions
- Team sharing
- Heavy graph intelligence
- Production-grade hosted backend

These will distract from proving the core loop.

## Success criteria for the next prototype

The next version is successful if Woon can do this repeatedly:

1. Ingest 20 real sources from his AI/trading/design workflow.
2. Query the vault and get source-cited useful answers.
3. See meaningful backlinks/graph clusters.
4. Revert or clean up bad AI writes easily.
5. Export/backup with confidence.
6. Return after a few days and re-enter context faster than with raw Obsidian notes.

If this works, continue.

If this fails, more features will not save it.

---

# Advisor Notes

## Contrarian

### Core attack

The fatal flaw is that v4 killed the review queue before the safety substrate exists.

The prototype says:

- AI writes direct.
- User can revert via git/edit.

But current reality is:

- Primary storage is `localStorage`.
- No real git-backed markdown files.
- No per-page history.
- No diff preview.
- No rollback UI.

So the safety model depends on future infrastructure.

### Key warnings

- One bad ingest can pollute the vault.
- `localStorage` can fail silently.
- `vault.json` sync is not safe for multi-agent co-authoring.
- Browser-stored keys are a serious architectural ceiling.
- Current ingest mostly creates stubs, not deep synthesis.
- Graph can look useful before it is semantically useful.
- Docs are already inconsistent, which is an early governance smell.

### Contrarian recommendation

Treat v4 as a strong interface prototype, not a working knowledge-system prototype.

Next hard proof:

- Real markdown files or safe export
- Per-page diffs
- Reversible AI write batches
- Source chunks with stable citations
- Basic retrieval

## First Principle

### Core reframing

The wrong question is:

> Can this replace Obsidian?

The right question is:

> What job is Obsidian doing for Woon, and which parts should AI Wiki Vault preserve, improve, or abandon?

Obsidian provides:

- Durable Markdown files
- Daily writing interface
- Graph/navigation
- Plugins
- Sync/mobile/recovery
- Psychological trust/home base

AI Wiki Vault should not attempt to beat all of this immediately.

### First-principles recommendation

Position AI Wiki Vault as:

> An AI-native wiki compiler and query layer that may eventually make Obsidian optional.

Preserve Markdown compatibility so replacement is optional.

## Expansionist

### Core upside

If it works, this becomes more than a note app.

It becomes:

- A local cognitive operating system
- An AI-maintained cited knowledge substrate
- A domain-specific living textbook generator
- A benchmark for AI knowledge curation
- A source-to-synthesis pipeline for fast-moving domains

### Expansionist opportunities

Potential future products/uses:

- AI Systems Vault
- AI Agents Vault
- Trading Research Vault
- Design Reference Vault
- Public forkable knowledge bases
- Eval datasets for AI librarian quality
- Expert-domain living textbooks

### Expansionist caution

Do not expand too early. The core compounding loop must work first.

## Outsider

### Fresh-eye verdict

The prototype is credible but insider-heavy.

A new user may not understand:

- Karpathy Method
- Anchors
- Lint
- Drafts
- Graph semantics
- Sync safety
- Direct AI writes vs review drafts

### Outsider UX warnings

- First-run path is not guided enough.
- Empty states are too sparse.
- Icons/glyphs are cryptic.
- Text is small and low-contrast.
- Citations are not visually foregrounded enough.
- LocalStorage risk is not obvious.
- Welcome page list rendering appears broken.

### Outsider recommendation

Make the first 5 minutes obvious:

- “Paste this sample source.”
- “Here are the pages AI created.”
- “Here is why you can trust/revert it.”
- “Ask this question.”
- “See the graph update.”

## Executor

### Core buildability verdict

Yes, this can be done as a working prototype if scoped tightly.

No, it should not be treated as a robust production knowledge system yet.

### Fastest implementation sequence

1. Harden deterministic state and schema.
2. Add export/import and visible persistence errors.
3. Normalize source ingestion.
4. Validate AI operations before applying them.
5. Add reversible change batches.
6. Strengthen citation validation.
7. Keep graph/lint as projections over vault state.
8. Leave GitHub sync until the local loop is trustworthy.

### Minimum tests

Add tests or test harnesses for:

- Slug uniqueness
- Page creation/update/delete
- Source creation
- Ingest batch rollback
- Markdown rendering
- Broken wikilink detection
- Import/export roundtrip
- Provider JSON parsing
- GitHub merge conflict behavior if retained

---

# Specific Action List for Claude

## Highest priority

1. Update stale docs:
   - `README.md`
   - `CLAUDE.md`
   - clarify `AI_Wiki_Vault_Product_Spec.md` is pre-v4/superseded where applicable

2. Fix `renderMarkdown()` list rendering.

3. Add export/import buttons:
   - Export current state as `vault.json`
   - Import `vault.json`
   - Show localStorage risk warning

4. Add batch metadata to AI writes:
   - `batch_id`
   - `source_id`
   - list of created pages
   - list of updated pages
   - reversible delete/restore path

5. Add “revert last ingest” or “revert batch” feature.

6. Improve first-run onboarding:
   - sample source
   - guided action text
   - clearer empty states

## Medium priority

7. Improve citation/source visibility:
   - right rail source provenance
   - answer citations as clickable page/source chips
   - show source IDs clearly

8. Make localStorage persistence failures visible.

9. Add manual backup reminders.

10. Strengthen lint:
   - broken links
   - orphan pages
   - missing summary
   - pages without source/citation
   - duplicate title/slug
   - weak tag-only graph clusters

## Later

11. File-native markdown export.
12. Git-backed sync with real files, not only one blob.
13. Chunked retrieval / hybrid search.
14. Better graph filters.
15. Optional backend only if browser-only constraints become unacceptable.

---

# Final Recommendation to Woon

Woon should continue this project.

The v4 prototype is good enough to justify another iteration because it found a sharper thesis than the earlier compartment/review-heavy version.

But the next iteration should be boring and trust-focused, not feature-expansive.

Build the safety rail before building more magic:

- Export/import
- Version/revert
- Source provenance
- Citation visibility
- Markdown/file compatibility
- Guided first-run loop

If those work, AI Wiki Vault becomes a serious candidate for replacing parts of the current Obsidian workflow.

Until then, treat it as an AI-native companion/compiler layer — not the canonical second brain yet.
