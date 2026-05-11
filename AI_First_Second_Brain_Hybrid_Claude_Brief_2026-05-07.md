# Claude Implementation Brief: AI-First Second Brain for Woon

## 1. Executive verdict

Woon should not replace the current Karpathy-style lightweight second-brain method with a heavy “infinite brain” ontology.

The right design is a hybrid:

Karpathy-style intake remains the front door.
AI-generated typed atomic notes become the derived retrieval layer.
Specialist context packs become the actual working memory loaded by agents.

The goal is not to make Obsidian visually impressive. The goal is to make Woon’s dumped information usable by future specialist AI agents with provenance, auditability, and low-friction capture.

Core verdict:

- Keep raw sources immutable.
- Keep capture friction near zero.
- Let agents generate structure after capture.
- Use only a small number of note types and relationship types at first.
- Generate specialist context packs that agents actually load.
- Preserve human inspectability so Woon can audit important claims, decisions, and trading/finance conclusions.
- Treat AI research and finance/trading as separate specialist brains, not one undifferentiated knowledge graph.

The current Openclaw vault is already close to the right foundation. It needs an additional derived atomic layer and context-pack layer, not a total rewrite.

---

## 2. What to keep from Karpathy method

Keep these principles:

### 2.1 Low-friction capture

Woon should be able to dump:

- articles
- screenshots/OCR
- transcripts
- chats
- trading notes
- AI research links
- random thoughts
- product ideas
- agent decisions

without deciding the perfect taxonomy at capture time.

The system should not ask Woon to choose from many note types manually.

### 2.2 Context files as practical working memory

Karpathy-style context files are still valuable because agents can load them quickly.

Do not replace `CLAUDE.md`, `AGENTS.md`, `Openclaw.md`, or domain context files with only graph traversal.

Instead, use the graph/atomic layer to generate better context files.

### 2.3 Lightweight, readable Markdown

Markdown should remain the primary substrate.

Reasons:

- easy for humans to inspect
- easy for Claude/Hermes/OpenClaw agents to read
- git-friendly
- durable
- compatible with Obsidian
- avoids platform lock-in

### 2.4 Raw source preservation

Raw sources remain immutable and are the source of truth.

Existing Openclaw rule should remain:

- `raw/` is source of truth
- raw content should not be edited after ingest except metadata fixes during the same ingest pass
- synthesis must live outside raw notes

This is non-negotiable for auditability.

### 2.5 Human-readable schema note

`Openclaw.md` should remain the operational contract.

Woon may not browse the vault daily, but when an agent retrieves the wrong thing, the system must be inspectable and debuggable by a human.

---

## 3. What to adopt from the video method

Adopt only the useful parts:

### 3.1 Atomic derived notes

Create small, reusable notes extracted from raw sources and maintained pages.

These notes should represent:

- claims
- facts
- decisions
- hypotheses
- concepts
- questions
- playbooks
- entities/profiles

Atomic notes reduce token waste because agents can retrieve a precise unit instead of reading a long transcript or article.

### 3.2 Typed enough for retrieval

Use lightweight `type:` fields.

Do not create a giant ontology.

The type system should answer:

- Is this a source?
- Is this a claim?
- Is this a decision?
- Is this a hypothesis?
- Is this a playbook?
- Is this an entity?
- Is this a context pack?

That is enough to start.

### 3.3 Provenance-backed knowledge

Every extracted item must point back to source material.

Each extracted note should include:

```yaml
sources:
  - raw/articles/example-source.md
source_claim_ids:
  - optional-local-anchor-or-id
confidence: low | medium | high
review_state: raw | extracted | reviewed | promoted | deprecated
```

Agents must be able to answer:

- Where did this come from?
- Is it reviewed?
- Is it still current?
- Is it contradicted?
- Which specialist brain uses it?

### 3.4 Specialist memory packs

The most important adoption is not the graph. It is generated specialist context.

Examples:

- AI research specialist context
- finance/trading specialist context
- Hermes/Mission Control operations context
- product/business ideas context
- personal preferences context

These are the files agents actually load.

The atomic layer exists to generate these packs.

### 3.5 Contradiction and update tracking

Track contradictions and updates, but lightly.

This is especially important for:

- AI model capability claims
- benchmark claims
- pricing and product claims
- finance/trading theses
- market claims
- agent architecture decisions

However, do not overuse `supports` or `contradicts` when confidence is low. Bad edges can mislead agents.

---

## 4. What to reject for now

Reject these for the first implementation:

### 4.1 Full 16-node ontology

Do not add 16 note types.

That creates ontology debt and classification errors.

### 4.2 Manual graph maintenance

Woon should not manually maintain atomic notes, backlinks, edge types, or specialist packs.

Agents should do this.

Woon should only be asked to review high-impact ambiguity.

### 4.3 One universal brain for everything

Do not collapse AI research, finance/trading, personal decisions, product ideas, and agent operations into one flat knowledge system.

Woon needs separate specialist brains.

At minimum:

- AI Research Brain
- Finance / Trading Brain
- Hermes / Mission Control Brain

### 4.4 Obsidian graph aesthetics

Do not optimize for graph beauty.

Optimize for:

- retrieval
- provenance
- context-pack generation
- auditability
- low-friction capture

### 4.5 Automated full-vault refactor

Do not migrate the entire current vault immediately.

Start with a small implementation layer and pilot workflow.

### 4.6 Complex custom retrieval engine

Do not build custom semantic retrieval yet.

Start with:

- Obsidian-compatible Markdown
- frontmatter
- folder conventions
- filename discipline
- simple search
- context packs
- periodic generated indexes

Only build advanced retrieval after repeated failures prove the need.

---

## 5. Target architecture for Woon

The target system has four layers.

### Layer 1: Raw source layer

Purpose:

Preserve original dumped information.

Current folder:

```text
raw/
```

Rules:

- raw sources are immutable
- all raw files need provenance metadata
- raw files are not rewritten into summaries
- raw files may include OCR/text capture from screenshots
- raw files may be messy
- raw files are allowed to be long

Example raw source frontmatter:

```yaml
---
title: Example Source Title
type: source
source_url: https://example.com/article
origin: web
author:
published:
ingested: 2026-05-08
domain: ai-research
processed_by:
sha256: <hash>
review_state: raw
---
```

### Layer 2: Maintained wiki layer

Purpose:

Preserve the current Openclaw method of compiled human-readable synthesis.

Current folders:

```text
concepts/
models/
labs/
products/
use-cases/
benchmarks/
ideas/
people/
projects/
comparisons/
queries/
journal/
archive/
_meta/
```

Rules:

- maintained pages are human-readable
- maintained pages synthesize from raw sources
- maintained pages can be updated over time
- maintained pages should remain relatively durable and not be flooded with extraction noise

### Layer 3: Derived atomic layer

Purpose:

Create precise, retrievable units for AI agents.

New folder:

```text
atoms/
```

Subfolders:

```text
atoms/claims/
atoms/facts/
atoms/decisions/
atoms/hypotheses/
atoms/questions/
atoms/playbooks/
atoms/entities/
atoms/concepts/
```

Rules:

- atomic notes are derived from raw or maintained sources
- atomic notes should usually be short
- atomic notes should keep provenance
- atomic notes should be typed and review-state tagged
- atomic notes should not become a giant dumping ground

### Layer 4: Specialist context-pack layer

Purpose:

Generate compact, role-specific working memory for agents.

New folder:

```text
contexts/
```

Subfolders:

```text
contexts/ai-research/
contexts/finance-trading/
contexts/hermes-ops/
contexts/business-product/
contexts/personal/
```

Each context pack contains generated files like:

- `overview.md`
- `active-beliefs.md`
- `open-questions.md`
- `key-sources.md`
- `decisions.md`
- `playbooks.md`
- `contradictions.md`
- `agent-context.md`

Rules:

- these are generated outputs
- agents actually load these files
- they should be concise
- they should update from reviewed notes, not raw speculative drafts alone
- they should be specialist-specific, not generic

---

## 6. Concrete Obsidian folder/schema design

Keep current Openclaw folders and add only two major folders plus optional helper folders.

### Proposed directory layout

```text
Openclaw.md
Hermes.md
index.md
log.md

raw/
  clips/
  articles/
  papers/
  transcripts/
  assets/

concepts/
models/
labs/
products/
use-cases/
benchmarks/
ideas/
people/
projects/
comparisons/
queries/
journal/
archive/
_meta/

atoms/
  claims/
  facts/
  decisions/
  hypotheses/
  questions/
  playbooks/
  entities/
  concepts/

contexts/
  ai-research/
  finance-trading/
  hermes-ops/
  business-product/
  personal/
```

### Why this is the right amount of change

- keeps Openclaw structure intact
- avoids a full vault rewrite
- adds a derived machine-readable layer
- adds a generated agent-working-memory layer
- preserves human auditability
- supports domain separation

### Root-file policy

Keep root minimal:

- `Openclaw.md`
- `Hermes.md`
- `index.md`
- `log.md`
- optional high-value hub notes only

Do not place atomic clutter or generated context files at root.

### Folder responsibilities

#### `raw/`
Immutable source evidence.

#### domain folders (`concepts/`, `models/`, etc.)
Human-readable maintained knowledge.

#### `atoms/`
Derived retrieval substrate for AI.

#### `contexts/`
Generated role-specific working memory for specialist agents.

#### `_meta/`
Maintenance reports, audits, duplicate checks, index refresh reports, migration notes.

---

## 7. Minimal note types to start with

Do not use a giant ontology.

### Keep current Openclaw maintained-page types

Keep existing maintained types such as:

- `schema`
- `source`
- `concept`
- `model`
- `lab`
- `product`
- `use-case`
- `benchmark`
- `idea`
- `person`
- `project`
- `comparison`
- `query`
- `journal`
- `index`
- `log`
- `archive`
- `meta`

### Add only these new AI-first note types

For atomic layer:

- `claim`
- `fact`
- `decision`
- `hypothesis`
- `question`
- `playbook`
- `entity`
- `context-pack`

### Why these are enough

They cover the major distinctions an agent needs:

- source evidence
- verified statement
- tentative statement
- reusable workflow
- unresolved issue
- stable decision
- person/company/model/tool identity
- generated role memory

### Type guidance

- Use `fact` only for relatively stable statements with strong provenance.
- Use `claim` for statements extracted from external sources that are not yet fully verified.
- Use `hypothesis` for Woon’s or an agent’s working interpretation.
- Use `decision` for operating choices that should shape future behavior.
- Use `playbook` for reusable procedures.
- Use `entity` for important named objects not already cleanly covered by existing domain-page types.
- Use `context-pack` only for generated specialist agent memory files.

---

## 8. Minimal relationship types to start with

Do not implement 10 edge types yet.

Start with only these:

- `derived_from`
- `supports`
- `contradicts`
- `updates`
- `depends_on`
- `related_to`

### Meanings

#### `derived_from`
This note was extracted or created from another source/note.

#### `supports`
This note strengthens another claim or hypothesis.

#### `contradicts`
This note conflicts with another claim, fact, or hypothesis.

#### `updates`
This note is a newer revision or replacement of another note.

#### `depends_on`
This note’s validity or execution depends on another note.

#### `related_to`
Use when there is a meaningful relation but the exact semantics are not strong enough to justify a narrower edge.

### Relationship discipline

- only add explicit relation metadata when it improves retrieval or reasoning
- do not force relationships on every note
- if confidence is low, prefer `related_to` or no explicit edge over a wrong `contradicts`
- favor fewer, more reliable edges

---

## 9. Review-state model

Every raw, extracted, and context-pack note should have `review_state`.

Use:

- `raw`
- `extracted`
- `reviewed`
- `promoted`
- `deprecated`

### Definitions

#### `raw`
Captured but not yet extracted.

#### `extracted`
Agent has parsed the source and created derivative notes, but they are not yet trusted.

#### `reviewed`
A human or trusted reviewer agent has checked the note enough for operational use.

#### `promoted`
This is stable enough to shape specialist context packs and higher-trust synthesis.

#### `deprecated`
The note is retained for history but should not be used as a primary belief.

### Domain-specific rule

Finance/trading content should have a stricter promotion threshold than general AI research notes.

Examples:

- finance article thesis -> usually `claim`, maybe `reviewed`, not automatically `promoted`
- model release facts from official docs -> may become `fact` and `promoted` more quickly
- Woon preference or operating decision -> may become `decision` and `promoted` quickly after confirmation

---

## 10. Specialist context-pack model

Context packs are generated outputs that agents actually load.

This is the key bridge between the vault and specialist AI behavior.

### Each specialist gets a folder

Examples:

```text
contexts/ai-research/
contexts/finance-trading/
contexts/hermes-ops/
contexts/business-product/
contexts/personal/
```

### Standard files per specialist

#### `overview.md`
What this brain is for, scope boundaries, main topics.

#### `active-beliefs.md`
Current working beliefs or positions with provenance.

#### `open-questions.md`
What remains unresolved.

#### `key-sources.md`
Sources that strongly shape this specialist.

#### `decisions.md`
Operating decisions relevant to the specialist.

#### `playbooks.md`
Reusable workflows and procedures.

#### `contradictions.md`
Known tensions, conflicting claims, and unresolved disputes.

#### `agent-context.md`
Compact summary optimized for direct agent loading.

### Example uses

#### AI Research Brain
Tracks important concepts, labs, models, tooling, and Woon’s evolving architecture beliefs.

#### Finance / Trading Brain
Tracks source notes, claims, theses, invalidation notes, review states, and risk-sensitive context.

#### Hermes Ops Brain
Tracks Mission Control doctrine, routing choices, model policy, and system decisions.

### Generation rule

Context packs are not handwritten from scratch when avoidable.

They should be generated from:

- promoted decisions
- reviewed claims/facts
- maintained wiki pages
- high-signal open questions
- contradiction trackers

### Critical rule

Do not let agents query the whole vault blindly every time.

Use context packs as the first retrieval tier.

---

## 11. Mapping from current Openclaw vault to proposed design

The current Openclaw schema already has three layers:

1. raw layer
2. wiki layer
3. schema layer

The proposed design keeps those and adds:

4. atomic layer
5. specialist context-pack layer

### Current -> Proposed mapping

#### Current: `raw/`
Status: keep

Role: immutable source capture remains unchanged.

#### Current: domain folders like `concepts/`, `models/`, `people/`, etc.
Status: keep

Role: still hold maintained knowledge and synthesis.

#### Current: `Openclaw.md`
Status: keep and expand lightly

Role: operational schema and rules.

Needed change: add sections clarifying:

- atomic layer purpose
- context-pack layer purpose
- review-state model
- specialist-brain separation

#### Current: `index.md`
Status: keep

Needed change: add sections for:

- atomic notes (high-value only, likely grouped summaries rather than every file)
- specialist context packs
- relevant project notes for the rollout

#### Current: `log.md`
Status: keep

Needed change: track:

- schema additions
- context-pack generation setup
- atomic extraction rollout
- pilot-domain implementation

### New additions

#### Add `atoms/`
Purpose: machine-usable retrieval substrate.

#### Add `contexts/`
Purpose: generated working memory for specialists.

#### Optional add `projects/ai-first-second-brain-hybrid.md`
Purpose: the rollout note tracking architecture decisions and migration phases.

### What should not change yet

- Do not rename all current folders.
- Do not migrate old notes into atoms immediately.
- Do not rewrite all maintained notes into strict templates.
- Do not remove current root schema files.
- Do not break compatibility with Obsidian-native browsing.

---

## 12. First implementation changes Claude should make

Claude should make minimal, surgical changes.

### Phase 1: schema and folder scaffolding

1. Create new folders:

```text
atoms/
contexts/
```

2. Create `atoms/` subfolders:

```text
atoms/claims/
atoms/facts/
atoms/decisions/
atoms/hypotheses/
atoms/questions/
atoms/playbooks/
atoms/entities/
atoms/concepts/
```

3. Create `contexts/` subfolders:

```text
contexts/ai-research/
contexts/finance-trading/
contexts/hermes-ops/
contexts/business-product/
contexts/personal/
```

4. Update `Openclaw.md` to add:

- atomic layer
- context-pack layer
- minimal note types
- minimal relationship types
- review-state model
- specialist-brain model

### Phase 2: add starter templates

Create templates or documented patterns for:

- raw source note
- atomic claim note
- atomic fact note
- decision note
- hypothesis note
- context-pack file

### Phase 3: create pilot specialist packs

Create starter context folders/files for:

- AI Research Brain
- Finance / Trading Brain
- Hermes Ops Brain

Each should at least include:

- `overview.md`
- `active-beliefs.md`
- `open-questions.md`
- `key-sources.md`
- `agent-context.md`

### Phase 4: pilot extraction workflow

Do not migrate everything.

Instead, pick a small pilot:

Option A:
- one AI research source
- one finance source
- one Hermes-ops/system source

For each pilot source:

1. ingest raw source
2. create 2-5 atomic notes
3. connect provenance
4. update one specialist context pack
5. verify that an agent can answer better using the pack

### Phase 5: update navigation

Update `index.md` to include:

- new `atoms/` high-level section
- new `contexts/` section
- rollout project note if created

Update `log.md` with all structural changes.

### Phase 6: retrieval discipline

Document a retrieval workflow:

1. identify task domain
2. load matching context pack first
3. retrieve atomic notes second
4. inspect raw source only when needed
5. update context pack after validated learning

---

## 13. What not to overbuild yet

Do not build these yet:

- 16 note types
- 10 edge types
- full automatic graph traversal engine
- custom vector database
- full-vault conversion
- universal ontology for all domains
- automatic promotion of finance claims
- giant context packs that become unreadable
- agent writes with no review-state tracking
- full semantic dedupe for the entire vault
- mandatory explicit relation metadata on every note
- graph-visualization-driven workflow decisions

---

## 14. Acceptance criteria

Claude’s implementation is acceptable when all of the following are true.

### Structural acceptance

- `atoms/` exists with the proposed minimal subfolders
- `contexts/` exists with the proposed specialist subfolders
- `Openclaw.md` documents the new layers and rules
- `index.md` reflects the new structure
- `log.md` records the schema changes

### Workflow acceptance

- Woon can still dump information with low friction
- raw sources remain immutable
- at least one pilot AI research source produces atomic notes plus updated specialist context
- at least one pilot finance source produces atomic notes plus review-state discipline
- at least one Hermes/system source updates Hermes Ops context

### Retrieval acceptance

- an agent can load a specialist context pack without reading the whole vault
- atomic notes improve precision compared with raw-source-only retrieval
- provenance from context pack -> atomic note -> source note remains inspectable

### Safety / maintainability acceptance

- no giant ontology was introduced
- no manual classification burden was shifted onto Woon
- finance/trading claims are not silently promoted to durable truth
- humans can still inspect the system easily in Obsidian
- the new layers can be adopted incrementally without breaking current notes

---

## Final implementation principle

The winning architecture for Woon is:

**Karpathy intake + Openclaw synthesis + derived atomic memory + generated specialist context packs**

In short:

- Woon dumps
- agents structure
- sources stay intact
- context packs make specialists useful
- the graph serves retrieval, not aesthetics
