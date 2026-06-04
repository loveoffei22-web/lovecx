# EKEDP CX Reporting Tool — Project Context

## Live Site
**Live URL:** https://lovecx.love-offei22.workers.dev/index.html
**Repo:** loveoffei22-web/lovecx
**Deploy branch:** main (deployed via Cloudflare Workers)
**Working branch:** claude/practical-thompson-LTEad

## App Structure
| File | Purpose |
|------|---------|
| `index.html` | Landing / home page |
| `agents hub.html` | Agent-facing app — log tickets, My Tickets, Slot Reports, Cloud Sync |
| `managers hub.html` | Manager-facing app — All Complaints, roster, bulk ops, settings |
| `Daily Email Report.html` | Email report generator — loads data from Agents Hub |
| `success criteria.html` | Success criteria reference |
| `baseline.js` | Shared baseline data |

## Agents
| ID | Name | Role |
|----|------|------|
| matthew | Matthew Akhigbe | Team Lead |
| racheal | Racheal Ogunleke | CX Agent |
| tomina | Tomina Egbokwu | CX Agent |
| esther | Esther Okafor | CX Agent |
| loveth | Loveth Okpara | CX Agent |

## Shift Schedule (WAT)
| Day | Duty Window |
|-----|------------|
| Monday | Sun 4:01 PM → Mon 5:00 PM |
| Tue – Fri | Prev day 5:01 PM → today 5:00 PM |
| Saturday | Sat 10:00 AM → Sat 4:00 PM |
| Sunday | Sat 4:01 PM → Sun 4:00 PM |

Tickets logged after 5:01 PM on weekdays count as the **next day's** tickets.

## Shared Data Layer
- All data lives in **localStorage** key `ek_v2_tickets` (shared across all pages on same domain)
- Shifts stored in `ek_v2_shifts`
- Support guide stored in `ek_guide`
- Per-agent legacy backup: `ek_tix_<agentId>`
- Both hubs sync via `storage` event listener + 15s polling fallback
- `loggedAt` field = when agent entered the ticket (used for duty window filtering)
- `created` field = actual complaint date (may be a past date from paste)

## Key Bugs Fixed (June 2026)
1. **My Tickets empty** — `inDutyWindow` now checks `loggedAt` (log time) not `created` (complaint date). Old tickets backfilled via ID timestamp extraction in `loadTix`.
2. **Dedup crash** — `dedupTickets` was accessing `result[idx]` beyond array bounds when same ticket number appeared twice in incoming batch. Fixed with bounds check.
3. **Delete Selected no-op** — bulk delete was looping `onDelete(id)` with stale React state. Fixed with single `onDeleteBulkAllTix(ids)` that filters all IDs in one pass.
4. **Duty window wrong** — rewrote `getDutyWindow` in both hubs to match actual shift schedule.
5. **Sync gaps** — `ticketSig` mismatch fixed; Managers Hub gained 15s polling; Agents Hub now reloads `tickets` state on `syncTick` so manager edits appear in My Tickets immediately.

## Deploy Workflow
```
# Make fixes → commit → push to feature branch
git add <files>
git commit -m "description"
git push -u origin claude/practical-thompson-LTEad

# When ready to go live — merge to main
git checkout main
git merge claude/practical-thompson-LTEad
git push origin main
# Netlify auto-deploys within ~1 minute
```
