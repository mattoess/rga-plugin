---
name: meeting-prep
description: Run a Revenue Growth Agent (RGA) meeting prep on a prospect, or on a list of leads such as trade show contacts. Use when the user asks to prep, research or get ready for a meeting or call with a named person at a company, e.g. "run a meeting prep on Jordan Lee at Acme", "prep me for my call with the CFO of Globex", "here are five leads from the show, prep each one".
argument-hint: <first last> at <company>, or a list
---

# RGA meeting prep

Turns a name and a company into a full RGA meeting prep: pains, talking points, discovery questions, company and competitive context. A prep costs real money and runs on exactly the values passed, so the seller confirms the inputs before anything runs.

## Steps

1. **Collect who.** For each prospect you need first name, last name and the company name spelled correctly. If any is missing or the company is ambiguous, ask. If the user gave a spreadsheet or list, read every row before starting.
2. **Check for an existing prep.** If the user might have prepped this person before, call `rga_find_preps`. If one exists, offer it (with its link) before spending money on another.
3. **Enrich.** Call `rga_prepare_prospect` with firstName, lastName, companyName, plus email, title and **LinkedIn URL** whenever the user, the spreadsheet or the CRM has them. A LinkedIn URL is used as an exact key, so a typo in the company name can't hide the person. For a list, enrich every prospect before showing anything.
   - If a prospect comes back with a "No LinkedIn match" warning, show it as written: it says which company spellings were tried. Ask for the LinkedIn URL or a corrected company name, then re-run that one prospect.
   - Show any email warnings (shared inbox, personal address, email domain that doesn't match the website). They usually mean a CRM typo.
4. **Confirm.** Render each result's `confirmTable` as a markdown table: field, value, source. Point out anything in `stillMissing` and any `warnings`. Then stop and wait for the user to approve or correct.
   - For a list, show all the tables in one message and ask for one approval. Number the prospects so the user can say "fix #3's title" without retyping the rest.
   - When the user corrects a value, call `rga_prepare_prospect` again with the corrected fields and `runEnrichment=false`.
5. **Run.** Only after approval, call `rga_run_meeting_prep` with the confirmed values and `userConfirmed=true`. For a list, start every prep, then poll them all.
6. **Wait.** Tell the user a prep usually takes under 2 minutes. Poll `rga_get_job_status` with `kind='prep'` at the `retryAfterMs` it returns, never faster.
7. **Deliver.** When status is `completed`, call `rga_get_prep` with the `prepMeetingId` from the job result. Show the content as returned, headings and bullets intact, and give the `htmlUrl` link. For a list, give a short summary line per prospect with its link, then offer to show any prep in full.

## Guardrails

- Never fill a missing field by web searching, inferring or guessing without showing the user. If enrichment left industry or company size empty, ask. A wrong industry quietly degrades the research and nothing flags it.
- Industry and company size only accept the values in the tool's schema. Pick the industry by what its label means, not by which slug sounds right: a sign or print shop is `packaging-printing`, while `marketing` means marketing agencies and marketing software. If none fits, say so and ask; don't approximate.
- Don't summarize or critique the finished prep. The user asked for the prep.
- If a job comes back `failed`, say which prospect failed and why (from the job's `error`), and offer to retry that one. Don't silently retry.
- On an error, read `error.code`. `MCP_ORGID_MISSING` means the connection has no RGA workspace: suggest the `setup` skill.

## Output

One prospect: the prep in full, then the link.
A list: a table of prospect, company, status and link, then "Want me to open any of these?"

If a CRM connector is available, offer once: "Want me to add this to {CRM}?" If yes, follow the `crm-sync` skill.
