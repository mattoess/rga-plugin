# CRM reference for crm-sync

How each CRM models people, deals and notes, and what to watch for. Connector tool names differ between connectors and versions, so match on what a tool does (search contacts, create note), not on its exact name. If a connector can't do something listed here (for example it has no notes tool), use the fallback and tell the user.

## HubSpot

- **Person:** Contact. Email is the unique key; search by `email` first. Properties: `firstname`, `lastname`, `email`, `jobtitle`, `company`, `hs_linkedin_url` (if present in the portal).
- **RGA app properties:** portals with the RGA HubSpot app have `rga_prep_doc_url`, `rga_last_prep_date` and `rga_last_prep_session` (group "Revenue Growth Agent"). See "RGA fields" in SKILL.md. Write the note as well.
- **Deal:** Deal, in a pipeline with a `dealstage`. Find open deals associated with the contact, or with the contact's company.
- **Note:** a Note engagement (`hs_note_body`, `hs_timestamp`), associated to the contact and, for MEDDIC/proposal, the deal. Note bodies accept basic HTML; plain text with line breaks is fine.
- **Don't create:** companies, pipelines, stages, properties. HubSpot may auto-create a company from the email domain; that's HubSpot's behavior, not something to do yourself.
- **Fallback:** if there's no notes tool, create a completed Task with the note text in the body.

## Salesforce

- **Person:** Lead or Contact. RGA works with people only (Lead and Contact), never Accounts.
  - Search both Leads and Contacts by email. If a Contact exists, use it; otherwise an unconverted Lead.
  - To create, make a **Lead** (`FirstName`, `LastName`, `Company` (required), `Email`, `Title`). Don't create a Contact: it normally needs an Account.
- **Deal:** Opportunity. Use one only if it already exists (search by the person's Account via the Contact, or by name). **Never create an Opportunity or an Account.**
- **Note:** a completed Task is the most reliably writable activity: `Subject` (first line of the template), `Description` (the rest), `Status` = Completed, `ActivityDate` = today, `WhoId` = the Lead/Contact, `WhatId` = the Opportunity when there is one. Use an Enhanced Note (ContentNote) instead only if the connector supports it and the user prefers notes.
- **RGA package fields:** orgs with the RGA package have `rgaapp__RGA_Prep_Doc_URL__c` and `rgaapp__RGA_Prep_Session_ID__c` on Lead and Contact (no namespace prefix in unpackaged orgs). See "RGA fields" in SKILL.md. Write the Task as well.
- **Leads converted later** keep their Tasks, which follow the Lead to the Contact.

## Microsoft Dynamics 365 (Sales)

- **Person:** Contact (`firstname`, `lastname`, `emailaddress1`, `jobtitle`), or Lead (`subject` (Topic, required), `firstname`, `lastname`, `companyname`, `emailaddress1`, `jobtitle`). Search Contacts by `emailaddress1` first, then Leads. If neither exists and the org works leads-first, create a Lead (Topic: "RGA meeting prep: {Company}"); otherwise ask.
- **Deal:** Opportunity. Its customer can be a Contact (`parentcontactid`) or an Account. Use an existing open Opportunity. Create one only if the user asks, linked to the Contact, never by creating an Account.
- **Note:** Note (annotation): `subject`, `notetext`, regarding the Contact/Lead or Opportunity (`objectid`). Plain text. If the connector can't create annotations, use a completed Task activity regarding the same record.
- **Don't create:** Accounts, custom columns, sales processes.

## Pipedrive

- **Person:** Person (`name`, `email` (list), `org_id`). Search persons by email first.
- **Deal:** Deal (`title`, `person_id`, optional `org_id`, `stage_id`). A deal can belong to just a person. Create one only if the user asks, in their default pipeline's first stage.
- **Note:** Note (`content`, plus `person_id` and/or `deal_id`). `content` accepts simple HTML.
- **Don't create:** Organizations unless the user asks (attach an existing one by name if it clearly matches), pipelines, stages, custom fields.

## GoHighLevel (HighLevel)

- **Person:** Contact (`firstName`, `lastName`, `email`, `companyName`). Upsert by email is safe here: it updates an existing contact rather than duplicating.
- **Deal:** Opportunity, which needs a pipeline and stage. Use an existing one. Create one only if the user asks and names the pipeline.
- **Note:** Contact Note, if the connector has a notes tool. Many GoHighLevel connectors don't. In that case:
  1. If the contact has a custom field for an RGA link, set it.
  2. Add a tag such as `rga-prepped` or `rga-meddic` so the contact is findable.
  3. Tell the user the note text couldn't be saved, and give it to them to paste.
- **Don't create:** custom fields, pipelines, workflows.

## Any other CRM (Attio, Close, Zoho, Copper, monday CRM...)

Same pattern: person by email, existing deal only, a note or activity on each, no schema changes. If the connector's tools don't make the object model clear, ask the user one question ("Do you track these as Leads or Contacts?") rather than guessing, then carry on.
