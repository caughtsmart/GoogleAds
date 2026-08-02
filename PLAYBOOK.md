# Loaded Dice — Daily Google Ads Analysis Playbook

This playbook drives the automated daily analysis of the Loaded Dice Google Ads
account (**719-313-4499**, GBP). A scheduled Claude session runs it every
morning, writes a dated report to `reports/`, and surfaces recommendations.

## Scope

- **Account:** Google Ads via the Windsor.ai connector (`google_ads`).
- **Live campaigns only:** `campaign_status = ENABLED`.
- **EXCLUDE all Glopal campaigns** — any campaign whose name contains
  `Glopal` (the DE/NL/FR PMax + Brand campaigns managed by Glopal). Report
  their total spend as a single FYI line, nothing more.

## Data pulls (Windsor.ai `get_data`, connector `google_ads`)

1. **30-day campaign totals:** fields `campaign, campaign_id, campaign_status,
   bidding_strategy_type, spend, clicks, impressions, conversions,
   conversion_value, search_impression_share`, preset `last_30d`.
2. **Daily trend:** fields `campaign, date, spend, clicks, conversions,
   conversion_value`, preset `last_14d`.
3. Derive per campaign: ROAS (`conversion_value / spend`), CPC, CVR,
   yesterday vs trailing-7-day and trailing-30-day averages.

## Analysis checklist

For each live, non-Glopal campaign:

- **ROAS vs target.** Blended target: **3.0** (breakeven floor ~2.0 on
  typical tabletop margins). Flag anything below 2.0 sustained over 14+ days.
- **Zero-converter check.** Spend > £50 over 30 days with 0 conversions →
  recommend pause.
- **Spend spikes.** Yesterday's spend > 1.5× its 7-day average → flag.
- **ROAS deterioration.** 7-day ROAS < 60% of 30-day ROAS → flag.
- **Scale opportunities.** ROAS ≥ 5 and search impression share < 40% →
  candidate for budget increase.
- **Conversion hygiene.** Watch for campaigns whose "conversions" are
  £1-value micro-conversions (e.g. Demand Gen Brand Awareness counts
  page-engagement-style actions at £1 each). Never treat these as revenue;
  report such campaigns on engagement cost, not ROAS.

## Adjustment policy

**Current mode: RECOMMEND-ONLY.** The daily run makes no changes to the
account. Every recommendation is listed in the report with the exact
Windsor.ai action that would implement it, so Graham can say "do rec 2"
in any session.

Pre-agreed guardrails for when auto-apply is enabled (not yet):

- Budget changes capped at ±20% per campaign per day.
- Pauses only for campaigns meeting the zero-converter rule.
- Never touch: Glopal campaigns, paused campaigns, bidding strategy types,
  campaign creation/deletion.
- Every change must be logged in the daily report with before/after values.

## Report format

Write `reports/YYYY-MM-DD.md` containing:

1. **Headline** — yesterday's spend, revenue (conversion value), blended ROAS
   for live non-Glopal campaigns, vs 7-day average.
2. **Campaign table** — 30d spend, conversions, value, ROAS, trend arrow.
3. **Flags & recommendations** — prioritised, each with the concrete action.
4. **Actions taken** — always "none (recommend-only mode)" until the policy
   changes.
5. **FYI** — one line of Glopal total spend; anything odd in the data.

Commit the report to the repo and push. Keep the report under ~60 lines —
it's a daily brief, not an audit.
