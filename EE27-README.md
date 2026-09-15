# Endurance Exchange 2027 — 4-Stage Event System

Four self-contained HTML pages that share one live Supabase database.

**Live site:** https://enduranceexchange2027.com/

| Page | Live URL |
|---|---|
| Participant Dashboard (share this one) | https://enduranceexchange2027.com/ee27-3-participant-dashboard.html |
| Session Intake (admin) | https://enduranceexchange2027.com/ee27-1-session-intake.html |
| Schedule Builder (admin) | https://enduranceexchange2027.com/ee27-2-schedule-builder.html |
| Tracking Dashboard (admin) | https://enduranceexchange2027.com/ee27-4-tracking-dashboard.html |

The site root redirects to the Participant Dashboard, so the short link to give
attendees is simply: **enduranceexchange2027.com**. The old
justintrolle.github.io/EE27-Event-System/ links automatically redirect here.
DNS lives at GoDaddy (A records on @ to GitHub Pages IPs, CNAME on www).

**To publish changes:** edit the HTML files in this folder, then
`git add -A && git commit -m "update" && git push` — GitHub Pages redeploys in ~1 minute.

| Stage | File | Who uses it | Access |
|---|---|---|---|
| 1. Session Intake | `ee27-1-session-intake.html` | You (admin) | Passcode |
| 2. Schedule Builder | `ee27-2-schedule-builder.html` | You + planning team | Passcode |
| 3. Participant Dashboard | `ee27-3-participant-dashboard.html` | Attendees (public) | Open — registers on first visit |
| 4. Tracking Dashboard | `ee27-4-tracking-dashboard.html` | You (admin) | Passcode |

**Admin passcode:** `EE27-CaribeRoyale` — change it any time in Supabase with:
`update ee27_config set admin_pass = 'NewPasscode' where id = 'main';`

## How the data flows

```
Google Sheet (presenter submissions)
        │  Stage 1: download CSV → drop into intake page → import
        ▼
ee27_sessions (Supabase)  ←— Stage 2: assign day + slot in the builder (live-synced)
        │
        ▼
Stage 3: participants see only slotted sessions, register, pick sessions,
         check in on site, and rate content/presenter afterward
        │
        ▼
Stage 4: real-time rollup — planned vs checked-in vs scores, feedback wall,
         follow-up outreach list, attendee list, CSV exports (auto-refresh 60s)
```

## Stage 1 — Session Intake
- Source sheet: [EE27 presenter submissions](https://docs.google.com/spreadsheets/d/1gFTyB-rV0xvlsaoJ_6WKOGuEXrmEArtinOt9_KXpriw/edit)
- The sheet is private, so the flow is **File → Download → CSV**, then drop the file on
  the intake page (or paste the CSV text).
- The Current Database table shows every session with editable Track and Format
  dropdowns, a presenter-folder column (add/edit/open links in place), and click-to-expand
  summaries and key takeaways.
- Re-importing is always safe: rows are matched to existing sessions (by a stored sheet
  signature, then title, then speaker+title similarity). Matched rows only update
  contact info/bio; your curated titles, summaries, day themes, and slot assignments are
  never overwritten. Unmatched rows are added to the "Needs a deliberate home" pool.
- The database was seeded on 2026-08-25 with all 77 sheet submissions merged with the
  curated session content and the 30 slot assignments from the previous builder.

## Stage 2 — Schedule Builder
- Same planning grid as before (3 days, 14 numbered slots × rooms a–d, K1–K6 keynotes),
  but sessions now live in the database instead of being hard-coded.
- Each card has a **DAY** selector (move sessions between day pools, incl. TBD) and the
  **SLOT** number/letter selectors. Changes save instantly and sync to everyone.
- Print (landscape letter) and per-session PDF downloads still work.
- Presenter names still link to their Google Drive folders where available.

## Stage 3 — Participant Dashboard
- Attendee registers once per device (name, email, coach/RD/other) — no password.
- Day tabs show only sessions that have a slot; open slots and unscheduled sessions are
  hidden from participants automatically.
- Per session: **Add to my schedule** (interest signal), **Check in** (attendance),
  **Give feedback** (1–5 stars for content and presenter, comments, follow-up request).
- "My Schedule" tab shows their picks in chronological order.

### Conference-wide post-event survey
- The participant dashboard has an **EVENT SURVEY** tab that stays locked 🔒 until
  **Sunday, January 10, 2027 at 3:30 PM ET** (right after the final sessions end), then
  opens automatically. Change the open time with:
  `update ee27_config set survey_opens_at = '2027-01-10T15:30:00-05:00' where id = 'main';`
- Questions mirror the 2026 post-event survey (roles, value, experience, recommend,
  reasons, satisfaction matrix, open feedback, format/feature/month preferences, future
  attendance), updated for the EE27 program (Friday/Saturday Happy Hours, Sunday Fun
  Run 5K, Morning Group Workouts).
- To preview the form before it opens, add `?preview=1` to the participant dashboard URL.
- Attendees can revise and resubmit; each attendee holds one response.
- Results appear on the tracking dashboard's **EVENT SURVEY** tab: response counts with
  bar charts per question, a satisfaction matrix with 1–5 averages (Did Not Attend and
  N/A excluded), all written comments, and a full CSV export.

## Stage 4 — Tracking Dashboard
- KPI tiles: attendees, selections, check-ins, feedback responses, follow-up requests.
- Session rollup: planned (by role split) vs checked-in vs average scores, sortable;
  click a row to jump to that session's feedback.
- Feedback wall, follow-up outreach list, attendee roster — each with CSV export.

## Backend (Supabase project `vzwpdsnxstazpfukdbak`)
- Tables: `ee27_sessions`, `ee27_contacts` (presenter email/phone — admin-only),
  `ee27_attendees`, `ee27_selections`, `ee27_attendance`, `ee27_feedback`, `ee27_config`.
- Attendee emails, presenter contacts, and feedback comments are **not** publicly
  readable — they only leave the database through passcode-checked functions
  (`ee27_admin_data`, `ee27_import_sessions`, `ee27_save_session`, `ee27_submit_feedback`).
- The old `ee2027_schedule` table (used by the previous single-file builder) is left in
  place but is no longer read or written by these pages.
