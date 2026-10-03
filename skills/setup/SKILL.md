---
name: setup
description: Check and set up the Revenue Growth Agent (RGA) connection in Claude. Use when the user asks to set up, connect, test or troubleshoot RGA, asks which RGA workspace or organization they're in, or when an RGA tool is missing or returns MCP_ORGID_MISSING, e.g. "is RGA connected?", "set up RGA", "which RGA org am I in?".
---

# RGA setup

Confirms the RGA connector is connected and acting under the right workspace, then shows the seller what they can ask for.

## Steps

1. **Check the tools are there.** If no `rga_*` tools are available, the connector isn't connected or isn't enabled in this conversation. Tell the user:
   - In Claude, open Customize > Connectors, find RGA, and connect. If it isn't listed, add a custom connector with the URL `https://rga-mcp-production.up.railway.app/mcp` and leave the client ID and secret empty.
   - Sign in with their RGA account and pick their organization on the consent screen.
   - Make sure the connector is switched on for this chat (the tools menu).
   - Full steps: https://www.revenuegrowthagent.com/mcp-connector
2. **Check the workspace.** Call `rga_whoami`. Report the organization name.
   - If `workspaceProvisioned` is false, or there's no org: the connection has no RGA workspace. They need to disconnect, reconnect and choose their organization on the consent screen. If they don't see one, their RGA admin needs to invite them.
   - If it's the wrong organization: same fix. Disconnect, reconnect, and pick the right one. The choice made at sign-in applies to every call.
3. **Show what's possible.** Once connected, list what they can ask:
   - "Run a meeting prep on Jordan Lee at Acme."
   - "Here's my transcript from the Acme call. Run a MEDDIC analysis."
   - "Where do we stand with Acme?"
   - "Create a proposal for Acme." (needs a discovery analysis first)
   - A 7am daily briefing and automatic call scoring, as scheduled tasks. Offer to set either up.

## Optional: a project for discovery work

Sellers who analyze a lot of calls do better with a Claude project whose instructions list their solutions and active opportunities (company and RGA opportunity id, the part of the RGA link after `/opportunity/`). It lets Claude match a transcript to the right deal instead of asking every time. Offer to draft those instructions from `rga_find_opportunities` results.
