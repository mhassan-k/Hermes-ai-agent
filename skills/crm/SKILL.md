---
name: crm
description: >-
  Lightweight CRM for a solo founder. Use when the user wants to track leads,
  contacts, companies, and deals; log interactions; schedule and surface
  follow-ups; move deals through a sales pipeline; or get a snapshot of their
  funnel. Works off a single plain-text/CSV store so it needs no external
  service.
---

# CRM for a Solo Startup

A solo founder doesn't need Salesforce — they need to never drop a lead and
always know who to follow up with today. This skill keeps a simple, durable
record of people, companies, and deals, and turns it into a daily action list.

## Data model

Everything lives in `data/` as CSV so it's greppable, diffable, and editable by
hand. Create these files on first use if they don't exist (templates are in
`templates/`):

- `data/contacts.csv` — one row per person.
  `id,name,company,role,email,phone,source,owner,created,notes`
- `data/companies.csv` — one row per organization.
  `id,name,website,industry,size,notes`
- `data/deals.csv` — one row per opportunity.
  `id,title,contact_id,company_id,stage,value,currency,opened,expected_close,status,notes`
- `data/interactions.csv` — append-only activity log.
  `timestamp,contact_id,deal_id,type,summary`
- `data/followups.csv` — open tasks.
  `due_date,contact_id,deal_id,action,done`

IDs are short slugs (e.g. `acme`, `jane-doe`) so they read well in references.
Dates are ISO `YYYY-MM-DD`. Never rewrite history in `interactions.csv` — only
append.

## Pipeline stages

Use this default funnel unless the user defines their own. Keep it short — a
solo founder can't run a 9-stage process.

1. `lead` — identified, not yet contacted
2. `contacted` — reached out, awaiting reply
3. `qualified` — confirmed fit, budget, and need
4. `proposal` — quote/proposal sent
5. `won` — closed-won (set deal `status=closed`)
6. `lost` — closed-lost (set deal `status=closed`, record reason in notes)

## Core workflows

### Add a lead
1. Add/confirm the company row in `companies.csv`.
2. Add the contact in `contacts.csv` with `source` (where they came from).
3. Open a deal in `deals.csv` at stage `lead` with an estimated `value`.
4. Append an interaction describing how they came in.
5. Create a follow-up so the lead doesn't go cold.

### Log an interaction
Append one row to `interactions.csv` (`type` = call/email/meeting/note). If it
implies a next step, close the relevant follow-up and create the next one. If it
changes deal momentum, update the deal `stage`.

### Move a deal
Update `stage` in `deals.csv`, append an interaction noting the change, and set
the next follow-up. On `won`/`lost`, set `status=closed` and clear open
follow-ups for that deal.

### Daily briefing
When asked "what should I do today" (or similar), produce:
- **Overdue & due-today follow-ups** from `followups.csv` where `done` is empty
  and `due_date <= today`, sorted by date.
- **Stale deals**: open deals whose most recent interaction is older than 7 days.
- **Pipeline snapshot**: count and total `value` of open deals per stage.
Present it as a short, prioritized list — actions first, summary last.

## Operating principles

- **Every lead gets a next step.** Never leave a contact or open deal without a
  follow-up; if one is missing, propose one.
- **Append, don't overwrite** the interaction log; correct mistakes in `notes`.
- **Confirm before bulk edits or deletes.** This is the user's business memory.
- **Keep it light.** Suggest only fields and stages the user will actually
  maintain; default to fewer.
- **Surface, don't nag.** Lead with what's actionable today, then the snapshot.
