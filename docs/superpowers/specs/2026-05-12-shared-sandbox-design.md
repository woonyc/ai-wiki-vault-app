# v5.2 — Shared Sandbox (Layer 0)

**Status:** approved 2026-05-12
**Source brief:** Hermes shared-sandbox diagnosis (external)
**Replaces:** v5.1 local-only Temp Inbox as the *primary* capture surface (legacy local items still rendered).
**Deployment context:** Hermes runs on a single always-on machine (PC or Mac mini). Both browsers + Hermes share state via the existing GitHub repo backing `vault.json`.

## Problem

In v5.1, captures land in `state.tempItems[]` inside `vault.json`. That blob only sync on a manual full-vault push. So a capture on PC is invisible to Mac (and to Hermes) until the user remembers to ↻ Sync. The LLM/cron processor can't know "what was captured today" — it has to infer from sync timing, which is fragile.

This conflates two concerns in one storage seam:
- **Intake coordination state** — short-lived, mutates often, multiple writers, must propagate fast.
- **Canonical knowledge state** — pages + sources, mutates carefully, one effective writer at a time.

## Goal

Capture lands in a **shared sandbox** first, visible to all devices and to Hermes immediately on push. Canonical vault sync stays separate and remains a curated, manual operation.

## Non-goals

- Realtime websocket / SSE. Polling only.
- Auto-Hermes wiring. We document the contract; user runs Hermes manually.
- Per-item encryption. PAT scope already restricts to the repo.
- Cross-repo sandbox. Single repo for now.
- Backend service. Browser → GitHub Contents API direct, same as `vault.json` today.

## Glossary additions

| Term | Meaning |
|------|---------|
| **SharedInbox** | New module in the HTML app that owns intake-coordination state. Backed by GitHub `inbox/` folder. Disjoint from `Vault`. |
| **Capture** | A user action that produces one inbox item (URL, multi-URL paste, text). Now writes to SharedInbox, not `Vault`. |
| **Capture item** | One JSON file under `inbox/{state}/cap-{ts}-{rand}.json`. |
| **State folder** | One of `captured`, `claimed`, `processed`, `discarded`. Folder name = item status (no mutable status field). |
| **Atomic move** | Two-step `PUT` new path + `DELETE` old path. The DELETE step is the commit point — if DELETE fails (409) the move is abandoned. |
| **Pending push** | A capture written to local cache that hasn't yet succeeded against GitHub. Surfaced with badge. |

## Architecture

### New module: `SharedInbox`

Single seam over intake coordination. Two adapters (real seam per LANGUAGE.md):
- `GitHubInboxAdapter` — primary. Talks to GitHub Contents API.
- `LocalInboxCache` — fallback for offline + queue for `pending_push` items.

**Public interface:**
```js
SharedInbox.list(state = 'captured')             // -> [{capture_id, sha, ...meta}]  (cache + remote merged)
SharedInbox.capture(payload)                     // -> item   (writes cache, queues push)
SharedInbox.claim(captureId)                     // -> { ok | conflict }
SharedInbox.process(captureId, { draft_refs })   // -> { ok }
SharedInbox.discard(captureId, reason)           // -> { ok }
SharedInbox.releaseStale()                       // -> n  (sweep stale claims back to captured/)
SharedInbox.refresh()                            // -> { added, removed, total }
SharedInbox.flushPendingPush()                   // -> { pushed, failed }
SharedInbox.status()                             // -> { connected, last_refresh, pending_push, ... }
```

### Folder layout in the GitHub repo

```
inbox/
  captured/   cap-2026-05-12T071500-x7k2.json
  claimed/    cap-...json     (carries claimed_by + claimed_at)
  processed/  cap-...json     (carries processed_at + draft_refs)
  discarded/  cap-...json     (carries discard_reason)
```

State transitions = atomic move (PUT new + DELETE old). Folder name is authoritative.

### Capture item schema

```json
{
  "capture_id": "cap-2026-05-12T071500-x7k2",
  "schema_version": 1,
  "device_id": "pc-woon-2026-04",
  "captured_at": "2026-05-12T07:15:00Z",
  "input_type": "url|text|mixed",
  "raw_input": "...",
  "source_url": "https://example.com/...",
  "source_hint": "finance-trading|null",
  "user_note": "...",
  "input_hash": "djb2 hash of raw_input — for client-side dedupe",
  "claimed_by": null,
  "claimed_at": null,
  "processed_at": null,
  "draft_refs": [],
  "discard_reason": null
}
```

### Capture flow (browser)

1. User submits Capture modal.
2. `SharedInbox.capture(payload)`:
   - Build item with `device_id` + `captured_at` + `input_hash`.
   - Write to `LocalInboxCache` with `local_status: 'pending_push'`. UI renders immediately.
   - Background `GitHubInboxAdapter.put(captureId, item)`. On 201 → `local_status: 'pushed'` + cache the returned `sha`. On failure → leave pending, retry on next ↻ Inbox or ↻ Sync.
3. `vault.json` is NOT touched.

### Polling — manual only

- No auto-poll. No interval.
- Topbar gets **↻ Inbox** button next to ⊕ Capture. Inbox view header also has ↻ + last-fetch timestamp.
- Click → `GitHubInboxAdapter.list('captured')` with `If-None-Match` header (last ETag). 304 → no body, no cost. 200 → diff against `last_seen_ids` → render new + toast `"+N from {device}"`.
- API cost: 0/hour idle. Each manual click ≤ 1 GET (304 = free quota).

### Capacity

| Volume | Behavior |
|--------|----------|
| ~50 captures/day | trivial |
| ~500/day | listing API paginates at 1000/page, still one call |
| ~5000/day | works, recommend periodic move to `inbox/archive/YYYY-MM/` for snappier listing |

Each file ~0.5–2 KB. 1000 captures ≈ 1 MB total. Repo size negligible.

### Claim semantics (lock)

`SharedInbox.claim(captureId)`:
1. GET `inbox/captured/{id}.json` (capture sha).
2. PUT `inbox/claimed/{id}.json` with patched `claimed_by` (this device) + `claimed_at` (now).
3. DELETE `inbox/captured/{id}.json` with sha. **This is the commit point.**
4. If DELETE returns 409 (sha stale — file changed) or 404 (file already gone) → another device beat us. Best-effort: DELETE the `inbox/claimed/{id}.json` we just wrote (could lose the race here too — accept that worst-case both folders briefly hold the item; next refresh resolves). Return `{ conflict: true }`.

Stale claim recovery: on `SharedInbox.refresh()`, also list `claimed/`. Any item with `claimed_at` older than 10 minutes is movable back to `captured/` by any device. Same atomic move pattern. Worst case: two workers process the same item — last-writer-wins on `processed/` move, vault gets duplicate page that lint catches.

### Process flow

1. User in Inbox view clicks "Prepare" on a captured item.
2. `SharedInbox.claim(id)`. On conflict → toast, refetch list.
3. Run existing `IngestPipeline.normalize` against the inbox item's `raw_input` to produce v5.1 `IngestDraft`(s).
4. User reviews + commits via existing v5.1 Review view.
5. After successful commit: `SharedInbox.process(id, { draft_refs: [pageId, ...] })` moves the file to `processed/`.

### Discard flow

1. User clicks "Discard" on a captured or claimed item.
2. `SharedInbox.discard(id, reason)` moves to `discarded/` with `discard_reason`.
3. Local cache entry removed.

### UI surfaces

- **Inbox view** renders union of:
  - Local pending-push items (badge: `pending push`)
  - Remote `inbox/captured/` items (badge: `device_id`, own = rust, other = lime)
  - Optional toggle: show `claimed/` items (read-only, with `claimed_by` shown)
  - Optional toggle: show `processed/` and `discarded/` for audit
- **v5.1 local-only `state.tempItems`** rendered separately at bottom of Inbox view, marked "v5.1 local-only", with a "Push to shared inbox" button per item.
- **Topbar**: `⊕ Capture` (unchanged behavior, now writes to SharedInbox), `↻ Inbox` (new, manual refresh).
- **Tree**: Inbox count badge now reflects union (local pending + remote captured).
- **Statusbar**: small `⊞ N` indicator showing pending-push count.

### Storage seam

```
SharedInbox
├─ GitHubInboxAdapter      (primary: GitHub Contents API)
└─ LocalInboxCache         (offline cache + pending-push queue, in localStorage)
```

LocalStorage key: `vault-v4-inbox-cache` (own key, separate from `vault-v4`).

Cache shape:
```js
{
  schema_version: 1,
  // device_id NOT stored here — reuses Vault.state.device_id (set in v5.1 schema bump)
  last_etag: "...",
  last_refresh: "2026-05-12T07:15:00Z",
  items: { [captureId]: { ...item, local_status, sha } },
  pending_push: [captureId, ...]
}
```

**Persist scope:** `SharedInbox` writes its own localStorage key on every mutation. It does NOT call `Vault.persist()`. Vault and SharedInbox are independent persist surfaces — that's the whole point of the separate seam.

### Vault state additions (schemaVersion 3 → 4)

`Vault.state.inbox = { last_etag, last_refresh, last_seen_ids: [] }` — minimal pointer for cross-session continuity. The actual cache lives in its own localStorage key (separate sync surface). Vault stays clean.

### Failure modes

| Failure | Behavior |
|---------|----------|
| GitHub down on capture | Item queued in `pending_push`, badge shown, retry on focus/sync |
| GitHub down on poll | Toast `"inbox offline (cached)"`, last-known list rendered |
| Claim 409 (race) | Cancel claim, refetch, surface conflict toast |
| Stale claim (>10min) | Auto-recoverable on next refresh |
| LocalStorage corrupt | Re-fetch from GitHub on next boot |
| GitHub PAT missing or wrong scope | Capture falls back to v5.1 local-only `tempItems` with warning toast |
| Capture before GitHub configured | Same fallback — v5.1 local-only with banner |

### Migration v3 → v4

- Add `state.inbox = { last_etag: null, last_refresh: null, last_seen_ids: [] }` to vault state.
- Existing `state.tempItems[]` not auto-pushed (could be stale junk). Surfaced in Inbox view with explicit "Push to shared inbox" action.
- `vault.json` schema otherwise unchanged.

### Hermes contract

Documented in new top-level file `HERMES_PROTOCOL.md`. Summary:

```
Hermes lives on one always-on machine. It has the GitHub repo cloned and a PAT.

Loop:
1. git pull
2. ls inbox/captured/
3. For each candidate (chunk 25 max per cycle):
     a. Read the file.
     b. git mv to inbox/claimed/, write claimed_by + claimed_at, commit, push.
     c. If push 409 → git pull + retry; if still claimed by other → skip.
     d. Process: parse raw_input, run extraction, edit vault.json (concept stubs / atoms / etc).
     e. git mv claimed file → inbox/processed/, write processed_at + draft_refs.
     f. Commit, push.
4. Sweep: any inbox/claimed/ file with claimed_at > 10min ago → git mv back to inbox/captured/.
5. Sleep N seconds, repeat.

Hermes MUST stamp claimed_by with its identifier (e.g. "hermes-mac-mini-2026") so browser sees a non-device claim.
```

## Out of scope (defer to v5.3+)

- Auto-Hermes integration in the HTML app.
- Realtime websocket / SSE.
- Per-item encryption.
- Cross-repo sandbox.
- Bulk-archive UI for `processed/` and `discarded/`.
- Conflict-resolution UI for the rare claim race.

## Open questions

None at spec time. (User confirmed Hermes deployment topology = one always-on machine, storage shape = per-item files, polling = manual only.)
