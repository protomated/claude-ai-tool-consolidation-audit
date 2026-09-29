# AI Tool Consolidation & Data-Hygiene Audit v1.0.1

Adds Legal Builder Hub freshness frontmatter (`freshness_category: procedural`). No functional changes.

## What's included

### `/ai-tool-audit` — AI Tool Consolidation & Data-Hygiene Audit

A data-hygiene audit assistant for solo and small-firm attorneys managing a growing sprawl of AI tools:

- **Short guided interview:** the skill asks which AI tools the firm uses, for what workflow, who uses them, and what data they touch — about 10 minutes, with an optional existing AI-tools list to speed it up.
- **Data-handling rating per tool:** confirmed appropriate, partial/mixed, confirmed risk, or UNCONFIRMED — never a guessed rating dressed up as a confirmed one.
- **Redundancy flagged by workflow:** tools serving the same stated task are grouped and flagged, without assuming overlap automatically means one has to go.
- **Consolidation recommendation, tied to real workflows:** every recommendation to consolidate cites the specific workflow and the specific gap found. Genuinely specialized tools — practice management systems, docketing/deadline engines, e-discovery, e-signature — are named explicitly as "keep," not silently folded into a blanket "move everything to Claude" pitch.
- **No vendor claims from model knowledge:** the skill never asserts what a named tool's current data-handling terms are from its own training data — unconfirmed terms are flagged for the firm to verify with the vendor directly.
- **Recommends, doesn't act:** the skill never accesses, changes, migrates, or cancels anything at any vendor. Every output is a recommendation for the firm (or a separate engagement) to carry out.

Handles: a firm-wide AI-tool inventory and data-hygiene first pass — the "what are we even using, and is any of it a problem" audit most firms have never had time to do, positioned as one governed system rather than one more point tool to add to the pile.

## Setup

Install time: about 5 minutes. Download the zip, drag it into Claude Desktop's Extensions panel. No connectors to authorize. Open a new chat, type `/skills`, and verify `/ai-tool-audit` appears. Optionally attach a workspace folder with an existing AI-tools list before running it.

## Compliance

Requires Claude for Work, Claude Team, or Claude Enterprise for any interview touching your firm's actual tool landscape. Every audit carries an "ASSISTED AI-TOOL AUDIT — ATTORNEY REVIEW REQUIRED BEFORE USE" header and footer. The skill is not a security assessment and does not certify compliance with any bar rule, ethics opinion, or security standard; it never invents a tool's data-handling terms; it never drafts the firm's actual AI-use policy (a separate skill); and it never accesses, changes, migrates, or cancels anything at any vendor.
