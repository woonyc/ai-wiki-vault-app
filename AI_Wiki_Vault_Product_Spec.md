# AI Wiki Vault — Product & Architecture Spec

## One-line concept

A web-native, Obsidian-inspired AI research vault that ingests sources, turns them into linked wiki pages, keeps raw sources as truth, and gives each life/work domain its own “brain” with graph navigation and AI review control.

---

## Product positioning

This should **not** try to clone every Obsidian feature.

It should feel like:

> Obsidian’s graph-first note-taking + a cloud AI librarian + source ingestion + domain-specific research compartments.

The product should win on:
- AI-assisted source ingestion
- clean compartment separation
- citations back to raw source material
- review queue before AI changes pollute the vault
- graph navigation that stays useful instead of becoming a visual hairball
- web sync across devices

The product should not initially compete with Obsidian on:
- plugin ecosystem
- full offline-first vault behavior
- infinite customization
- local file power-user workflows

---

## User profile this is designed around

The app is designed for a user who has several active knowledge domains:

1. **Trading / Finance**
   - small-cap short selling
   - 13F research
   - stock catalysts
   - investor tracking
   - risk management
   - data-backed thinking

2. **Design**
   - motion design
   - typography
   - visual systems
   - branding references
   - creative frameworks

3. **AI Agents / Coding**
   - OpenClaw
   - Hermes
   - Claude/Cursor/vibe coding
   - mission control dashboards
   - local AI workflows

4. **Fitness / Life**
   - training logs
   - heart-rate observations
   - 2.4 km performance
   - recovery notes
   - long-term self-improvement

The app should treat these as **separate compartments** by default, while still allowing approved cross-links between them.

---

## Core mental model

The vault has four layers.

### 1. Raw Layer

Immutable source-of-truth material.

Examples:
- web articles
- YouTube transcripts
- PDFs
- copied notes
- uploaded audio/video transcripts
- trading reports
- design references
- meeting notes

Raw sources should never be overwritten. The AI can summarize and extract from them, but the original source remains preserved.

### 2. Processing Layer

AI and system jobs that transform raw sources into structured knowledge.

Tasks:
- clean article text
- extract transcript
- chunk source
- generate embeddings
- identify entities
- identify concepts
- extract claims
- suggest backlinks
- detect possible contradictions
- propose wiki updates

### 3. Wiki Layer

The living knowledge base.

Objects:
- markdown pages
- concept pages
- entity pages
- source pages
- claim records
- contradiction records
- index pages
- activity logs

### 4. Interaction Layer

User-facing interface.

Core views:
- Inbox
- Wiki
- Graph
- Research Missions
- Review Queue

---

## Required Phase 1 Features

Phase 1 should be useful even with mocked or simple AI.

### 1. Compartment system

Top-level compartments:
- Trading / Finance
- Design
- AI Agents
- Fitness / Life

Each compartment has:
- pages
- sources
- graph nodes
- AI rules
- review queue
- tags

Default behavior:
- keep knowledge private to a compartment
- suggest cross-links only when useful
- require approval for cross-compartment links

### 2. Source Inbox

User can add:
- URL
- pasted article
- manual note
- uploaded text/markdown
- later: PDF, audio, video

For the MVP, the source inbox can support:
- URL field
- title
- notes
- source type
- selected compartment

Each source should store:
- source_id
- compartment_id
- source_type
- title
- original_url
- raw_text
- clean_markdown
- created_at
- processed_at
- processing_status

### 3. AI ingestion proposal

When a source is added, AI should not directly rewrite the wiki. It should create proposals first.

Proposal types:
- create_page
- update_page
- add_backlink
- add_claim
- flag_contradiction
- merge_duplicate
- create_cross_link

Each proposal should include:
- proposal_id
- source_id
- target_page_id
- proposal_type
- suggested_content
- reasoning
- confidence
- status: pending / approved / rejected
- created_at

### 4. Wiki pages

Pages should be markdown-style.

Minimum page structure:

```markdown
---
type: concept
compartment: Trading / Finance
sources: [source_001, source_004]
confidence: medium
updated: 2026-04-26
---

# Page Title

## Summary

## Key Ideas

## Claims

## Related Pages

## Source Notes

## Open Questions
```

### 5. Graph view

The graph should not be a decorative gimmick. It should have useful modes.

Graph modes:
- Global graph
- Compartment graph
- Source graph
- Claim graph
- Cross-compartment graph

Nodes:
- compartment
- source
- concept
- entity
- claim
- contradiction
- question

Edges:
- mentions
- supports
- contradicts
- updates
- related_to
- source_of
- cross_links

### 6. Review queue

This is the most important screen.

The review queue prevents AI-generated noise from corrupting the vault.

Actions:
- approve
- reject
- edit before approve
- approve all safe/high-confidence
- defer

### 7. Search

MVP search:
- page title
- body text
- source title
- tags

Future search:
- vector search
- hybrid keyword + semantic
- graph-aware search

---

## Phase 2 Features

Phase 2 should make the app feel like an actual AI-maintained wiki.

### 1. Compartment-specific AI rules

Each compartment should have its own instruction file.

Example: Trading / Finance

```markdown
# AI Rules — Trading / Finance

- Prefer primary sources.
- Separate fact, inference, and speculation.
- Mark time-sensitive claims with dates.
- Do not turn simulation into prediction.
- Cite raw sources.
- Flag uncertainty.
- Preserve risk management context.
```

Example: Design

```markdown
# AI Rules — Design

- Extract visual principles.
- Identify typography, layout, motion, and brand strategy.
- Connect examples to frameworks.
- Allow more interpretive synthesis, but label interpretation clearly.
- Save useful references as inspiration nodes.
```

Example: Fitness

```markdown
# AI Rules — Fitness / Life

- Track logs by date.
- Separate measurable data from subjective feeling.
- Look for trends, not single-day overreactions.
- Avoid extreme recommendations.
- Keep training suggestions conservative and sustainable.
```

### 2. Claim-level storage

To support contradiction detection, store claims separately.

Claim fields:
- claim_id
- page_id
- source_id
- claim_text
- claim_type: fact / inference / opinion / speculation
- confidence
- valid_from
- valid_until
- status: active / outdated / disputed
- created_at

### 3. Contradiction detection

When a new source conflicts with an old claim, create a contradiction record.

Fields:
- contradiction_id
- old_claim_id
- new_claim_id
- summary
- severity
- status: unresolved / resolved / ignored
- resolution_note

### 4. Cross-compartment suggestions

Example:
- “Risk Management” appears in Trading, Fitness, and AI Token Budgeting.
- App suggests creating a shared concept page.

User options:
- create shared page
- keep separate
- link only this time
- ignore

### 5. Source timeline

Every compartment should have a source history timeline:
- newest sources
- recently updated pages
- unresolved proposals
- stale pages

---

## Video ingestion plan

Video support is feasible but should be phased.

### Phase 1 / 2: Transcript-first

For YouTube or uploaded video:
1. get transcript if available
2. or extract audio
3. transcribe audio
4. chunk transcript
5. summarize
6. create source page
7. propose wiki updates

This means the app understands what was said, not necessarily what was visually shown.

### Future: Visual understanding

Later support:
- frame extraction
- OCR from frames
- scene detection
- key frame captions
- timestamped visual references
- multimodal summary

This is useful for design videos, trading screen recordings, and tutorials, but should not be MVP.

### Risks with video

- large file storage cost
- slow transcription
- noisy transcripts
- multiple speakers
- copyrighted content
- YouTube transcript availability issues
- visual information lost if only transcript is used

Recommendation:
- start with transcript ingestion
- allow manual transcript paste
- add uploaded audio/video transcription later
- add visual frame understanding only after core app works

---

## Autoresearch integration

Do not integrate Karpathy-style autoresearch literally at first.

Instead, borrow the pattern:

> Research Mission = agent loop that finds sources, ingests them, updates wiki, and produces a synthesis.

Example mission:

```text
Research Topic:
“Best AI + Obsidian workflows for traders.”

Agent steps:
1. Search sources
2. Save top sources
3. Ingest each source
4. Propose wiki updates
5. Generate synthesis page
6. Suggest next questions
7. Log all changes
```

This should live under the **Research** tab.

Do not allow research agents to auto-write into the wiki without the review queue unless the user enables high-trust mode.

---

## MiroFish-style integration

MiroFish-like swarm intelligence should not be core.

Use it later as a **Scenario Simulator**.

Good use cases:
- “What could happen if a small-cap runner announces an offering?”
- “How might competitors react to this AI product trend?”
- “What are possible second-order effects of this design direction?”
- “How could this trading thesis fail?”

Bad use cases:
- precise stock prediction
- financial signals
- certainty-based forecasting
- fully automated decision-making

Label all simulation output as:
- speculative
- scenario-based
- not prediction
- for hypothesis generation only

---

## Recommended app structure

Main tabs:

1. **Inbox**
   - add URL/source
   - upload/paste content
   - run ingest
   - see processing status

2. **Wiki**
   - markdown pages
   - backlinks
   - citations
   - related pages
   - page history

3. **Graph**
   - global graph
   - compartment graph
   - source graph
   - filters
   - selected node inspector

4. **Research**
   - research missions
   - source collection
   - synthesis pages
   - next questions

5. **Review Queue**
   - pending AI proposals
   - approve/reject/edit
   - contradiction alerts
   - merge suggestions

---

## Recommended UI style

The interface should feel like:
- Obsidian-inspired
- dark mode first
- clean sidebars
- purple accent
- pane-based layout
- graph-first but not cluttered
- more structured than Obsidian
- less corporate than Notion
- more useful than a generic dashboard

Suggested layout:

```text
┌──────────────────────────────────────────────────────────────┐
│ Top bar: App name / Search / Command palette / Sync status    │
├───────────────┬──────────────────────────────┬───────────────┤
│ Compartments  │ Main content                 │ Context panel │
│               │                              │               │
│ Trading       │ Wiki page / Inbox / Graph    │ Sources       │
│ Design        │                              │ Backlinks     │
│ AI Agents     │                              │ AI proposals  │
│ Fitness       │                              │ Node details  │
│               │                              │               │
└───────────────┴──────────────────────────────┴───────────────┘
```

---

## Suggested technical stack

### Frontend
- Next.js or React
- Tailwind CSS
- Tiptap or markdown editor
- React Flow, Cytoscape.js, or custom SVG graph

### Backend
- Supabase or Postgres
- pgvector for embeddings
- background jobs with Inngest, Trigger.dev, Celery, or serverless queue

### Storage
- Supabase Storage or S3-compatible storage

### AI
- OpenAI / Anthropic / OpenRouter
- embeddings for source chunks
- LLM for extraction, linking, synthesis, and review proposals

### Scraping
- Firecrawl
- Jina Reader
- Playwright fallback
- manual paste fallback

### Transcription
- Whisper-style speech-to-text
- upload audio/video
- transcript chunking

---

## Database sketch

### compartments

```sql
id
name
slug
description
ai_rules
accent_color
created_at
updated_at
```

### sources

```sql
id
compartment_id
type
title
url
raw_text
clean_markdown
metadata_json
status
created_at
processed_at
```

### pages

```sql
id
compartment_id
title
slug
type
markdown
summary
confidence
created_at
updated_at
```

### page_sources

```sql
page_id
source_id
relationship_type
created_at
```

### graph_nodes

```sql
id
compartment_id
type
label
ref_type
ref_id
metadata_json
created_at
```

### graph_edges

```sql
id
from_node_id
to_node_id
edge_type
confidence
source_id
created_at
```

### proposals

```sql
id
compartment_id
source_id
target_page_id
type
title
suggested_content
reasoning
confidence
status
created_at
reviewed_at
```

### claims

```sql
id
page_id
source_id
claim_text
claim_type
confidence
status
valid_from
valid_until
created_at
```

### contradictions

```sql
id
old_claim_id
new_claim_id
summary
severity
status
resolution_note
created_at
resolved_at
```

---

## AI ingest prompt template

```text
You are the AI librarian for a compartment-specific research wiki.

Compartment: {{compartment_name}}
Compartment Rules:
{{compartment_rules}}

Raw Source:
Title: {{title}}
URL: {{url}}
Content:
{{content}}

Task:
1. Summarize the source.
2. Extract key concepts.
3. Extract entities.
4. Extract factual claims.
5. Suggest new wiki pages.
6. Suggest updates to existing wiki pages.
7. Suggest backlinks.
8. Flag contradictions against existing claims if any.
9. Separate fact, inference, and speculation.
10. Return structured JSON only.

Do not directly modify the wiki.
Only create reviewable proposals.
```

---

## Blind spots and constraints

### 1. AI slop risk

Without a review queue, the wiki will degrade.

Solution:
- proposals first
- approval required
- confidence scores
- source citations
- page history

### 2. Graph hairball problem

Too many nodes make graph view useless.

Solution:
- filters
- graph modes
- hide low-confidence edges
- show only neighborhood around selected node
- compartment graph by default

### 3. Token cost

Deep ingestion can get expensive.

Solution:
- ingest modes:
  - quick
  - normal
  - deep
- only deep-ingest high-value sources
- cache summaries
- avoid reprocessing unchanged sources

### 4. Scraping limitations

Some sites block extraction.

Solution:
- URL scraping
- reader fallback
- Playwright fallback
- manual paste fallback
- browser extension later

### 5. Privacy

Finance, career, and personal notes may be sensitive.

Solution:
- private by default
- exportable markdown
- clear AI provider settings
- allow local/private model option later

### 6. Portability

Do not trap notes in the app.

Solution:
- markdown export
- JSON export
- source export
- Obsidian vault export
- GitHub sync later

---

## MVP scope recommendation

Build only this first:

- authentication placeholder
- compartments
- source inbox
- mock AI proposal generation
- wiki pages
- review queue
- graph view
- local persistence
- markdown export

Do not build yet:
- real swarm simulation
- full video visual understanding
- fully autonomous research
- plugin system
- team collaboration
- complex permissioning

---

## Best first version promise

The first version should do this well:

> “I paste a source. The app extracts what matters, proposes wiki changes, lets me approve them, and shows the knowledge as pages and a useful graph.”

If this works, the app is worth continuing.

If this does not feel useful, adding Autoresearch, MiroFish, or advanced AI will not save it.
