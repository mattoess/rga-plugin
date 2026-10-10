---
name: proposal-sow
description: Draft a proposal, or a statement of work (SOW), in Revenue Growth Agent (RGA) from a deal's discovery analysis. Use when the user asks to write, draft, create or generate a proposal or SOW for an opportunity, or to see one that exists, e.g. "create a proposal for Acme, $15,000 a month for three months built around a $1M ROI", "generate the SOW for Globex".
argument-hint: <company> [pricing, timeline, ROI]
---

# RGA proposal and SOW

Builds a client-facing proposal from what the buyer actually said in discovery, with relevant case studies. Once a person approves the proposal in RGA, the SOW follows. Both are client-facing; the SOW is contractual.

## Find the opportunity (both documents)

1. Call `rga_find_opportunities` with the company name. If there's more than one, show the table and ask which. If there's none, explain that a proposal needs a discovery analysis first and offer the `discovery-analysis` skill.
2. Call `rga_get_opportunity` to confirm it has been analyzed (`analyzedMeetings` above 0, `analysisPending` false). If not, say so and stop.

## Proposal

1. **Ask which solution.** Ask the seller which of their solutions to propose, by its name in RGA. Never pick one for them; there is no tool that lists solutions, so don't guess from the transcript.
2. **Capture the terms.** Pricing, term length, ROI target, scope notes and tone go in `instructions`, in the seller's own words. If they gave none, ask once whether there's anything to build around, then proceed.
3. **Start.** Call `rga_generate_proposal` with the session id, the solution and the instructions. Tell the user it usually takes under 2 minutes.
4. **Wait.** Poll `rga_get_job_status` with `kind='proposal'` at `retryAfterMs`.
5. **Show it.** Call `rga_get_document` with `kind='proposal'` and show the markdown as returned, with the RGA link.
6. **Hand off.** Tell the seller to review it and, when happy, approve it in RGA. Approval can't be done from here, and the SOW needs it.

If a CRM connector is available, offer to log the proposal on the deal, following the `crm-sync` skill.

The seller can keep refining in conversation. To regenerate, call `rga_generate_proposal` again with updated instructions.

## SOW

1. **Check approval.** Call `rga_get_document` with `kind='proposal'` and read `proposalStatus`. If it isn't approved, tell the seller to approve the proposal in RGA first, give the link, and stop.
2. **Start.** Call `rga_generate_sow` with the session id. Pass `sowInstructions` for anything the seller wants (milestones, start date, payment terms) and `templateName` only if they named one.
   - `PROPOSAL_NOT_APPROVED` means the same thing as step 1: send them to RGA to approve it.
3. **Wait.** Poll `rga_get_job_status` with `kind='sow'` at `retryAfterMs`.
4. **Show it.** Call `rga_get_document` with `kind='sow'`, show it as returned with the link, and remind the seller to review it in RGA before it goes to the client.

## Guardrails

- There is no approve tool, by design. Never tell the user a proposal is approved unless `proposalStatus` says so.
- Don't invent prices, dates or ROI figures. Use only what the seller gives you.
- Show documents as returned. Don't rewrite or summarize them unless asked.
