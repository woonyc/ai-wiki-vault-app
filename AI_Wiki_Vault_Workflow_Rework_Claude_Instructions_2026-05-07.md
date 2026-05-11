# AI Wiki Vault — Workflow Rework Instructions for Claude

> Prepared for Claude after a 3-pass GPT-5.5 review/synthesis of Woon's proposed ingest workflow rework.
> Date: 2026-05-07
> Project: `AI_Wiki_Vault`
> Current app: `2026-04-30-ai-wiki-vault-v4.html`
> Companion brief for the broader AI-first second-brain architecture: `AI_First_Second_Brain_Hybrid_Claude_Brief_2026-05-07.md`

## Executive verdict

Proceed with the rework, but **do not** implement it as a single opaque "Data Sort & ingest" button.

The correct v1 shape is:

**Temporary capture area**
+ **deterministic normalization**
+ **AI-generated ingest drafts**
+ **explicit review gates**
+ **finance quarantine**
+ **local transaction safety**
+ **manual guarded GitHub sync**

The current app's problem is not just model quality. The problem is that noisy inputs can move too directly toward durable vault state. Finance content makes this especially dangerous because summaries can accidentally turn time-sensitive opinion or promotion into long-lived "facts."

## What Claude should change conceptually

### Replace this mental model

`capture -> ingest -> sync`

### With this mental model

`capture -> normalize -> draft -> review gate -> commit local -> sync remote`

Important: **drafts are not vault knowledge**.

## Product position for this rework

This rework should make Woon feel:

> "I can dump anything quickly, then later safely prepare, review, and commit only what deserves to become durable knowledge."

This is **not** a push toward more automation first. It is a push toward **trustworthy staged ingestion**.

## Main design decisions

### 1. Keep GitHub sync

GitHub sync is still necessary because Woon wants vault data available across devices.

But in v1:
- keep sync **manual and guarded**
- do **not** auto-sync after every ingest
- do **not** treat "committed locally" as equivalent to "synced remotely"
- refuse overwrite when remote changed

### 2. Add a temporary inbox / sandbox

Add a first-class staging area for raw source capture.

This inbox should accept messy input without forcing structure at capture time.

Accepted inputs:
- one URL
- many URLs in one paste
- pasted article text
- note + URL
- copied newsletter
- YouTube/video link
- tweet/X link
- PDF link
- raw personal note like "check this later"

### 3. Rename the main action

Do **not** use `Data Sort & ingest` as the first button name.

Use:
- `Prepare for ingest`

Why:
- reduces false promise
- makes it clear the first step is reversible preparation, not canonical mutation
- separates staging from commit

### 4. Finance must be quarantined by default

Finance/trading sources should **not** directly update durable wiki truth by default.

Default finance outcomes should be one of:
1. **Dated market note**
2. **Trade thesis draft**
3. **Company fact update** only if strongly source-backed and approved
4. **Archive**
5. **Discard**

For v1, any finance/ticker/trading content should be marked `review_required = true`.

## Required v1 workflow

### Step 1 — Capture

User dumps source into Temp Inbox.

At capture time, store minimal fields only:

```js
temp_item = {
  id,
  raw_input,
  input_hash,
  captured_at,
  user_note,
  source_hint,
  status,        // captured
  warnings,
  normalized_item_ids
}
```

Rules:
- do not summarize yet
- do not deeply classify yet
- do not mutate vault notes yet
- do not run expensive AI unless explicitly requested

### Step 2 — Normalize

When user clicks `Prepare for ingest`, do deterministic work first before any LLM call:

- split multi-URL dumps into separate items
- detect URL vs pasted text
- canonicalize URLs
- fetch title and readable text where possible
- compute content hash
- detect exact duplicates
- detect source domain
- extract obvious dates
- detect finance/trading keywords
- detect tickers with regex
- detect extraction failure or partial extraction
- detect whether item was already processed

This step should be cheap and idempotent.

### Step 3 — Create ingest drafts

After normalization, AI should create **draft packets**, not final vault writes.

Draft schema:

```js
ingest_draft = {
  id,
  temp_item_id,
  source_id,
  cleaned_title,
  source_type,
  domain,
  proposed_category,
  short_summary,
  key_claims,
  entities,
  tickers,
  dates,
  source_quality,
  confidence,
  suggested_action,
  review_required,
  reasons_review_required,
  provenance_snippets,
  proposed_vault_changes,
  cost_metadata,
  warnings
}
```

Important:
- the draft is reviewable
- the draft can be edited
- the draft can be archived or discarded
- the draft is not yet part of canonical vault state

### Step 4 — Review gate

Prepared items should render as review cards.

Each card should show:
- title
- source URL/domain
- source type
- proposed category
- short summary
- 3-7 key points
- detected entities/tickers
- source date / captured date
- provenance snippet(s)
- warnings
- whether review is required
- why review is required
- proposed vault changes

Card actions:
- `Commit`
- `Save as dated source only`
- `Needs review`
- `Merge manually`
- `Archive`
- `Discard`
- `Re-run deeper`
- `Edit draft`

### Step 5 — Commit locally

Only after approval:
- validate schema
- create backup snapshot
- apply changes to a cloned state
- run integrity checks
- replace canonical local vault only after validation
- append ingest log
- mark sync state as pending

Commit must be transactional in spirit, even if still implemented in single-file/localStorage form.

### Step 6 — Sync manually to GitHub

Manual button:
- `Sync to GitHub`

Before push:
- fetch latest remote SHA
- compare with `last_synced_remote_sha`
- refuse overwrite if remote changed
- validate JSON before upload
- push `vault.json`
- update sync metadata

If sync fails:
- local committed state remains valid
- state becomes `sync_failed`
- user gets a recovery/export path
- do not silently roll back successful local ingest

## Required state model

Use explicit states instead of loose booleans.

```js
captured
normalized
draft_ready
needs_review
approved
committed_local
sync_pending
synced
archived
discarded
failed
```

## Required vault metadata additions

The vault state should grow to include at minimum:

```js
vault = {
  schema_version,
  vault_id,
  revision,
  updated_at,
  device_id,

  temp_items: [],
  normalized_items: [],
  ingest_drafts: [],
  raw_sources: [],
  notes: [],
  ingest_log: [],

  sync: {
    last_synced_remote_sha,
    last_synced_at,
    sync_status
  },

  backups: []
}
```

## Finance-specific hard rules

For v1, enforce deterministic finance review triggers.

If source contains any of the following, set `review_required = true`:
- ticker symbols
- market/trading keywords
- earnings
- SEC
- price targets
- analyst rating
- short squeeze
- catalyst framing
- stock movement commentary
- company valuation claims

For finance claims, require these fields before durable promotion:
- `claim_type`: factual | reported | opinion | forecast | promotional | catalyst | historical_price_action | user_thesis
- source URL
- source date or captured date
- provenance snippet
- confidence
- ticker/entity mapping
- freshness or expiry flag where relevant

Default finance path for v1:

`captured -> draft -> finance review -> dated source note / trade thesis draft / archive`

Not:

`captured -> summarized -> durable company truth`

## Model / agent routing recommendation

### Do not let Kimi 2.6 ingest alone

Kimi 2.6 should be used as a **cheap first-pass worker**, not as the sole authority for canonical ingest.

Use a simple v1 two-layer pattern:

1. **Kimi worker** prepares the draft
2. **Reviewer agent / stronger model** checks the draft
3. Only then can the item be approved for local commit

This is effectively a lightweight **generator -> evaluator** loop, not a large multi-agent swarm.

### Kimi 2.6 should handle

- boilerplate removal
- first-pass cleanup
- title cleanup
- source type guessing
- category guessing
- first-pass summary
- ticker/entity extraction
- duplicate candidate detection
- draft generation for review cards
- cheap batch preparation

### Reviewer agent / stronger model should handle

- checking whether the draft actually matches the source
- catching overclaiming, missing nuance, or stale interpretation
- final durable-ingest approval logic
- ambiguous finance claims
- contradiction detection
- merge decisions
- schema consistency
- provenance enforcement
- high-value synthesis
- deciding whether a claim becomes durable vault knowledge

### Review outcomes

The reviewer layer should be able to return one of these outcomes:

- `approve_commit`
- `approve_source_only`
- `needs_manual_review`
- `archive`
- `discard`
- `rerun_deeper`

### Key principle

A cheap model may help decide *what to prepare*, but it should not silently decide *what becomes trusted knowledge*.

For this vault, the right v1 doctrine is:
- **Kimi = worker**
- **reviewer agent / stronger model = checker**
- **Woon = final approver for important or finance-sensitive items**

Do not build a large permanent agent network for every source. Start with one reviewer/checker layer only, and escalate only when the item is ambiguous or high-stakes.

## Backlog policy

Do not make reminders naggy at first.

Use levels:
- 0-5 items: healthy
- 6-10 items: light warning/badge
- 11-25 items: recommend batch processing
- 26+ items: show stale/risky inbox warning, but still allow capture unless storage risk is real

Provide quick actions instead of hard blocks:
- `Process top 5`
- `Prepare finance only`
- `Prepare AI only`
- `Discard obvious duplicates`
- `Archive older than X days`
- `Quick skim mode`
- `Deep ingest mode`

For v1:
- show inbox badge when >0
- warning at >10
- stronger warning at >25
- night reminder can come later after the prepare/review loop is proven

## What not to build yet

Do **not** build these in the first implementation pass:
- fully automatic ingest from temp inbox into canonical vault
- automatic finance article promotion into durable notes
- auto-sync after every ingest
- complicated reminder scheduler before the core loop works
- semantic full-vault duplicate matching for every item
- rich multi-device merge UI beyond guarded remote-SHA checks
- heavy dashboard/polish before trust/recovery/provenance work
- strong-model review for every single source
- hard capture blocking based on backlog alone
- one magic button that prepares, commits, and syncs in one opaque step

## Implementation milestone Claude should target first

### Milestone name
`Temp Inbox + Prepare Drafts + Local Commit`

### Goal
Prove the workflow on 20-50 messy real sources, especially finance articles.

### Milestone scope

#### A. Temp Inbox
Implement:
- paste box
- captured item list
- raw input preview
- timestamp
- item count
- delete
- archive
- warning at >10
- stronger warning at >25

#### B. Prepare for ingest
Implement:
- split multiple URLs
- canonicalize URLs
- detect source type
- detect duplicates via hashes
- detect finance/ticker keywords
- fetch readable text where possible
- preserve raw input always
- create ingest drafts

#### C. Review cards
Implement cards with:
- title
- source URL/domain
- proposed category
- summary
- key points
- detected tickers/entities
- warnings
- `review_required`
- reasons for review
- proposed action buttons

Actions for v1:
- Ingest
- Save as dated source only
- Archive
- Discard
- Needs review

#### D. Local commit with backup
Implement:
- backup snapshot before commit
- schema validation
- save raw source
- save cleaned draft
- create/update note only when approved
- append ingest log
- increment revision
- mark sync pending

#### E. Export / import
Implement:
- export `vault.json`
- import/restore `vault.json`
- visible last backup time

#### F. Manual GitHub sync
Implement:
- show last synced remote SHA
- check remote SHA before push
- refuse overwrite if remote changed
- push only validated `vault.json`
- visible sync status

## Acceptance criteria

This milestone is only successful if all of the following are true:

- raw input is never silently lost
- re-running prepare does not duplicate items
- finance articles do not become durable facts without review
- every committed claim has provenance
- local backup exists before batch commit
- failed items remain visible and recoverable
- sync refuses overwrite if remote changed
- user can archive noisy sources without guilt or confusion

## Suggested file/code areas to amend

Based on current v4 structure, Claude should likely inspect and update at least:
- `Vault`
- `Vault.migrate()`
- `GitHubSync`
- `IngestFlow`
- `SourceFetcher`
- `LLM.ingest(...)`
- views/modals around ingest and source capture
- status bar / sync state rendering
- any anchor/index/log generation that needs new ingest-log visibility
- `CHANGELOG.md` for architectural decision updates
- `CLAUDE.md` if workflow assumptions materially change

## Final instruction to Claude

Do not optimize for elegance or maximum automation first.

Optimize for:
1. trust
2. reversibility
3. provenance
4. explicit states
5. safe GitHub sync
6. finance-specific caution
7. making capture easy but canonical writes hard enough to trust

If you must choose between convenience and trust in v1, choose trust.
