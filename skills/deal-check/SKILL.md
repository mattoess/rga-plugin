---
name: deal-check
description: Check where a deal stands in Revenue Growth Agent (RGA), read only. Use when the user asks about the status, health, stage, MEDDIC gaps or open questions on an opportunity at a company, e.g. "where do we stand with Acme?", "what are the open gaps on the Globex deal?", "quick deal check before my pipeline review".
argument-hint: <company>
---

# RGA deal check

A fast, read-only answer on one opportunity, short enough to read on a phone between meetings. This skill never starts an analysis or changes anything.

## Steps

1. **Find the deal.** Call `rga_find_opportunities` with the company name (and the prospect's name if the user gave one).
   - None found: say so and offer to run a discovery analysis if they have a transcript.
   - More than one: show the table and ask which one.
2. **Read it.** Call `rga_get_opportunity` with that opportunity's session id.
3. **Answer briefly.** If `analysisPending` is true, say an analysis is still running and give the link.

## Output

Keep it to a few lines:

- **Stage:** commitment stage
- **Meetings analyzed:** analyzed of total
- **Biggest gaps:** 2 or 3 MEDDIC gaps, one line each
- **Still open:** the top questions for the next conversation
- **Link:** the RGA opportunity link

Offer the full MEDDIC breakdown only if they ask for more detail.

## Guardrails

- Read only. Don't call any tool that starts a job.
- Don't add opinions or a forecast the analysis doesn't support. Report what RGA has.
