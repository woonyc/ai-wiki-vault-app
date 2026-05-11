# Hermes Protocol — AI Wiki Vault Shared Inbox

**Version:** 1 (matches v5.2 SharedInbox `schema_version: 1`)
**Spec:** `docs/superpowers/specs/2026-05-12-shared-sandbox-design.md`
**Audience:** the Hermes agent (and any future cron / live-artifact processor) consuming the Shared Inbox.

## What Hermes is

A single always-on processor (Mac mini / home server / wherever Woon parks it) that:
- watches the Shared Inbox folder in the GitHub repo,
- picks up captures from any device,
- runs LLM extraction / classification / draft generation,
- writes results into the canonical vault,
- moves the inbox file to the appropriate state folder.

Hermes is the only non-browser writer. Browsers (PC, Mac) are also writers for **capture** and **claim from a UI**, but Hermes is the workhorse for unattended processing.

## Topology

```
        ┌──────────────┐         ┌──────────────┐
        │  PC browser  │         │ Mac browser  │
        │  vault HTML  │         │  vault HTML  │
        └──────┬───────┘         └──────┬───────┘
               │                        │
               │ GitHub Contents API    │
               │                        │
               ▼                        ▼
        ┌──────────────────────────────────────┐
        │       GitHub repo (one)              │
        │  vault.json   — canonical knowledge  │
        │  inbox/                              │
        │    captured/  cap-*.json             │
        │    claimed/   cap-*.json             │
        │    processed/ cap-*.json             │
        │    discarded/ cap-*.json             │
        └──────┬───────────────────────────────┘
               │ git pull / push (PAT-authed)
               ▼
        ┌──────────────────┐
        │     HERMES       │   always-on processor
        │  (Mac mini, etc) │   reads inbox/, edits vault.json, moves files
        └──────────────────┘
```

## Capture file schema

`inbox/{state}/cap-{tsCompact}-{rand}.json`

```json
{
  "capture_id": "cap-20260512T071500-x7k2",
  "schema_version": 1,
  "device_id": "pc-abcdef12",
  "captured_at": "2026-05-12T07:15:00.000Z",
  "input_type": "url" | "text" | "mixed",
  "raw_input": "<verbatim user paste>",
  "source_url": "https://example.com/article",
  "source_hint": "finance-trading" | "ai-research" | null,
  "user_note": "free text",
  "input_hash": "djb2 hash of raw_input — for client-side dedupe",
  "claimed_by": null | "<device_id>",
  "claimed_at": null | "ISO timestamp",
  "processed_at": null | "ISO timestamp",
  "draft_refs": [] | ["<vault page id>", ...],
  "discard_reason": null | "free text"
}
```

Folder name = authoritative status. `claimed_by` / `claimed_at` / `processed_at` / `discard_reason` are journaling fields, not state machines.

## State transitions

```
captured ── claim ──▶ claimed ── process ──▶ processed
                  └─ discard ──▶ discarded
captured ── discard ─────────────────────▶ discarded
claimed  ── release-stale (>10 min) ────▶ captured
```

Every transition = **atomic move**: PUT new file at new path, DELETE old file (with sha). The DELETE is the commit point. If DELETE returns 409 (sha stale) or 404 (already moved) → another writer beat you. Roll back the PUT and retry.

## Hermes loop (recommended)

```python
# pseudocode — adapt to your runtime
import time, json, subprocess, requests

HERMES_ID = "hermes-mac-mini-2026"
REPO = "owner/ai-wiki-vault"
BRANCH = "main"
INBOX = "inbox"
PAT = os.environ["GH_PAT"]
H = {"Authorization": f"Bearer {PAT}", "Accept": "application/vnd.github+json"}

def gh(method, path, body=None, sha=None):
    url = f"https://api.github.com/repos/{REPO}/contents/{path}"
    payload = body.copy() if body else {}
    payload["branch"] = BRANCH
    if sha: payload["sha"] = sha
    return requests.request(method, url, headers=H, json=payload)

def loop():
    while True:
        listing = requests.get(
            f"https://api.github.com/repos/{REPO}/contents/{INBOX}/captured?ref={BRANCH}&per_page=100",
            headers=H
        ).json()
        items = [f for f in (listing or []) if f["type"] == "file" and f["name"].startswith("cap-")]
        for f in items[:25]:           # chunk per cycle
            cap_id = f["name"][:-5]    # strip .json
            try:
                process_one(cap_id)
            except Exception as e:
                log_error(cap_id, e)

        release_stale_claims()
        time.sleep(60)

def process_one(cap_id):
    # 1. fetch + claim (atomic move captured/ -> claimed/)
    g = requests.get(f"https://api.github.com/repos/{REPO}/contents/{INBOX}/captured/{cap_id}.json?ref={BRANCH}", headers=H).json()
    sha = g["sha"]
    item = json.loads(base64.b64decode(g["content"]))
    item["claimed_by"] = HERMES_ID
    item["claimed_at"] = iso_now()
    put = gh("PUT", f"{INBOX}/claimed/{cap_id}.json", {
        "message": f"hermes claim {cap_id}",
        "content": base64.b64encode(json.dumps(item, indent=2).encode()).decode()
    })
    if put.status_code in (409, 422):
        return  # someone else claimed it
    delete = gh("DELETE", f"{INBOX}/captured/{cap_id}.json", {"message": f"hermes claim move"}, sha=sha)
    if delete.status_code in (404, 409):
        # race lost — roll back our claim
        try:
            put_sha = put.json()["content"]["sha"]
            gh("DELETE", f"{INBOX}/claimed/{cap_id}.json", {"message":"hermes rollback"}, sha=put_sha)
        except: pass
        return

    # 2. process: parse raw_input, run LLM extraction, write to vault.json
    page_ids = extract_and_commit_to_vault(item)   # YOUR LLM PIPELINE

    # 3. move claimed/ -> processed/ (or discarded/ on failure)
    item["processed_at"] = iso_now()
    item["draft_refs"] = page_ids
    claim_g = requests.get(f"https://api.github.com/repos/{REPO}/contents/{INBOX}/claimed/{cap_id}.json?ref={BRANCH}", headers=H).json()
    gh("PUT", f"{INBOX}/processed/{cap_id}.json", {
        "message": f"hermes processed {cap_id}",
        "content": base64.b64encode(json.dumps(item, indent=2).encode()).decode()
    })
    gh("DELETE", f"{INBOX}/claimed/{cap_id}.json", {"message":"hermes processed move"}, sha=claim_g["sha"])

def release_stale_claims():
    # Anyone may move claimed/ items older than 10 minutes back to captured/.
    listing = requests.get(f"https://api.github.com/repos/{REPO}/contents/{INBOX}/claimed?ref={BRANCH}", headers=H).json()
    for f in listing or []:
        g = requests.get(f["url"], headers=H).json()
        item = json.loads(base64.b64decode(g["content"]))
        age_ms = (now_ms() - parse_iso(item.get("claimed_at") or item["captured_at"]))
        if age_ms < 10 * 60 * 1000: continue
        item["claimed_by"] = None
        item["claimed_at"] = None
        gh("PUT", f"{INBOX}/captured/{f['name']}", {
            "message": f"release stale claim {f['name']}",
            "content": base64.b64encode(json.dumps(item, indent=2).encode()).decode()
        })
        gh("DELETE", f"{INBOX}/claimed/{f['name']}", {"message":"release stale move"}, sha=g["sha"])
```

### Notes

- **Chunk per cycle** (25 items max) keeps API usage polite. GitHub authenticated rate limit is 5000 / hour.
- **Stale claim sweep** runs once per cycle. If your cycle is 60 s, a stuck claim is recoverable in ~11 minutes worst case.
- **Vault writes** go through `vault.json` exactly the same way the browser writes them (read+SHA → modify → PUT with sha; on 409 → re-pull + merge + retry). Reuse Woon's existing v4/v5 sync logic if you fork code from the HTML app.
- **Identity**: stamp `claimed_by` with a stable Hermes identifier (e.g. `hermes-mac-mini-2026`). Browsers recognize non-`pc-*` / non-`mac-*` device IDs and render them as "claimed by Hermes" in UI.

## What Hermes should NOT do

- **Do not auto-promote finance content** to durable vault truth. Finance captures arrive with `source_hint: "finance-trading"` (or are detectable by ticker / keyword regex). Default action = write a `dated_source_only` page with `review_required: true`. The user reviews on next browser visit.
- **Do not delete** items from `processed/` or `discarded/`. Those folders are the audit trail. Run a separate monthly archive job if cleanup matters.
- **Do not skip the claim step.** Even if Hermes is the only processor today, the claim ensures a future second worker (e.g. mobile capture, separate browser) won't double-process.
- **Do not write `state.tempItems[]` / `state.ingestDrafts[]`** in `vault.json`. Those are browser-side coordination state for the Inbox/Review views — Hermes' equivalent is the `inbox/` folder.

## Browser-Hermes coordination contract

| Browser does | Hermes does |
|--------------|-------------|
| `⊕ Capture` → PUT `inbox/captured/cap-X.json` | nothing (browser writes; Hermes will pick up next cycle) |
| `↻ Refresh` → list `inbox/captured/` + `inbox/claimed/` | nothing |
| `Claim + Prepare` → atomic move to `claimed/` (UI session takes over) | nothing — Hermes sees `claimed_by` is a `pc-*` / `mac-*` device, skips |
| `Commit` → write vault page + atomic move to `processed/` | nothing |
| `Discard` → atomic move to `discarded/` | nothing |
| nothing | runs loop above; processes anything `captured/` has had sitting > capture wait time |

A user can always claim a captured item ahead of Hermes — Hermes will skip it on next cycle. If a user claim is abandoned (browser tab closed mid-flow), the stale-claim sweep returns it to `captured/` after 10 min.

## Future v5.3+ extensions

- Per-domain Hermes routing (finance Hermes vs research Hermes).
- Reviewer agent layer (Brief §Model routing) — Hermes drafts, separate stronger model approves.
- Webhooks instead of polling (GitHub repo dispatch → Hermes push notification).
- Per-item encryption for sensitive captures.
