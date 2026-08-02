# GoogleAds

Daily Google Ads analysis for Loaded Dice (account 719-313-4499).

- `PLAYBOOK.md` — the rules the automated daily analysis follows: scope
  (live campaigns only, Glopal excluded), data pulls, thresholds,
  adjustment policy and report format.
- `reports/` — one dated markdown report per day, written by the
  scheduled Claude session each morning.

Currently in **recommend-only** mode: the daily run never changes the
account. Enable auto-apply by updating the Adjustment policy section of
`PLAYBOOK.md`.
