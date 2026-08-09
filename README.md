# Bant Labs LinkedIn Outreach Agent

## Why this is split across cloud + local
GitHub Actions has no LinkedIn login. Claude in Chrome runs inside *your*
browser using *your* live session. So anything that touches LinkedIn
(sending requests, reading notifications/replies) has to run on your
machine through Claude in Chrome — it cannot be scheduled unattended in the
cloud without storing your session credentials somewhere, which is a real
account-ban risk and this repo deliberately avoids it.

What CAN run unattended in the cloud: reading your company list, finding
POC candidates via public search, drafting notes, and preparing the day's
review batch. That's Mode A below.

## Daily routine

**Automatic (GitHub Actions, ~10:00 AM IST):**
Runs Mode A — researches new companies from `data/company-list.xlsx`,
finds POCs, drafts connection notes, writes `data/daily-batch-<date>.csv`.
Nothing gets sent. Check the Actions tab or your repo's commit history to
see the day's batch land.

**Manual, whenever convenient (your machine, Claude in Chrome open):**
1. Open `data/daily-batch-<date>.csv` in Excel/Sheets, review each drafted
   note, set `Approved` to `yes` for the ones you want sent (leave blank to
   skip). This is your review step.
2. In Claude Code, say: *"Run Mode B — send today's approved batch."*
   The `connection-sender` subagent sends only the approved rows, paced
   45–90s apart, capped at 20/day.
3. Also run: *"Run Mode B — check LinkedIn activity."* to pull the last 24h
   of accepted connections and replies via `linkedin-monitor`.
4. Then: *"Run Mode C — draft today's digest."* — `email-summarizer` drafts
   the digest via Gmail. It will show you the draft and wait for you to say
   "send it" before it actually sends.

You can do steps 2–4 anytime that suits you — there's no reason they need
to be exactly 10 AM; only the *research* half is on a fixed schedule.

## One-time setup
1. Add your `ANTHROPIC_API_KEY` as a GitHub repo secret (Settings → Secrets
   and variables → Actions → New repository secret).
2. Replace `data/company-list-template.xlsx` with your real list — same two
   columns: Company Name, Website.
3. Connect Claude in Chrome locally and confirm you're logged into LinkedIn
   before running Mode B for the first time.
4. Confirm Gmail is connected for the digest draft/send step.
5. Trigger the workflow manually once from the Actions tab
   ("Run workflow") to confirm it runs end-to-end before trusting the
   10 AM schedule.

## Safety guardrails already built in (don't remove these)
- 20 connection requests/day hard cap
- 45–90s randomized pacing between sends
- Every send (LinkedIn or email) requires your explicit approval — nothing
  in this repo sends autonomously, by design
- `poc-finder` marks unknowns as unknown rather than guessing a name
