# meet2invoice

**meet2invoice makes EU freelance deals fluent.** A sales call in any language
becomes a lawful, provable Qonto invoice: the AI extracts the deal, you approve
twice, the document never leaves your machine — only a cryptographic hash is
written into the official invoice footer.

📹 **Demo video:** _link here in the PR description_

## Why this matters

After every sales call, someone re-types the deal — scope, amount, VAT,
payment terms — into a finance tool. meet2invoice closes that loop from the
raw meeting artifact (transcript, notes, calendar event) to a finalized,
sent Qonto invoice, with two hard confirmation gates and an offline-verifiable
proof trail:

- **Guarded action.** Nothing is written to Qonto without an explicit preview
  and confirmation — once before the quote, once before the invoice.
- **Zero document exfiltration.** The authoritative quote PDF is downloaded
  and hashed locally (`shasum -a 256`). Only the 64-hex-char SHA-256 travels
  back into Qonto, rendered in the footer of the finalized invoice PDF.
- **Law-aware.** VAT treatment (domestic / intra-EU reverse charge / export)
  is decided from the parties' countries via `references/invoicing-law.md`,
  including the mandatory legal mentions — in the client's language.
- **No money movement.** Quotes and invoices only; payments always stay with
  Qonto and the user's own SCA.

## Quick start

1. Connect the [Qonto MCP server](https://qonto.com) in Claude Code.
2. Copy this folder into your skills directory (or clone `qonto/skills`).
3. Feed it a meeting artifact:

   - "Here's the transcript from my Bouygues call — meet2invoice it."
   - 🇩🇪 "Mach aus diesem Telefonprotokoll ein Angebot und dann die Rechnung."
   - 🇫🇷 "Cet appel est clôturé — facture en autoliquidation, TVA 0, mentions OK."
   - 🇮🇹 "Deal chiuso: crea la fattura, IVA 22%, serve il codice destinatario SdI."
   - 🇪🇸 "Crea un borrador de factura con IVA 21%, pero no la emitas ni la envíes."

   Sample transcripts for every case live in [`demo/`](demo/).

## Verify the proof yourself

```bash
scripts/verify-proof.sh quote.pdf invoice.pdf
# ✔ PROOF VERIFIED
#   The invoice PDF contains the full SHA-256 of the local quote PDF.
#   The document never left this machine — only the hash traveled.
```

Works offline with no MCP and no dependencies beyond `shasum`
(uses `pdftotext` when available, otherwise a built-in python3 fallback).

**Both sides of the deal can run this.** The freelancer verifies before
sending. The company that hired them downloads the quote PDF from the public
`quote_url` and independently checks that the invoice in their inbox matches
the quote they accepted — no Qonto account, no trust in the sender required.

## Production-ready, not demo-ware

Every call is a GA Qonto Business API endpoint through the official MCP — the
identical flow runs on a real account today. The skill documents exactly what
changes in production (SCA prompts on sensitive writes, multi-account IBAN
selection, manual-numbering orgs, currency handling, FR/DE/IT e-invoicing
mandates, payment links once a provider is connected) and handles real cases
the demo never shows: existing-quote escalation, B2C vs B2B VAT, non-EU
exports, seller small-business schemes (§19 UStG / art. 293 B CGI), credit
notes and payment lifecycle after send. Validated end-to-end twice against the
official sandbox — including a full intra-EU reverse-charge invoice with the
mandatory legal mention, offline-verified proof, and localized client email.

## What this skill deliberately does NOT do

- It never moves money, creates transfers, or touches payment approval.
- It never sends an invoice without the second confirmation gate.
- It never uploads your documents anywhere — only the hash travels.
- It does not give legal or tax advice; it applies documented invoicing rules
  and asks when data (VAT ID, rate, country) is missing.
- Recurring/retainer invoices and deposit (down-payment) invoices are not
  automated yet — create them in Qonto directly; the skill focuses on the
  deal-to-first-invoice loop.

## Contents

| Path | Purpose |
|---|---|
| `SKILL.md` | The skill definition (flow, gates, verified tool chain, error handling) |
| `scripts/verify-proof.sh` | Offline proof verifier (quote hash ↔ invoice footer) |
| `references/invoicing-law.md` | Per-country VAT / e-invoicing rules used at extraction time |
| `demo/` | Multi-language demo transcripts + the 3-minute video script |
| `sign-ring/` | Optional zero-dependency signature MCP (hash-only multi-party sign-off) |
| `SANDBOX-VALIDATION.md` | Full validation log against the official Qonto sandbox MCP |

Validated end-to-end against the official Qonto sandbox MCP on 2026-07-12:
`create_quote` → `get_attachment` → local hash → `create_client_invoice` (proof
in `terms_and_conditions`) → finalize → footer renders the proof → `send_client_invoice`.
