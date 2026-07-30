# Demand Letter & Client Correspondence Drafter v1.0.0

Initial release.

## What's included

### `/demand-letter` — Demand Letter & Client Correspondence Drafter

A demand-letter and client-correspondence drafter for solo and small-firm attorneys:

- **Attach a case folder — get a first-pass demand letter:** the skill reads your case facts and your firm's own demand-letter template from an attached workspace folder and drafts a populated first pass, following your firm's structure and phrasing.
- **Or draft a plain-English client status update:** summarizes what's happened and what's next in language a client can follow, no legal jargon.
- **Never sets a demand amount or a legal conclusion:** the skill leaves an explicit placeholder for the demand figure and declines to assess liability or case value — that stays the attorney's call.
- **Clarifies before drafting, never guesses:** if the output type, template, recipient details, or facts are unclear, the skill asks — one question at a time — rather than filling gaps with plausible-sounding detail.
- **Attorney review gate:** presents every draft with an explicit confirmation step before marking it ready to send. Never sends, files, or transmits anything anywhere.

Handles: demand letters populated from a firm template and case facts, and plain-English client status-update emails — the two correspondence types attorneys draft nearly identically, matter after matter.

## Setup

Install time: about 5 minutes. Download the zip, drag it into Claude Desktop's Extensions panel, and attach a workspace folder with your case facts and (for demand letters) your firm's template. No connectors to authorize. Open a new chat, type `/skills`, and verify `/demand-letter` appears.

## Compliance

Requires Claude for Work, Claude Team, or Claude Enterprise. Do not use a consumer Claude plan (Claude Pro or Personal) with confidential matter information. Every output carries an "ASSISTED DRAFT — ATTORNEY REVIEW REQUIRED" header and footer, shown around the draft — never inside the document you'll send. The skill never marks a draft ready without your explicit confirmation, never sets a demand amount or liability position, and never sends anything itself. All drafts are populated from your case folder only — no facts are invented.
