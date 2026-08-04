# Connectors

This plugin requires no MCP connector. It reads the contract you're reviewing and, optionally, your firm's own playbook from a workspace folder you explicitly attach in Claude Desktop / Cowork — no separate authorization step, no credentials.

## How the plugin reads your files

Cowork's filesystem access is attach-only: the plugin can only see files inside a folder you've explicitly attached to the conversation. It does not browse your computer, does not search beyond that folder, and does not retain access after the conversation ends.

To use `/contract-review`, attach a folder containing:

- The contract you want reviewed (paste the text, or attach it as a file the conversation can read)
- Your firm's own `playbook.md`, encoding your negotiation positions clause by clause — the skill uses its own bundled generic playbook for any clause type it doesn't find covered there, and says so plainly

The plugin reviews those files and presents its clause-by-clause findings, with suggested redline language, for your review. It does not edit your contract file directly, apply Word tracked changes, or send the review or the contract anywhere — you copy the suggested language into your own document and handle negotiation, execution, and delivery entirely yourselves.

## Privacy note

The plugin processes the files in your attached folder within your Claude Desktop / Cowork conversation under your Claude plan's data handling terms. No contract text, playbook positions, or reviews are transmitted to Protomated or any third party.

For confidential client or matter information: confirm you are on Claude for Work, Claude Team, or Claude Enterprise before attaching a folder with contract or client details. See the main README for plan requirements.
