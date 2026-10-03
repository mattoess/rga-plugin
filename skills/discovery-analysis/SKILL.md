---
name: discovery-analysis
description: Run Revenue Growth Agent (RGA) MEDDIC discovery analysis on a sales call transcript. Use when the user shares or points to a call transcript (pasted, attached, or in a notetaker like Fireflies, Gong or Otter) and asks to analyze it, score it, run MEDDIC, find the gaps, or add it to a deal, e.g. "here's my transcript from the Acme call, run a MEDDIC analysis".
argument-hint: <company> (transcript pasted or attached)
---

# RGA discovery analysis (MEDDIC)

Scores a discovery call against MEDDIC with evidence, finds the gaps, and writes the questions for the next meeting. Follow-up calls stack on the same opportunity so the score reflects everything heard so far.

The one mistake that can't be undone is adding a transcript to the wrong deal. Evidence is appended and there is no way to remove it. Every step below exists to prevent that.

## Steps

1. **Get the transcript.** If the user pasted or attached it, use that. If it lives in a notetaker the user has connected, fetch it from there. If you can't reach it, ask the user to paste it. It must be the full transcript, not a summary.
2. **Confirm the company.** Ask for, or confirm, the prospect's company name. Don't take it from the transcript alone: transcripts often name the product being discussed rather than the account.
3. **Decide new deal or existing deal.**
   - If the user named a deal, or said it's new, use that.
   - Otherwise call `rga_find_opportunities` with the company name. If it returns opportunities, show the table and ask: "Is this call part of one of these, or a new opportunity?" Wait for the answer.
   - If the people or company in the transcript don't match the opportunity the user picked, stop and say so. Don't reconcile it yourself.
4. **Get the discovery goal.** `discoveryGoal` is required. Ask the seller in one line what they wanted to learn or achieve on the call, unless they already said.
5. **Start.** Call `rga_analyze_discovery` with transcriptText, companyName, discoveryGoal and, for an existing deal, `mode='existing'` plus its `rootSessionId`. For a new deal use `mode='new'`. Add prospect details if you have them.
   - If it returns `needsChoice`, show the candidates and ask the question in step 3. Never pick for the user.
6. **Set expectations.** Tell the user how long it will take using `estimatedSeconds` (usually a few minutes; a follow-up re-analyzes the whole deal, so longer). Share the `rgaUrl`, which works straight away and is safe to save to a CRM.
7. **Wait.** Poll `rga_get_job_status` with `kind='discovery'` at `retryAfterMs`.
8. **Deliver.** When `completed`, call `rga_get_opportunity` with the session id.

## Output

Lead with what the seller acts on, not the full readout:

- The RGA link
- Commitment stage
- The 2 or 3 biggest MEDDIC gaps, each with one line on why it matters
- The questions to ask in the next meeting

Then offer: "Want the full MEDDIC breakdown?" If yes, show `rga_get_opportunity`'s markdown as returned.

## Guardrails

- Never chain to an existing opportunity on your own inference.
- Uploading the same transcript twice creates two meetings on the deal. If the user may have analyzed this call before, ask first.
- If the job `failed`, report the error and stop. Don't re-submit the transcript without asking, since a retry can double-count.
