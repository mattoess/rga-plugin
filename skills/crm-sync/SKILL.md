---
name: crm-sync
description: Save Revenue Growth Agent (RGA) results to the user's CRM through their CRM connector (HubSpot, Salesforce, Microsoft Dynamics 365, Pipedrive, GoHighLevel and others), the same way every time, with no duplicates. Use after an RGA meeting prep, MEDDIC discovery analysis, proposal or SOW when the user wants it in their CRM, e.g. "add this to HubSpot", "log the prep in Salesforce", "create the contact and attach the prep", "update the opportunity with the RGA link", "save this to my CRM". Also follow these rules whenever another RGA skill writes to a CRM.
argument-hint: [prep | meddic | proposal | sow]
---

# Save RGA results to the CRM

RGA never writes to a CRM itself. Claude does it through the user's own CRM connector. This skill makes those writes predictable: the right record, the same note format, a link that keeps working, no duplicates, and nothing written without the user seeing it first.

CRM object names and quirks are in `references/crms.md`. Read the section for the user's CRM before writing.

## What gets written

| RGA result | Person record | Deal record | Note text |
| --- | --- | --- | --- |
| Meeting prep | Create or update the contact (or Salesforce lead): name, title, email, company, LinkedIn | none | "RGA meeting prep" note with the prep link |
| Discovery (MEDDIC) | Note on the contact | Note on the matching open deal/opportunity | Stage, top gaps, next questions, opportunity link |
| Proposal / SOW | none | Note on the deal | Document, status, link |

Links: use the prep's `htmlUrl` from `rga_get_prep` or `rga_find_preps`, and the opportunity's `rgaUrl` from `rga_analyze_discovery`, `rga_get_opportunity` or `rga_get_document`. Opportunity links point at the deal's first meeting and never change as meetings are added, so they're safe to store. Never store a raw storage link (anything on `blob.vercel-storage.com`).

## Steps

1. **Find the CRM connector.** Look for a connected CRM's tools (search, create, update, notes or activities). If there's none, skip to "No CRM connector" below. If there are two, ask which one is the system of record.
2. **Gather the RGA facts.** Use what's already in the conversation. If something's missing, fetch it: `rga_get_prep`, `rga_get_opportunity`, `rga_get_document`, or `rga_find_preps` for a prep link.
3. **Find before you create.** Never create a record you could have found.
   - **Person:** search by email first (exact match). If there's no email, search by full name and check the company matches. Never match on name alone.
   - **Deal:** search open deals or opportunities for that company or person. Prefer one already linked to the person.
   - One clear match: use it. Several: show them and ask. None: see step 4.
4. **Decide what to create.**
   - **Person not found:** create one only if you have at least a name plus an email or a company. Follow the CRM's rule for which object (Salesforce: a Lead unless the user says otherwise; see references).
   - **Deal not found:** don't create one unless the user asks. In Salesforce, never create an Opportunity: it needs an Account, which is outside what RGA touches. Put the note on the person instead and say so.
   - **Never** create custom fields, pipelines, stages or companies/accounts. Those are CRM admin decisions.
5. **Check for an existing note.** Search the record's notes or activities for this RGA link. If one exists, update it (or leave it, if nothing changed) instead of adding a second one. This keeps repeated runs, like the 2-hour call autopilot, from piling up duplicates.
6. **Preview and confirm.** Show the user in one message:
   - each record, with "found" or "will create"
   - the fields you'll set (only blank fields, plus fields the user just confirmed in RGA; never overwrite a non-empty CRM value without asking)
   - the note text exactly as it will appear
   Then wait for OK. Skip this only in a scheduled task the user has approved (see "Unattended runs").
7. **Write.** Person first, then deal, then notes, then associations (note to person and deal; person to deal if the CRM links them). If a field exists for an RGA link (named like "RGA Prep URL", "RGA Opportunity", "RGA Link"), also set it. Don't go looking for one beyond a quick check of the record's fields.
8. **Report.** One short list: what was created or updated, each with a link to the CRM record if the connector returns one. List anything you skipped, and why.

## Note templates

Keep notes plain text with short lines, so they read well in every CRM. Dates in the user's locale.

**Meeting prep**
```
RGA meeting prep: {First Last}, {Title} at {Company}
Prepared {date} for a meeting on {meeting date, if known}.
Open the prep: {prep link}
Logged from Revenue Growth Agent.
```

**Discovery (MEDDIC)**
```
RGA discovery analysis: {Company}, meeting {n} ({call date})
Stage: {commitment stage}
Top gaps:
- {gap 1}
- {gap 2}
Next questions:
- {question 1}
- {question 2}
Full MEDDIC and coaching plan: {opportunity link}
Logged from Revenue Growth Agent.
```

**Proposal / SOW**
```
RGA {proposal | SOW} for {Company}: {status, e.g. Draft / Approved}
{solution name, if any}
Open it in RGA: {link}
Logged from Revenue Growth Agent.
```

## Unattended runs (scheduled tasks)

When `daily-briefing` or `call-autopilot` runs on a schedule, there's nobody to confirm. Then:

- Update records you matched on email, and add or update notes. That's all pre-approved.
- Create a person only when you have their email and searching by it found nothing.
- Never create deals, never overwrite a non-empty field, never pick between several matches. Put those on the "needs you" list in the run's report.
- If the user asked for review mode, show everything and wait, as in step 6.

## No CRM connector

Say so in one line, then give the note text from the templates, ready to paste, with the links. Offer to set up the CRM connector (Customize > Connectors).

## Guardrails

- Find before create, every time. Duplicate contacts are the most common CRM complaint.
- Never write values RGA didn't produce or the user didn't confirm. Don't fill CRM fields with guesses.
- Never store transcript text in the CRM. Summaries and links only.
- If a write fails, say which record and why, and keep going with the rest. Don't retry silently.
