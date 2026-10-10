---
name: discovery-analysis
description: Run Revenue Growth Agent (RGA) MEDDIC discovery analysis on a sales call transcript. Use when the user shares or points to a call transcript (pasted, attached, or in a notetaker like Fireflies, Gong or Otter) and asks to analyze it, score it, run MEDDIC, find the gaps, or add it to a deal, e.g. "here's my transcript from the Acme call, run a MEDDIC analysis".
argument-hint: <company> (transcript pasted, attached or in your notetaker)
---

# RGA discovery analysis (MEDDIC)

Scores a discovery call against MEDDIC with evidence, finds the gaps, and writes the questions for the next meeting. Follow-up calls stack on the same opportunity so the score reflects everything heard so far.

The one mistake that can't be undone is adding a transcript to the wrong deal. Evidence is appended and there is no way to remove it. Every step below exists to prevent that.

## Steps

1. **Get the transcript. Never retype it.**
   - Pasted by the user: use the text.
   - In a notetaker the user has connected: fetch it with that connector. Large results are usually saved to a file; use the file.
   - Attached file: use the file.
   - Also collect what the notetaker gives you: meeting title, date and time, attendee emails, the notetaker's meeting id.
2. **Check the speaker labels.** If someone introduces themselves under another name ("I'm Dan" said by "Jake Marmer"), or there are only generic labels ("Speaker 1/2/3"), propose a speaker map (`{"Jake Marmer": "Dan Shinefield"}`) and confirm it with the user. Wrong labels can make RGA name the wrong person as economic buyer or champion.
3. **Confirm the company.** Ask for, or confirm, the prospect's company name. Don't take it from the transcript alone: transcripts often name the product being discussed rather than the account.
4. **Find the person's prep.** Call `rga_find_preps` with the prospect's name or email and the company.
   - One match on email, or on full name plus company: confirm in one line ("Linking this to your prep for Dan Shinefield at Quantum Technology. OK?") and use its `prepMeetingId`.
   - Several possible matches: show them and let the user pick, or choose none.
   - No match: carry on without one. Don't run a prep first just to have one; a prep for a meeting that already happened isn't worth the cost. Mention they can run one later.
   - If the match has a `rootSessionId`, that person already has an opportunity. Use it in the next step.
5. **Decide new deal or existing deal.**
   - If the user named a deal, or said it's new, use that.
   - Otherwise call `rga_find_opportunities` with the company name. If it returns opportunities (or the prep has one), show them and ask: "Is this call part of one of these, or a new opportunity?" Wait for the answer.
   - If the people or company in the transcript don't match the opportunity the user picked, stop and say so. Don't reconcile it yourself.
6. **Get the discovery goal.** `discoveryGoal` is required. Ask the seller in one line what they wanted to learn or achieve on the call, unless they already said.
7. **Upload and start.**
   - **Transcript in a file, and you can run shell commands:** call `rga_create_upload_url`, then run its `curlExample` with the real file path in place of `transcript.txt`. Keep `--data-binary` (`-d` strips the newlines). It should print `"status": "complete"`. Then call `rga_analyze_discovery` with `uploadId`.
   - **Pasted text, or no shell:** use `transcriptText`, but only if it's under about 30,000 characters. For anything longer, stop and tell the user: either allow `www.revenuegrowthagent.com` in Claude's network settings (Settings > Capabilities) so the file can be uploaded, or upload the transcript in the RGA app.
   - Either way, pass: companyName, discoveryGoal, `prepMeetingId` (if found), `meeting` (source, sourceMeetingId, title, occurredAt, attendeeEmails), `speakerMap` (if confirmed), prospect details, and the mode: `mode='existing'` with `rootSessionId` for an existing deal, `mode='new'` otherwise.
8. **Handle what comes back.**
   - `needsChoice`: show the candidates and ask the question in step 5. Never pick for the user.
   - `DUPLICATE_TRANSCRIPT`: this call was already analyzed. Tell the user which meeting it matches (company and date from the error details). Only pass `allowDuplicate=true` if they explicitly ask to re-run it.
   - `ATTENDEE_DOMAIN_MISMATCH`: the call's attendees don't look like this deal's company. Show the domains and ask the user. Only pass `allowDomainMismatch=true` if they confirm.
   - `UPLOAD_EXPIRED` or `UPLOAD_ALREADY_USED`: get a new upload URL and upload again.
   - `warnings`: pass them on to the user.
9. **Set expectations.** Tell the user how long it will take using `estimatedSeconds` (usually a few minutes; a follow-up re-analyzes the whole deal, so longer). Share the `rgaUrl`, which works straight away and is safe to save to a CRM.
10. **Wait.** Poll `rga_get_job_status` with `kind='discovery'` at `retryAfterMs`.
11. **Deliver.** When `completed`, call `rga_get_opportunity` with the session id.

## Several calls for one deal

Submit them oldest first. The first uses `mode='new'` (or `existing` if the deal already exists). Each later one uses `mode='existing'` with the `rootSessionId` from the first. Wait for each job to reach `completed` before submitting the next.

## Output

Lead with what the seller acts on, not the full readout:

- The RGA link, and the prep it's linked to if any
- Commitment stage
- The 2 or 3 biggest MEDDIC gaps, each with one line on why it matters
- The questions to ask in the next meeting

Then offer: "Want the full MEDDIC breakdown?" If yes, show `rga_get_opportunity`'s markdown as returned.

## Guardrails

- Never chain to an existing opportunity on your own inference.
- Never retype a long transcript into a tool call. Upload the file.
- If the job `failed`, report the error and stop. Don't re-submit without asking.
