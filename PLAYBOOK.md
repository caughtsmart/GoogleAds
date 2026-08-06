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
  £1-value micro-conversions. Never treat these as revenue; report such
  campaigns on engagement cost, not ROAS.
- **Awareness campaigns are exempt.** Campaigns Graham has designated as
  awareness plays (currently Demand Gen Brand Awareness) are excluded from
  every conversion-based check above — see the standing watch items.

## Adjustment policy

**Current mode: RECOMMEND-ONLY** (confirmed by Graham 2026-08-02: no
auto-apply until a few manual run-throughs have built confidence). The
daily run makes no changes to the account. Every recommendation is listed
in the report with the exact Windsor.ai action that would implement it,
so Graham can say "do rec 2" in any session.

Standing watch items for the daily run:

- **Brand Search rebuild VERIFIED WORKING 5 Aug** — config confirmed in
  data (TARGET_SPEND, negatives present, CPC £0.19, spend £72 → £23 → ~£10,
  clean brand search terms). Routine monitoring now, not active watch:
  flag only if spend pegs the £20 cap with junk terms (recommend more
  negatives — "dnd books" and "ork dice" are the current strays), or if
  impression share falls while CPC sits at the £0.30 ceiling (competitor
  bidding on the brand — recommend nudging the ceiling up).
- **No generic Warhammer search campaign** — decided with Graham 4 Aug.
  Three attempts all failed (Warhammer Search £120/0 conv; WH40K Launch
  ROAS 0.55; warhammer keywords in Brand Search ~1.2 ROAS) while PMAX
  serves the same demand at 7+ ROAS with only ~20% impression share.
  Warhammer growth lever = scale PMAX while ROAS holds. Exception worth
  proposing around major releases: narrow exact-match campaigns on
  specific high-intent release terms with a tROAS, judged after 2 weeks.
- **Demand Gen Brand Awareness — SETTLED POLICY (Graham, 5 Aug).** This
  campaign is deliberately an awareness play at £20/day. Zero or very few
  conversions is the EXPECTED and ACCEPTED outcome, not a problem.
  Therefore:
  - Do NOT flag it under the zero-converter rule, the ROAS floor, or any
    conversion-based rule. It is exempt from all of them.
  - Do NOT recommend pausing it, cutting it, or "fixing" its conversions.
  - EXCLUDE it from account-level ROAS and revenue totals; report its
    spend separately as awareness cost.
  - Judge it only on awareness efficiency: spend vs budget, impressions,
    reach, CPC/CPM trend. Flag ONLY if spend exceeds the £20/day budget,
    or CPC/CPM rises sharply (i.e. awareness getting expensive).
  - **Flagged 6 Aug: CPM tripled** (£2.31 → £6.81) because the campaign
    still uses MAXIMIZE_CONVERSIONS bidding with zero conversions by
    design — no signal to optimise toward. Recommended switching to
    Maximise Clicks, budget unchanged. Awaiting Graham. Track CPM daily;
    baseline to restore is ~£2.50-3.50.
- **Conversion lag is large — revised 6 Aug: maturation takes 3 FULL
  DAYS.** 3 Aug PMAX revenue went £152 (day+1) → £345 (day+2) → £608
  (day+3). 2 Aug settled at day+3 and held. NEVER judge a day less than
  3 days old. The headline metric each morning must be the **matured
  window (days 3+ old)**; label anything newer "provisional" and do not
  draw conclusions from it. A drop in recent-day ROAS is lag until
  proven otherwise — do not raise it as a concern without checking the
  same day again once matured.
- **PMAX budget: scale-up recommended 6 Aug, awaiting Graham.** Both
  thresholds met on matured data (30d ROAS 6.57 ≥ 6; matured 5d ROAS 5.37
  = 82% of 30d ≥ 70%; IS 22%). Recommended +20% to ~£114/day; it already
  spends at that level under Google's 2× daily allowance. If approved,
  re-check matured ROAS 4 days later before any further increase.
- **Brand Search: BROAD match drift flagged 6 Aug.** Negatives work, but
  broad "loaded dice" now matches the generic dice/DnD/Kill Team market
  (62% of yesterday's spend). Recommended switching the keyword to PHRASE
  match — keeps all brand queries, kills the category drift, avoids
  negative-keyword whack-a-mole. If Graham declines the match-type change,
  fall back to recommending category negatives ("dice tray", "dnd dice",
  "kill team"). Awaiting decision.
- **EUGY watch CLOSED 6 Aug** — the £0.76 CPC spike was a one-day blip on
  16 clicks; CPC back to £0.44, matured ROAS 2.85. Routine monitoring.

- **Brand Search transition (from 2 Aug):** campaign-level phrase negatives
  "warhammer" / "wayland games" / "dawn of palpagos" were added on
  2026-08-02, which should cut its generic spend (~£30/day) sharply. Verify
  spend drops while brand-term revenue holds; flag if revenue falls too.
- **GS | PMAX New at £94.80/day (+20% on 2 Aug):** confirm ROAS holds ≥5
  at the higher budget before recommending further increases.
- **YouTube conversion actions set to Secondary by Graham on 2 Aug**
  ("channel subscriptions" + "follow-on views"). The API still showed
  primary_for_goal=True immediately after — verify the flip has propagated
  (both False, and Demand Gen conversions dropping to ~0 for new days).
  If still primary after 3 Aug, tell Graham the UI change didn't stick.
  Until confirmed, exclude Demand Gen from conversion totals.
- **Conversion hygiene verified 2 Aug:** all live campaigns' Conversions
  come solely from "Google Shopping App Purchase" (primary; Shopify
  server-side) — correct, no double counting; GA4 purchase is secondary.
  Outstanding UI cleanup: "Loaded Dice - GA4 (web) purchase_enhanced" is
  misconfigured (4,254 "purchases" @ £1 in 30d, secondary) — fix or remove;
  remind daily until resolved, but it does not affect bidding.
- **Paused on 2 Aug:** Warhammer Search (21071347135), WH40K 11th Ed
  Launch (24049932610). Leave paused; ignore their residual data.

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
