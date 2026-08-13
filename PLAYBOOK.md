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

- **Under-tracking: KNOWN AND ACCEPTED (Graham's call, 8 Aug). Do not
  re-investigate; do not raise it as an alarm.** From 4 Aug, Google
  tracked only 2-13% of Shopify revenue (was ~40%), across BOTH purchase
  actions, alongside a bot-traffic surge and a GA4/Shopify session
  divergence. Too much changed on the site to isolate a cause, and the
  bot traffic is expected to subside as TCG lines are wound down. Mention
  it in one line per report, not as a banner.
- **DECISION BASIS while tracked ROAS is unreliable.** Absolute ROAS is
  meaningless right now — the shop is trading normally (42-75 orders/day)
  so the shortfall is measurement, not performance. Use instead:
  1. **Shopify total sales** (`FROM sales SHOW orders, total_sales
     TIMESERIES day`) as the truth source for whether trading is healthy.
  2. **Relative comparison between campaigns** — under-tracking hits all
     campaigns roughly equally, so campaign-vs-campaign ROAS ranking and
     week-on-week direction remain valid even though the absolute level
     does not.
  3. **Non-revenue campaign health**: spend vs budget, CPC, impression
     share, search-term quality, click volume.
  Report the **tracked share %** each day as one context line. If it
  returns to ~30-40% for 3 consecutive days, say so and re-open the
  frozen budget decisions.
  **Recovering as of 13 Aug — TWO of three matured days in band:**
  6% (5 Aug) → 12% → 19% (8 Aug) → **33% (9 Aug)** → **37% (10 Aug)** →
  25% (11 Aug, d+2) → 10% (12 Aug, d+1). Days mature upward for ~3 days,
  so re-check aged days each morning before judging. If 11 Aug matures
  into the 30-40% band, the three-day rule is met and the PMAX budget
  decision re-opens. Do NOT re-open early — earlier recommendations were
  wrecked by acting on unmatured, under-tracked data.
- **Never present an absolute ROAS figure without the tracked-share
  caveat** while this persists — someone reading the report later must
  not mistake 0.84 for real performance.
- **TCG wind-down is in progress** (Graham, 8 Aug — internal only, never
  reference in any customer-facing copy). Expect product mix, PMAX feed
  composition and traffic profile to shift over the coming weeks. Treat
  drops in TCG-related campaign performance as expected, not as faults,
  and flag if the PMAX feed needs restructuring as lines disappear.
- **ALWAYS sanity-check Google-reported revenue against Shopify before
  concluding a campaign is underperforming.** Learned 8 Aug: two days of
  recommendations (a PMAX scale-up, then its withdrawal on
  "diminishing returns") were both built on tracked ROAS that was
  silently collapsing.

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
  - **Flagged 6 Aug: CPM tripled** (£2.31 → £8.41) because the campaign
    still uses MAXIMIZE_CONVERSIONS bidding with zero conversions by
    design — no signal to optimise toward. Track CPM daily; baseline to
    restore is ~£2.50-3.50.
  - **7 Aug: Maximise Clicks is NOT available for this campaign.** Graham
    approved the switch; Google rejected both `maximize_clicks` and
    `target_spend` with "operation is not allowed for the given context"
    (Demand Gen campaigns don't accept it via API). DO NOT retry this —
    it will fail again.
  - **7 Aug — ROOT CAUSE FOUND, and it is worse than a CPM problem.**
    Campaign-specific goals were ALREADY set ("YouTube engagements", both
    actions ticked), so that was never the fix. Checking `all_conversions`
    (counts output regardless of primary/secondary) shows YouTube channel
    subscriptions went 141-201/day up to 2 Aug, then **0 every day from
    3 Aug** — the day after the actions were set account-level Secondary.
    £67.98 spent since for zero subscriptions. Campaign-level ticks govern
    reporting; the account-level Secondary flag governs bidding.
  - **The 2 Aug Secondary change was a bad trade** — it was meant to
    protect PMAX/Search value bidding, but YouTube conversions land almost
    entirely on Demand Gen (582 vs 23 on PMAX over 30 days, against PMAX's
    293 purchases). Negligible protection gained, whole objective lost.
  - **FIXED by Graham 7 Aug.** Under Goals → Engagements → Goal settings:
    Account default = Off, campaign-specific optimisation = Demand Gen
    only, and "YouTube channel subscriptions" (YouTube hosted) set to
    **Primary**. This is the ideal configuration — Demand Gen gets its
    bidding signal back and PMAX/Search/EUGY are untouched.
  - **VERIFIED WORKING 8 Aug.** 7 Aug produced 40 YouTube subscriptions
    (first since 2 Aug) with impressions up 3× to 4,335. Still below the
    141-201/day norm; bid strategy is re-learning. Do not recommend
    changes to this campaign before ~11 Aug. Track subs/day and CPM
    (target ~£2.50-3.50; was £6.91 on 7 Aug). Spend hit £29.94 vs the £20
    budget — normal post-change, but flag if it persists past 11 Aug.
    Ignore CPC for this campaign: it optimises for subscriptions, so CPC
    rising (£0.20 → £0.64) is expected, not a problem.
  - **WATCH ITEM CLOSED 10 Aug — fully recovered.** Subs 0 → 40 → 46 →
    120; CPM £8.41 → £6.91 → £4.49 → £2.71 (inside the £2.50-3.50
    baseline); impressions back to 8,236/day. Re-learning took three days
    from Graham's 7 Aug campaign-specific goal fix. Routine monitoring
    only: flag if spend exceeds £20/day sustained or CPM returns above
    ~£5.
  - **Reading trap (cost a day, 7 Aug):** the "All your goals" summary row
    shows a COUNT of primary actions per goal, not which ones. Engagement
    showed "1 primary" — that was "Local actions - Other engagements",
    while YouTube channel subscriptions sat secondary beneath it. Never
    conclude an action's status from the goals summary; open Goal settings
    → Conversion action optimisation to see per-action status.
  - **General lesson: check `all_conversions`, not `conversions`, when
    judging whether a change broke a campaign's actual output.** The
    `conversions` field hides everything marked secondary.
- **Conversion lag — revised again 7 Aug: day+3 indicative, day+5
  final.** 3 Aug went £152 (d+1) → £345 (d+2) → £608 (d+3, then flat).
  2 Aug went £447 → £476 → £527 (d+3) → £578 (d+4) — i.e. still moving
  after day+3. Rule: never judge a day under 3 days old at all; treat
  day+3/+4 as indicative only; treat day+5 as final. The headline metric
  each morning is the **matured window (days 3+ old)**, labelled as
  indicative at its recent edge.
- **Do not recommend a budget change off a window that excludes the most
  recent test of the last change** (learned 7 Aug: recommended a PMAX
  scale-up on 6 Aug from a window ending 3 Aug, then had to withdraw it
  when 4 Aug — the only day that actually spent the raised budget —
  matured at 2.06 ROAS). Before any scale recommendation, explicitly
  check the highest-spend days since the previous change.
- **PMAX budget — HOLD at £94.80/day, decision suspended 8 Aug.** The
  6 Aug scale-up recommendation and its 7 Aug withdrawal were both built
  on ROAS from 4-7 Aug, which the under-tracking invalidated — 4 and 5
  Aug were the first badly under-tracked days, so their poor ROAS proves
  nothing about the budget. Re-open only when tracked share recovers to
  ~30-40% for 3 consecutive days. Holding is the right call meanwhile:
  spend is steady at ~£85-110/day, the shop is trading normally, and
  neither raising nor cutting can be justified from the data.
- **Brand Search PHRASE match — STANDING SUGGESTION, do NOT re-raise
  daily.** Structural fix: change the "Loaded Dice" keyword from BROAD to
  PHRASE (keeps all brand queries, removes the generic dice/D&D/40k
  category). Worth roughly £8/day — the generic share has run 62%, 24%,
  58%, 65%, 34% across measured days, averaging about half of spend on
  terms that have never once converted. It is a tidy-up, not a fire.
  **Mention only when Graham asks, or if spend exceeds the £20 budget for
  three consecutive days.** To apply: remove_keywords (needs ad_group_id
  + keyword_criterion_id) then push_keywords with match_type PHRASE.
  - *Process note (learned 11 Aug):* this recommendation was raised,
    softened, escalated and de-escalated across five days by reacting to
    single-day swings — 9 Aug's 65% was an outlier, and 10 Aug reverted
    to 34% with spend back inside budget. Judge recurring structural
    issues on multi-day averages and state a stable position; do not let
    daily noise drive the recommendation up and down.
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
