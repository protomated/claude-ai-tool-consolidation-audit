# Billing Narrative & Time-Entry Drafter v1.0.0

Initial release.

## What's included

### `/billing-narrative` — Billing Narrative & Time-Entry Drafter

A billing narrative drafter for solo and small-firm attorneys:

- **Paste rough notes — get a ready-to-bill narrative:** supply shorthand, fragments, or a forwarded email; the skill produces professional past-tense billing language specific to the activity. No templates, no form fields.
- **Clarifies before drafting, never guesses:** if the notes are ambiguous about the activity type, whether to split entries, or what time was spent, the skill asks — one question at a time — rather than filling gaps with plausible-sounding detail.
- **Suggests a time increment:** rounds to 0.1-hour or 0.25-hour billing increments based on your preference; flags time suggestions that are estimates rather than attorney-provided figures.
- **UTBMS/ABA task and activity codes on request:** suggest the appropriate L-code and A-code for corporate or insurance-defense billing; freeform for firms that don't use codes.
- **Attorney review gate:** presents every draft with an explicit confirmation step before marking it ready to paste. Never submits, records, or transmits entries anywhere.

Handles: conference call notes, email threads, court appearance descriptions, research sessions, drafting sessions, multi-activity bundles, and any rough time record where the bottleneck is writing the narrative, not remembering what happened.

## Setup

Install time: under 3 minutes. Download the zip, drag it into Claude Desktop's Extensions panel. No connectors to authorize. Open a new chat, type `/skills`, and verify `/billing-narrative` appears.

## Compliance

Requires Claude for Work, Claude Team, or Claude Enterprise. Do not use a consumer Claude plan (Claude Pro or Personal) with confidential matter information. Every output carries an "ASSISTED DRAFT — ATTORNEY REVIEW REQUIRED" header. The skill never marks an entry ready to paste without your explicit confirmation. All narratives are drafted from your notes only — no facts are invented.
