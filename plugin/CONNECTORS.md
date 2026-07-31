# Connectors

This plugin requires no MCP connector. It reads intake answers and, optionally, your firm's own state-specific templates from a workspace folder you explicitly attach in Claude Desktop / Cowork — no separate authorization step, no credentials.

## How the plugin reads your files

Cowork's filesystem access is attach-only: the plugin can only see files inside a folder you've explicitly attached to the conversation. It does not browse your computer, does not search beyond that folder, and does not retain access after the conversation ends.

To use `/estate-documents`, attach a folder containing:

- Your client's intake answers (family structure, assets, beneficiaries, healthcare wishes — whatever you've gathered)
- Your firm's own state-specific templates, for any of the four document types you have one for — the skill uses its own bundled generic placeholder for any document type where it doesn't find one, and says so plainly

The plugin drafts from those files and presents the result for your review. You handle execution — witnessing, notarization, signing — entirely yourselves; this plugin never notarizes, files, records, or submits anything.

## Privacy note

The plugin processes the files in your attached folder within your Claude Desktop / Cowork conversation under your Claude plan's data handling terms. No intake answers, client names, or drafts are transmitted to Protomated or any third party.

For confidential client or matter information: confirm you are on Claude for Work, Claude Team, or Claude Enterprise before attaching a folder with client details. See the main README for plan requirements.
