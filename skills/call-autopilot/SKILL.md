---
name: call-autopilot
description: Automatically score new sales calls with Revenue Growth Agent (RGA) MEDDIC analysis and log the results to the CRM, from transcripts in an AI notetaker. Use when the user asks to process recent calls, catch up the CRM from their notetaker, or schedule automatic call scoring, e.g. "process my calls from today", "score every sales call and log it to HubSpot", "run call processing every 2 hours".
argument-hint: [review first]
---

# RGA call autopilot

Checks the notetaker for new calls, keeps only sales calls with prospects, runs RGA's MEDDIC analysis on each one so the deal score builds call after call, and writes a note to the CRM. RGA never writes to the CRM itself: Claude does that with the seller's own CRM connector.

## Before the first run

| Needed | For |
| --- | --- |
| RGA connector | the analysis (required) |
| Notetaker (Fireflies, Gong, Otter, Fathom, etc.) | transcripts (required) |
| CRM (HubSpot, Salesforce, etc.) | deciding what's a sales call, and logging (required) |

If any is missing, say which and stop.

Ask once whether the user wants **review mode**: show the CRM notes and wait for an OK before writing them. Default to review mode the first time this runs interactively.

## Steps

1. **Find new calls.** Ask the notetaker for calls that ended since the last run. If you can't tell when that was, use the last 24 hours.
2. **Skip what's already done.** Skip a call when the CRM already has a note for it with an RGA opportunity link (`/opportunity/`) and that call's title and date. RGA also refuses a transcript it has already analyzed (`DUPLICATE_TRANSCRIPT`, matched on content and on the notetaker's meeting id), so always pass the meeting metadata in step 5.
3. **Keep sales calls only.** Keep a call if it has attendees from outside the user's company domain and their company has a contact or an open deal in the CRM. Skip internal meetings, recruiting, vendor and personal calls. When unsure, skip it and list it under "not sure".
4. **Match each call to an RGA opportunity.** Call `rga_find_preps` with the main outside attendee's email and name: a clear match (same email, or full name plus company) gives you a `prepMeetingId` to link, and its `rootSessionId` if they already have a deal. Then call `rga_find_opportunities` with the company name, taken from the CRM record, not from the transcript.
   - Exactly one opportunity, and the people on the call fit it: add to it (`mode='existing'` with its `rootSessionId`).
   - No opportunity: start a new one (`mode='new'`) and flag it in the report.
   - More than one, or the people don't fit: don't guess. In review mode, ask. Unattended, skip the call and list it under "needs you". Adding a call to the wrong deal can't be undone.
5. **Analyze.** Never retype a transcript. If the notetaker result is in a file and you can run shell commands, call `rga_create_upload_url`, run its `curlExample` on the file, and pass `uploadId`. Otherwise use `transcriptText` only if it's under about 30,000 characters; skip longer ones and list them under "needs you". Call `rga_analyze_discovery` with the transcript, the company name, the mode and root, `prepMeetingId` if found, `meeting` (source, sourceMeetingId, title, occurredAt, attendeeEmails from the notetaker), and a `discoveryGoal`. Take the goal from the CRM deal stage or the meeting title, e.g. "Discovery call to qualify budget and timeline". Process calls oldest first, one at a time, and poll `rga_get_job_status` with `kind='discovery'` at `retryAfterMs` until it completes, so that follow-ups on the same deal stack in order.
   - `DUPLICATE_TRANSCRIPT`: already processed. Skip it and count it as "already processed". Never pass `allowDuplicate` unattended.
   - `ATTENDEE_DOMAIN_MISMATCH`: the attendees don't match the deal. Skip it and list it under "needs you". Never pass `allowDomainMismatch` unattended.
6. **Read it.** Call `rga_get_opportunity` for the scores, gaps and next questions.
7. **Log it.** Add a note to the CRM deal (or the contact if there's no deal) with:
   - call title and date
   - a 2 to 3 line summary
   - MEDDIC scores
   - top gaps
   - next questions
   - the RGA opportunity link
   In review mode, show all the notes first and wait for an OK.
8. **Report.**

## Output

```
Processed (2)
- Acme, Tue 10:00 discovery: added to existing deal, stage <stage>. <RGA link>
- Initech, Tue 13:00: NEW opportunity created, check it's right. <RGA link>

Needs you (1)
- Globex, Tue 15:00: two open RGA opportunities, which one?

Skipped: 3 internal, 1 already processed
```

## Scheduling it

To run this every 2 hours on weekdays, create a scheduled task with this prompt:

> Every 2 hours on weekdays: check my AI notetaker for calls that ended since the last run. Keep only sales calls with prospects, meaning outside attendees from a company that has a contact or open opportunity in my CRM. Skip internal meetings and any call you've already processed.
>
> For each sales call, find the matching RGA opportunity and run a MEDDIC discovery analysis on the transcript, adding it to that deal. If there's no clear match, start a new opportunity and flag it for me rather than guessing.
>
> Then add a note to the opportunity in my CRM with a short call summary, the MEDDIC scores, the top gaps and the next questions to ask. Send me a quick list of what you processed and anything you weren't sure about.

For review mode, add: "Show me the notes and wait for my OK before updating the CRM."
