---
name: daily-briefing
description: Build a morning briefing that runs a Revenue Growth Agent (RGA) meeting prep for every first-time meeting on today's calendar, using email for context, and logs it to the CRM. Use when the user asks to prep today's meetings, wants a daily or 7am briefing, or asks to schedule automatic meeting prep, e.g. "prep everyone I'm meeting today", "set up a 7am briefing".
---

# RGA daily briefing

Every weekday morning: find today's first-time meetings, prep each new person with RGA, and leave a short list of who's ready. Works because Claude also has the seller's calendar, email and CRM. RGA itself never touches those.

## Before the first run

Check which of these the user has connected in Claude:

| Needed | For |
| --- | --- |
| RGA connector | the preps (required) |
| Calendar (Google Calendar, Outlook) | today's meetings (required) |
| Email (Gmail, Outlook) | context on each person (recommended) |
| CRM (HubSpot, Salesforce, etc.) | saving the prep and spotting existing relationships (recommended) |

If a required one is missing, say which and stop. If a recommended one is missing, say what will be skipped and carry on.

## Steps

1. **List today's meetings** from the calendar. Keep meetings with at least one attendee from outside the user's own email domain. Drop internal meetings, all-day events, declined events and personal holds.
2. **Keep only first-time meetings.** Drop a person if any of these are true:
   - An earlier meeting with them is on the calendar.
   - The CRM shows prior meetings or an open deal with them.
   - `rga_find_preps` finds a prep for them (match on their email, or full name plus company).
   - The CRM contact or the calendar event already has an RGA prep link (`/api/meeting-prepper/preview`).
   Checking first is how you avoid paying for a prep twice.
3. **Gather context.** For each person left, skim recent email threads with them or their company for role, company and what the meeting is about.
4. **Enrich.** Call `rga_prepare_prospect` for each. Pass the person's LinkedIn URL if the CRM or the email signature has it: RGA uses it as an exact key.
5. **Confirm or flag.**
   - **Run interactively:** show all the confirm tables in one message and wait for one OK, as in the `meeting-prep` skill.
   - **Run as a scheduled task with nobody watching:** only run preps where `readyToRun` is true and enrichment agrees with the calendar and email (same company, a plausible title). Set `userConfirmed=true` only on those, since the seller pre-approved this exact rule when they set up the task. Everything else goes on the "couldn't confirm" list with what's missing. Never guess a missing industry or company size.
6. **Run and wait.** `rga_run_meeting_prep` for each, then poll `rga_get_job_status` with `kind='prep'` at `retryAfterMs`.
7. **Save.** For each completed prep, add or update the contact in the CRM and attach the prep link (`htmlUrl` from `rga_get_prep`), or add it to the calendar event if there's no CRM.
8. **Report.** One short message.

## Output

```
Prepped (3)
- 10:00 Jordan Lee, VP Sales, Acme: <prep link>
- ...

Couldn't confirm (1)
- 14:00 Sam Ortiz, Globex: industry unknown. Reply with it and I'll run the prep.

Skipped: 2 internal, 1 existing relationship
```

## Scheduling it

To run this every weekday at 7am, create a scheduled task in Claude with this prompt:

> Every weekday at 7am: check my calendar for today. For anyone I haven't met with before and don't already have a prep for, read my recent email with them for context, then run an RGA meeting prepper. Add or update the contact in my CRM and attach the prep. Give me a short list of who's prepped and anything you couldn't confirm.

Offer to set it up if scheduled tasks are available. If they aren't, the user can paste the prompt each morning.
