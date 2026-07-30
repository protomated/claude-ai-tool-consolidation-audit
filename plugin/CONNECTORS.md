# Connectors

This plugin requires no MCP connector. It reads case facts and your firm's demand-letter template from a workspace folder you explicitly attach in Claude Desktop / Cowork — no separate authorization step, no credentials.

## How the plugin reads your files

Cowork's filesystem access is attach-only: the plugin can only see files inside a folder you've explicitly attached to the conversation. It does not browse your computer, does not search beyond that folder, and does not retain access after the conversation ends.

To use `/demand-letter`, attach a folder containing:

- The case facts you want drafted from (intake notes, correspondence, treatment summaries, incident reports — whatever you have)
- Your firm's demand-letter template, if you're drafting a demand letter (the skill asks for one rather than guessing at a structure if it can't find it)

The plugin drafts from those files and presents the result for your review. You copy the final letter or email and send it yourself — this plugin never emails, files, or submits anything.

## Privacy note

The plugin processes the files in your attached folder within your Claude Desktop / Cowork conversation under your Claude plan's data handling terms. No case facts, client names, or drafts are transmitted to Protomated or any third party.

For confidential matter information: confirm you are on Claude for Work, Claude Team, or Claude Enterprise before attaching a folder with client details. See the main README for plan requirements.
