# Signal — Status

*Working name (rename freely). Companion status doc; logs structural decisions as the project evolves.*

**Last updated:** 2026-06-28
**Stage:** Scoping — awaiting Stage 1 + Stage 3 answers (see `SIGNAL-QUESTIONS.md`)

## What this is
A manually-run research routine that tracks how commodity economics (prices, supply disruptions, extraction/policy deals) move specific company stock prices. First domain: rare earth elements (REE).

## Decisions logged
- **2026-06-28 — Framing: policy/supply is the spine, price is a secondary confirmer.**
  For REE specifically, individual oxides have no liquid public-market price; pricing is paid/assessed (Argus, Fastmarkets, Asian Metal). Stocks in this space move more on policy events, supply disruptions, and government contracts than on reported oxide prices. So the architecture leads with news / policy / filings; price *confirms* rather than *triggers*.
  *Open to revision if Stage 1 answers point to a short-horizon trading use case.*

## Open questions / load-bearing unknowns
- **Purpose not fixed** (investing vs content vs demo vs learning) — gates everything.
- **Price-data strategy** unresolved (pay vs proxy vs skip).
- **Source strategy** unresolved (primary vs trade press; scraping ToS constraints).
- **Company universe** undefined.

## Next action
Answer Stage 1 (Q1–3) and Stage 3 (Q8–10). Those unlock the first architecture draft.
