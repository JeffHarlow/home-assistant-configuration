---
name: file-bill
description: Process a newly scanned BILL or DEADLINE document (tax bill, invoice, insurance premium, renewal notice, anything with an amount or a due date) — read it, give it a clear name, file it in the right Google Drive folder, and put a high-priority, multi-reminder entry on Google Calendar for every due date. Use when the user runs /file-bill or attaches a scan and asks to file it and remind them to pay/act. Not for general filing of documents that have no due date or action.
---

# /file-bill — file a bill and make sure it never gets missed

The user (Jeff) runs this when new documents come off the scanner that require
payment or action by a date. Do every step below in order. Work from the
document itself; never guess amounts, dates or account numbers.

Input: one or more attached PDFs/images (`$ARGUMENTS`, or files the user
attached with `@`). If nothing is attached, ask the user to attach the scan and stop.

## 1. Read the document — every page

Use the Read tool with `pages` (e.g. `pages: "1-10"`) so you actually see each
page image. A scan often holds several bills (one per account or parcel). For
each bill, record:

- Issuer (e.g. Otero County Treasurer), document type, tax/policy year
- Owner / payer name (e.g. Double H Trust) and the property or item it covers
- Account / policy / parcel numbers
- Amounts: per installment and total, per account, and the combined totals
- Due dates **and** delinquency/penalty dates, plus the penalty terms
- How to pay: website, phone, mailing address, "make check payable to"

Add the combined totals yourself and double-check the arithmetic.

If the document has **no amount due and no deadline**, it is not a bill:
say so, and offer to just name and file it (steps 2–4) without calendar entries.

## 2. Name the file

Pattern: `<Year> <Issuer> <Document type> - <Owner> <Account number(s)>.pdf`

Example: `2026 Otero County Property Tax Bills - Double H Trust R032195 R032196.pdf`

Avoid commas and parentheses (they upload awkwardly). Keep it under ~100 characters.

## 3. Pick the Google Drive folder

Search before creating anything (`search_files`, folders only,
`mimeType = 'application/vnd.google-apps.folder'`), by issuer, topic and owner.

Known folders (verify they still exist before using them):

| Topic | Folder | Folder ID |
|---|---|---|
| Property taxes (Otero County) | Harlow Home › Property Taxes | `112njCuims7BFrcRy9wrd--Cr6O4OXThB` |
| Home records in general | Harlow Home | `1agIiEGf8r0WFgHLhJgGsyJPfaUx5c_hX` |
| Double H Trust legal papers | Documents › Long-Term Care – Jeff & Jill › Legal Documents (Double H Trust, 2021) | `1y3Iu9DY5Scatx31SNWwe7WQuqlmi56XK` |

Ignore the old 2019 `OTERO` folder (password-manager data) and `House-Land Papers`
(2010–2011 farm sale) — they are not for current bills.

If no folder fits, create a topic subfolder under **Harlow Home** (e.g.
`Insurance`, `Utilities`, `Vehicle Registration`) and tell the user. Ask before
creating anything at the top level of My Drive.

## 4. Get the file into Drive

The Drive connector cannot reliably upload a scanned PDF (it would have to send
the whole file as base64 text in one request). So:

1. Copy the original scan to the scratchpad under the new name (`cp`, keep full quality).
2. Send it with `SendUserFile` using `display: "attach"` and a caption telling the
   user to drag it into the folder (give the folder link).
3. **Do not** show preview images or crops of the scan in the chat — the user may
   save a preview by mistake instead of the PDF (this happened once).
4. When the user says it is uploaded, list the folder and confirm: the file is a
   PDF, its size matches the original byte-for-byte, and `read_file_content`
   shows every bill/account. If a wrong file (e.g. `download.png`) landed there,
   point it out and leave removal to the user unless they ask you to.

## 5. Calendar — one high-priority entry per due date

Use Google Calendar `create_event` on the primary calendar. First run
`search_events` for the issuer/account to avoid duplicates. Skip due dates that
are already past (tell the user instead — they may already be delinquent).

For **each** due date (e.g. 1st half and 2nd half are two entries):

- **Title:** `🔴 HIGH PRIORITY: Pay <Issuer short> <what> – <installment> $<combined amount>`
- **Time:** 9:00–9:30 am on the due date, user's own time zone (leave `timeZone` unset)
- `colorId: "11"` (red), `availability: AVAILABILITY_FREE`, `visibility: private`
- **Location:** where to pay in person, if there is one
- **Reminders** (`overrideReminders`, Google allows at most 5):
  - email 10080 (7 days before)
  - popup 4320 (3 days before)
  - email 1440 and popup 1440 (1 day before)
  - popup 60 (1 hour before)
- **Description (HTML):** amount per account and the combined total; the pay-in-full
  option if there is one; every way to pay; a **mail-by date about 5 days before
  the due date** if paying by check; the penalty date and terms; links to the
  Drive folder (and, after step 4, to the file).

Check the weekday of each due date. If it falls on a Saturday or Sunday, say so
in the description and in your summary (offices are closed; pay online or by
phone, or earlier in the week).

After the file is confirmed in Drive (step 4), update **each** event with
`update_event`: `addedAttachments` = the Drive file link, refresh the description
with the direct file link, and `notificationLevel: NONE`. Do not resend reminders
(fields you leave out stay unchanged).

Google Tasks (the checkbox to-do items) cannot be created with these tools — the
calendar entries are the "task". Say so if the user asks for a task.

## 6. Report back

A short summary for the user:

- A table of the bills: account, item, each installment, total, and the combined totals
- Where the file is (folder path plus link) and whether it is confirmed in Drive yet
- Each calendar entry: date, weekday, amount, and the reminder schedule
- Anything that needs the user's attention: mail-by dates, weekend due dates,
  penalty dates, the option to pay the full year at once, past-due items

Plain language: the user is a retired engineer and CFO; explain what you did,
not the plumbing.
