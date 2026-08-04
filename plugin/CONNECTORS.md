# Connectors

This plugin requires no MCP connector. The audit runs as a guided interview in chat; if your firm already has an AI-tools list written down, you can optionally attach it as a workspace folder in Claude Desktop / Cowork — no separate authorization step, no credentials.

## How the plugin reads your files

Cowork's filesystem access is attach-only: the plugin can only see files inside a folder you've explicitly attached to the conversation. It does not browse your computer, does not search beyond that folder, and does not retain access after the conversation ends.

To use `/ai-tool-audit`, you don't need to attach anything — the skill will interview you in chat about which AI tools your firm uses. If you'd rather start from a written list, attach a folder containing:

- An existing AI-tools inventory, in whatever format you have it (a markdown list, a spreadsheet exported as text, notes) — the skill confirms it's complete before treating it as the full inventory, and fills in any missing detail (workflow, data touched, data-handling status) through the interview

The plugin builds a tool inventory, rates each tool's data-handling status, flags redundant tools, and recommends consolidation candidates for your review. It does not log into, change a setting on, migrate data from, or cancel anything at any vendor — you (or a separate Protomated engagement) carry out anything the audit recommends.

## Privacy note

The plugin processes your interview answers and any attached inventory file within your Claude Desktop / Cowork conversation under your Claude plan's data handling terms. No tool inventory, data-handling findings, or audit results are transmitted to Protomated or any third party.

For your firm's actual AI-tool landscape: confirm you are on Claude for Work, Claude Team, or Claude Enterprise before running this interview, and describe data categories rather than real client names or matter numbers while answering. See the main README for plan requirements.
