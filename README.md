# PRS Tracker
 
A supervisor's tool for tracking a warehouse / data-labeling team's daily PRS against target. It reads the team's performance spreadsheet, shows who is on track and who needs attention, adjusts targets for downtime, and prepares personal reminder and caution emails.
 
This README covers the **standalone web app** (`prs-tracker-standalone.html`). A separate version runs on claude.ai; see [The claude.ai version](#the-claudeai-version) at the end.
 
---
 
## Contents
 
- [At a glance](#at-a-glance)
- [Quick start](#quick-start)
- [Pages](#pages)
- [How performance is calculated](#how-performance-is-calculated)
- [Importing the spreadsheet](#importing-the-spreadsheet)
- [Downtime](#downtime)
- [Below target alerts and cautions](#below-target-alerts-and-cautions)
- [Reminders and messages](#reminders-and-messages)
- [Settings](#settings)
- [Where data is stored, and backups](#where-data-is-stored-and-backups)
- [Privacy and security](#privacy-and-security)
- [Limitations](#limitations)
- [Troubleshooting](#troubleshooting)
- [Technical notes](#technical-notes)
- [The claude.ai version](#the-claudeai-version)
---
 
## At a glance
 
| | |
|---|---|
| **What it is** | One self-contained HTML file (HTML, CSS and JavaScript). No server, no installation, no account. |
| **Who uses it** | One supervisor, on one computer. |
| **Where the data lives** | In that browser only (IndexedDB). Back it up regularly. |
| **Works offline** | Yes. The spreadsheet and chart libraries are built into the file. |
| **Browsers** | Current Chrome, Edge, Firefox or Safari on a desktop or laptop. |
| **Default target** | 408 PRS per 8-hour shift (editable). |
 
---
 
## Quick start
 
1. **Open the app.** Double-click `prs-tracker-standalone.html`, or open it from any web server. Always use the same browser on the same computer, because the data is saved there.
2. **Get the spreadsheet.** In Google Sheets, open the team sheet and choose **File → Download → Microsoft Excel (.xlsx)**. CSV and Excel files from other sources also work.
3. **Import it.** Go to **Import data** and drop the file on the upload area. Check the column choices for each tab, then click **Save and import**.
4. **Approve your team.** Go to **Associates**. People found in the sheet are listed under **New associates detected**. Fix any names, then click **Approve** or **Approve all**.
5. **Set up.** In **Settings & backup**, enter your name (used in the audit log), check the target, shift length and **team time zone**, then save.
6. **Back up.** Click **Download backup** in **Settings & backup**, and do it again regularly.
To refresh the numbers, download a fresh copy of the sheet and import it again. Your column choices are remembered.
 
---
 
## Pages
 
| Page | What it's for |
|---|---|
| **Today** | Team overview for a chosen date: counts per status, one card per associate (PRS, adjusted target, expected-by-now, progress), data freshness banner, running-downtime banner, and **Send update reminder**. |
| **Below target** | Supervisor alerts for anyone below expected performance, with **Review data**, **Dismiss** and **Send caution**, plus the full caution history. |
| **Trends** | Charts over 7 / 30 / 90 days for the team average or one associate (PRS, SPL, ATOT, Quality). |
| **Downtime** | Start and end downtime with buttons, a live timer, approval, corrections and history. |
| **Messages** | Write one message from a template and open a personal email for each selected person. |
| **Associates** | Approve new associates; edit names, emails, shift, start time, work days and leave; remove people. |
| **Import data** | Upload the spreadsheet and review the column choices for each tab. |
| **Sheet data** | Every tab exactly as it was read, with row and column counts, for checking that nothing was missed. |
| **Settings & backup** | General settings, reminders, caution message, data sources, backup and restore, and the audit log. |
 
---
 
## How performance is calculated
 
**Adjusted target**
 
```
adjusted target = round( target × (shift hours − approved downtime hours) ÷ shift hours )
```
 
Example: 408 × (8 − 1) ÷ 8 = **357** for one hour of approved downtime.
 
**Expected PRS (pace)**
 
```
expected = adjusted target × (hours since shift start ÷ shift hours)
```
 
The expected value is 0 before the shift starts and stops rising at the full adjusted target. A shift that runs past midnight counts for the date it **started**. "Today" and all times use the team time zone set in Settings.
 
**Status**
 
| Status | Meaning |
|---|---|
| On track | PRS is at or above expected. |
| Slightly behind | Below expected, but by no more than the tolerance (default 10%). |
| Well behind | Further below expected than the tolerance. |
| Not updated | A scheduled work day with no PRS recorded. |
| Not started | The shift hasn't started yet. |
| Day off | Not one of the person's work days. |
| On leave | A leave date for that person. |
 
**Missed day.** A past, scheduled work day that is not a leave day and has no PRS. Days off and leave never count as missed.
 
The meanings of PRS, SPL, ATOT and Quality come from your company's systems. The app only reads and compares the numbers.
 
---
 
## Importing the spreadsheet
 
The app reads **every tab, every row and every column**, then decides which tabs hold daily performance data. Nothing is ever silently dropped, and bad cells produce warnings rather than stopping the import.
 
### Automatic column detection
 
For each tab, the app:
 
1. Finds the header row: the first row (within the first 15) that is mostly text.
2. Samples up to 300 values per column. These count as empty: blank cells, `-`, `n/a`, `#N/A`, `#DIV/0!`, `#VALUE!`, `#REF!`, `TRUE`/`FALSE`.
3. Matches columns by header name **and** by the kind of values in them:
| Field | Picked when |
|---|---|
| Associate | At least 60% of values are emails (headers like Labeler, Email, Actual Email, Login), or an ID column. |
| Date | Header contains "date", or is Period / Day / Week, **and** at least 50% of values are dates. |
| PRS | Numbers, header contains "PRS" (fallback: Quantity, Units, Output). |
| SPL | Numbers, header contains "SPL". |
| ATOT | Numbers, header contains "ATOT". |
| Quality | Numbers, header "Total Accuracy" (fallback: Accuracy, Quality). |
| Shift | Header "Shift". |
 
Headers containing *expected, target, goal, rating, hours, days, absent* or *month* are never treated as metrics.
 
4. Uses a tab only if it has an associate column, a date column and at least one metric. Every skipped tab shows the reason.
5. Turns off by default any tab whose name contains *wrong, conflict, backup, old, copy, test* or *archive*. You can switch these back on.
The review screen lets you change any choice with dropdowns. Your choices are saved per tab and reused next time. If a tab's columns change, you're asked to check it again rather than the app guessing.
 
**Also supported:** "dates across the top" layouts (one column per day), and sheets without a header row.
 
### Dates and numbers
 
Dates are accepted as real date cells, Excel serial numbers, `2026-05-15`, `15/05/2026` or `05/15/2026` (choose day-first or month-first in Settings), `May 15, 2026`, `Sep 9, 2026, 10:23`, `May 15-2026-2:33:00 PM` and `15-May-2026`. Times are ignored and dates are stored as `YYYY-MM-DD`.
 
Numbers can include commas, spaces and `%`. If ATOT or Quality is stored as a fraction between 0 and 1, it is converted to a percentage (0.95 → 95).
 
### Where each number comes from
 
Each metric uses **one** source tab. Tab order never decides. Defaults:
 
| Metric | Default source |
|---|---|
| PRS | A daily tab named like "ATOT…" (then "PRS…") |
| ATOT | A daily tab named like "ATOT…" |
| SPL | A daily tab named like "SPL…" |
| Quality | A daily tab named like "Quality…" or "Accuracy…" |
 
Weekly tabs (whose date column is called "Week") are never picked automatically. You can choose any source, or "Don't use", under **Settings & backup → Data**.
 
When one person has several rows for the same day in the source tab, they are combined per metric: PRS is **added up**, and SPL, ATOT and Quality are **averaged**. Each rule can be changed to Add up, Average, Highest or Last row.
 
Each import **adds and updates** days. Days that are missing from the new file are kept.
 
### New associates
 
People found in the sheet who aren't on the team yet are listed for approval, with a name suggested from their email (`john.doe@company.com` → John Doe). Until you approve them, their numbers are kept but they don't appear on dashboards or in reminders. You can also **Ignore** someone, and move them back later.
 
---
 
## Downtime
 
Downtime is recorded with buttons, so you never type or calculate times.
 
1. Choose a **reason** (required): Meeting, Training, System Outage, Equipment Issue, Safety Briefing or Other.
2. Choose **Whole team** or **Selected associates**, and optionally add notes.
3. Click **Start downtime**. The page shows **DOWNTIME IN PROGRESS** with the start time and a live timer. A banner also appears on Today.
4. Click **End downtime** and confirm. The app records the end time and calculates the exact duration (for example "35 minutes 27 seconds").
**Rules**
 
- Only one downtime can run at a time.
- Refreshing or closing the browser does **not** end downtime. When you come back, the timer continues from the stored start time.
- Each event is numbered (#1001, #1002…) and stores the date, start, end, duration, reason, scope, notes, who started it, who ended it, and its approval status.
- With approval on (the default), finished downtime waits under **Waiting for approval**. Only approved downtime lowers targets. You can turn approval off in Settings.
- **Correct** lets you fix times, reason or notes. A reason for the change is required, the original values are kept in the history, and an approved downtime needs approving again.
- **Delete** needs a reason. The record is kept, marked as deleted, and stops counting.
- Times come from the web server's clock when the file is served from a website. When opened directly from disk, they come from the computer's clock, and each record notes which clock was used.
---
 
## Below target alerts and cautions
 
Being below target **never** sends anything to an associate automatically. It only creates an alert for you.
 
```
Performance data → below expected? → alert for the supervisor
                                        ├── Dismiss (nothing is sent)
                                        └── Review data → Send caution → edit → confirm → email app
```
 
- **Review data** shows that day's PRS, expected, target, difference, imported numbers (PRS, SPL, ATOT, Quality and any extras), logged downtime, the last 14 days, and earlier cautions.
- **Dismiss** asks for an optional reason and is recorded in the audit log. Dismissed alerts can be reopened.
- **Send caution** opens an editor with the recipient, the channel, and a subject and message pre-filled from your template. Everything can be edited. **Confirm & open in email app** opens the email ready to send. Cancel does nothing.
- If a caution was already sent for that person and day, you must tick **"Yes, send another caution"** first, and the second one gets its own record.
- **Caution history** keeps each caution's time, channel, exact message, supervisor, status, and the **PRS, expected and target at that moment**. These are never recalculated later.
- Because the app can't see whether you pressed send in your email program, cautions are recorded as **"Opened in email app"**.
### Caution template fields
 
Edit the default caution under **Settings & backup → Caution message**. These fields are filled in automatically:
 
`{{associate_name}}` `{{first_name}}` `{{current_prs}}` `{{expected_prs}}` `{{target}}` `{{current_pct}}` `{{expected_pct}}` `{{difference}}` `{{date}}`
 
Cautions are completely separate from PRS update reminders.
 
---
 
## Reminders and messages
 
**Send update reminder** (Today page) includes only people who are scheduled to work, not on leave, whose shift has started, and who are **not updated** or **behind pace**. It opens a list where each **Open email** button creates one personal email in your email app:
 
```
Subject: Please update your PRS - Mon, Oct 5
 
Hi Jo,
 
Please update your PRS for Mon, Oct 5.
 
Your current performance:
  PRS so far: 320
  Target: 408
  Remaining: 88
 
Please update your PRS when ready.
 
Thank you.
```
 
If downtime affected the target, the email says so. **Copy all email addresses** puts every address on the clipboard if you'd rather send one group email.
 
**Scheduled reminders.** In Settings you can set reminders a number of minutes before each person's shift ends (default 30), plus extra fixed times. At those times, **while the app is open**, the same email list pops up for the people who need it.
 
**Messages** lets you pick a template (Behind pace, Great job, Missed update…), use `{name}`, `{prs}`, `{target}` and `{date}`, select recipients, and open a personal email for each.
 
---
 
## Settings
 
| Section | Settings |
|---|---|
| **General** | PRS target per shift (408), shift length (8 h), slightly-behind tolerance (10%), your name, team time zone, date format (month-first / day-first). |
| **Reminders** | Minutes before shift end (30, or 0 for off), extra reminder times, downtime approval on/off. |
| **Caution message** | Default subject and message, with the fields listed above. |
| **Data** | Source tab for each metric, and how several rows per person per day are combined. Changes apply at the next import. |
| **Your data** | Download backup, restore from backup, delete all data. |
| **Audit log** | Every important action: imports, approvals, settings changes (old → new), downtime start/end/approval/corrections/deletions, alerts dismissed, cautions, reminders and backups. |
 
Each associate's shift start time, work days and leave dates are set on the **Associates** page.
 
---
 
## Where data is stored, and backups
 
All data is saved in the browser's **IndexedDB** storage on this computer (with `localStorage` as a fallback). Nothing is sent anywhere.
 
This means:
 
- Clearing browsing data, using a private/incognito window, or switching browser or computer gives you an **empty** tracker.
- **Download a backup regularly** (Settings & backup → Download backup). It produces a file called `prs-tracker-backup-YYYY-MM-DD.json`.
- **Restore from backup** replaces everything in the current browser with the backup's contents. This is also how you move the tracker to another computer.
- **Delete all data** wipes the tracker in this browser (you must type DELETE to confirm).
A full copy of the spreadsheet is kept for the Sheet data page only if it is smaller than about 3 MB. Larger sheets can be viewed right after uploading them.
 
---
 
## Privacy and security
 
- The app and its backups contain **personal performance data and work emails**. Treat backup files as confidential and share them only in ways your company allows.
- There is **no login**. Anyone who can use the computer and browser can see the data. Use a password-protected computer account.
- Check with your IT or data-protection team before using company data in the app.
- The app never uploads data. When it's open from disk, the only outside request is for display fonts, and it works without them.
---
 
## Limitations
 
- One supervisor, one computer. There is no shared data between people and no associate logins or associate dashboards.
- Emails are opened in your email program one at a time. Nothing is sent automatically, and delivery can't be confirmed.
- Scheduled reminders only appear while the app is open.
- The Google Sheet is imported from a downloaded file, not read directly from its link. Reading it directly would require making the sheet public.
- Google Chat isn't supported.
For a shared, multi-user version with Google sign-in for associates, the app would need a backend (for example Firebase). See the V3 build prompt.
 
---
 
## Troubleshooting
 
| Problem | What to do |
|---|---|
| Dates look wrong (e.g. May 10 instead of 5 Oct) | Settings → change the date format to day-first or month-first, then import again. |
| A number is missing or wrong | Import data → check that tab's column choices. Settings → Data → check the source tab for that metric. |
| A tab was skipped | Read the reason shown on the review screen, then switch the tab on with the "This tab" menu and set its columns. |
| Someone isn't on Today | They may still be waiting under Associates → New associates detected, or be Ignored. |
| Downtime didn't change the target | Approve it under Downtime → Waiting for approval (or turn approval off in Settings). |
| "Data may be out of date" | The last import is over 24 hours old. Upload a fresh copy of the sheet. |
| Email buttons do nothing | Set a default email program on your computer (or in your browser's settings for mailto links). |
| "Storage is full" | Download a backup, remove people who left, or free browser storage. |
| Everything disappeared | You're probably in a different browser, a private window, or the browser data was cleared. Use **Restore from backup**. |
| "Today" is the wrong day | Settings → set the team time zone (e.g. Africa/Nairobi). |
 
---
 
## Technical notes
 
- **Single file:** HTML, CSS and JavaScript with no build step needed to run it.
- **Built-in libraries:** [SheetJS](https://sheetjs.com) 0.18.5 (Apache-2.0) for reading .xlsx and .csv files, and [Chart.js](https://www.chartjs.org) 4.4.1 (MIT) for charts. Display fonts (Barlow) load from Google Fonts when online and fall back to system fonts offline.
- **Storage:** IndexedDB database `prs-tracker-standalone`, object store `kv`. The fallback uses `localStorage` keys prefixed `prs:`.
| Key | Contents |
|---|---|
| `sup/team` | Associates and their schedules, leave and approval status |
| `sup/settings` | All settings |
| `recs/<associate>` | Daily numbers per associate |
| `sup/downtime` | Downtime events, including history of corrections |
| `sup/mapping` | Saved column choices, source candidates, chosen sources |
| `sup/rawmeta`, `raw/…` | Info about the last import, and the stored sheet copy (small sheets only) |
| `sup/alerts` | Dismissed below-target alerts |
| `sup/cautions` | Caution history with snapshot values |
| `sup/msglog` | Messages |
| `sup/remlog` | Which scheduled reminders already fired today |
| `sup/audit` | Audit log (last 300 entries) |
| `sup/backupinfo` | Time of the last backup |
 
- **Backup format:** JSON `{ "app": "PRS Tracker", "version": 1, "exportedAt": "…", "data": { <key>: <value>, … } }`.
- **Testing:** the app was tested end to end in a simulated browser with a real 25-tab, ~30,000-row workbook: import, approval, Today, reminders, cautions, downtime (including refresh during running downtime), trends, sheet view, reopen, and backup/restore.
---
 
## The claude.ai version
 
The same tracker also runs as a claude.ai artifact. It adds features that need claude.ai's sign-in and storage:
 
- Shared data for all supervisors (people with edit access).
- Associate dashboards: each associate signs in with a claude.ai account in your organization and sees only their own numbers, messages and missed days.
- Reading the Google Sheet directly through the Google Drive connector, with automatic sync while open.
- Reminders and cautions sent directly through the supervisor's Gmail, with delivery confirmed.
Both versions use the same calculations, import rules, downtime and caution workflows.
 
