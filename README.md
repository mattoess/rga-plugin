# Revenue Growth Agent plugin for Claude

Run Revenue Growth Agent (RGA) from Claude with a plain request: meeting prep, MEDDIC discovery analysis, deal checks, proposals and SOWs. The plugin bundles the RGA connector with skills that run each workflow in the right order, with the right checkpoints.

You need an RGA account. Sign up at [revenuegrowthagent.com](https://www.revenuegrowthagent.com).

## What you get

| Skill | Ask something like |
| --- | --- |
| `meeting-prep` | "Run a meeting prep on Jordan Lee at Acme." Or paste a list of trade show leads. |
| `daily-briefing` | "Prep everyone I'm meeting for the first time today." Runs as a 7am scheduled task. |
| `discovery-analysis` | "Here's my transcript from the Acme call. Run a MEDDIC analysis." |
| `deal-check` | "Where do we stand with Acme? What are the open gaps?" |
| `proposal-sow` | "Create a proposal for Acme: $15,000 a month for three months, built around a $1M ROI." |
| `call-autopilot` | "Score my sales calls from today and log them to the CRM." Runs every 2 hours as a scheduled task. |
| `crm-sync` | "Add this prep to HubSpot." / "Log the MEDDIC on the Acme opportunity in Salesforce." |
| `setup` | "Is RGA connected? Which org am I in?" |

`daily-briefing` and `call-autopilot` also use your own calendar, email, notetaker and CRM connectors in Claude. RGA never writes to your CRM itself: `crm-sync` has Claude do it through your CRM connector (HubSpot, Salesforce, Microsoft Dynamics 365, Pipedrive, GoHighLevel and others), finding records before creating them, adding one consistent note with the RGA link, and asking before it writes.

## What this plugin sends, and where

Everything RGA does runs through the RGA connector this plugin includes, under your own RGA login and the organization you choose when connecting.

- **To Revenue Growth Agent (revenuegrowthagent.com), through the connector:** prospect names, companies and contact details you ask to prep; call transcripts you ask to analyze; and proposal or SOW instructions. Results come back the same way.
- **Transcript files:** when a transcript is in a file, the `discovery-analysis` and `call-autopilot` skills have Claude upload it directly to `www.revenuegrowthagent.com` with a one-time upload link (a `curl` command in Claude's code sandbox), so long transcripts aren't retyped into the chat. Nothing is uploaded anywhere else.
- **Your CRM:** only when you ask, or in a scheduled task you set up, the `crm-sync`, `daily-briefing` and `call-autopilot` skills have Claude write contacts and notes to your CRM through **your own** CRM connector in Claude. RGA itself never connects to your CRM. Notes hold summaries and RGA links, never transcripts.
- **Your calendar, email and notetaker:** read through your own connectors only to prepare a request (who you're meeting, the call transcript). Only what the request needs is sent to RGA.

The plugin itself stores nothing and runs no code of its own; it contains only skill instructions and the connector address. See the [privacy policy](https://www.revenuegrowthagent.com/privacy) and [terms](https://www.revenuegrowthagent.com/terms).

## Install

**Claude (web, desktop, mobile) and Cowork**

1. Customize > Plugins > Add marketplace, and enter `mattoess/rga-plugin`.
2. Install **rga**.
3. Connect RGA when asked: sign in with your RGA account and choose your organization.

Already connected RGA as a connector? That's fine. Keep it, or remove it and use the plugin's.

**Claude Code**

```
/plugin marketplace add mattoess/rga-plugin
/plugin install rga@rga
```

**Teams:** a Team or Enterprise owner can install it for everyone under Organization settings > Plugins & skills.

Setup guide and FAQ: [revenuegrowthagent.com/mcp-connector](https://www.revenuegrowthagent.com/mcp-connector)

## How it's built

```
.claude-plugin/plugin.json       plugin manifest (bump version on every release)
.claude-plugin/marketplace.json  lets this repo act as its own install source
.mcp.json                        the RGA connector (remote, OAuth)
skills/<name>/SKILL.md           one folder per workflow
```

The connector itself lives in the `rga-mcp` repo. It enforces the rules that matter whatever a skill says: tenant isolation, a closed list of industries, explicit confirmation before a prep runs, explicit deal choice before a transcript is added to an existing opportunity, and human approval in RGA before a SOW. The skills cover the order of steps, what to ask the seller, and how to present results. They don't restate the server's rules, so there's nothing here to drift.

The connector also ships the same seven workflows as MCP prompts, for clients that don't install plugins.

## Develop

```
claude plugin validate --strict .claude-plugin/plugin.json
claude --plugin-dir .
```

CI runs the same validation, checks each skill's `name` matches its folder, and fails if a skill names a tool the connector doesn't register.

## Support

support@revenuegrowthagent.com · [Privacy](https://www.revenuegrowthagent.com/privacy) · [Terms](https://www.revenuegrowthagent.com/terms)
