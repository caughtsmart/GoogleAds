# Loaded Dice — Daily Google Ads Analysis Playbook

This playbook drives the automated daily analysis of the Loaded Dice Google Ads
account (**719-313-4499**, GBP). A scheduled Claude session runs it every
morning, writes a dated report to `reports/`, and surfaces recommendations.

## Scope

- **Account:** Google Ads via the Windsor.ai connector (`google_ads`).
- **Live campaigns only:** `campaign_status = ENABLED`.
- **EXCLUDE all Glopal campaigns** — any campaign whose name contains
  `Glopal` (the DE/NL/FR PMax, Brand and Shopping campaigns managed by
  Glopal). Report their total spend as a single FYI line, nothing more.
  Exception: DO flag structural changes on their side (campaigns
  launched, paused or removed), since those move total account spend and
  Graham may not be told. Glopal paused all three PMax campaigns on
  13/14 Aug; their Brand and Shopping campaigns remain live.

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
  candidate for budget increase. **Always check the impression-share
  SPLIT before recommending budget** (`search_budget_lost_impression_
  share` vs `search_rank_lost_impression_share`): budget only helps when
  impressions are lost to BUDGET. PMAX loses 69.8% to rank and 9.9% to
  budget, so more budget buys it almost nothing; EUGY loses 25.3% to
  budget, so it does. For small campaigns that are demonstrably
  budget-capped and above the 3.0 target, a modest test increase is a
  reasonable exception to the ROAS ≥ 5 bar — say so explicitly and let
  Graham decide.
- **Brand campaigns: judge coverage on EXACT-MATCH impression share.**
  The headline `search_impression_share` for a broad-matched brand
  keyword includes the whole generic category it deliberately avoids, so
  it reads misleadingly low. Brand Search showed 22.9% headline but
  **81% exact-match** — near saturation, nothing to recover. A false
  alarm was raised on this on 19 Aug and closed on 20 Aug.
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

- **EUGY budget raised to £15.00/day by Graham, 21 Aug** (from £12.50;
  confirmed in the API). Rationale: 7-day ROAS 5.66 with 21.5% of
  impressions lost to the budget cap. **Review on 28 Aug** using matured
  data for 22-27 Aug:
  - Keep at £15 if 7-day ROAS holds **above 3.5**.
  - Recommend reverting to £12.50 if it falls **below 3.0**.
  - Also re-check `search_budget_lost_impression_share`: if it is now
    near zero the cap is no longer binding and a further increase is not
    warranted; if it is still 15%+ at good ROAS, a second step may be.
  Until 28 Aug, report EUGY's spend and ROAS but do not recommend further
  budget changes — let the test run.

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
  0. **🔴 ALWAYS filter Shopify to the Online Store channel.** Every
     ShopifyQL query MUST carry `WHERE sales_channel = 'Online Store'`.
     Graham caught on 19 Aug that unfiltered totals include the physical
     shop, eBay and other channels — Online Store is only ~59% of
     revenue (POS 26%, eBay 11%). Google Ads drives the online store
     only, so an unfiltered denominator understates tracked share and ad
     cost of sale by roughly 40%, and contaminates AOV with counter
     trade. Three separate conclusions were wrong before this was fixed.
  1. **Ad cost as % of ONLINE STORE revenue — the primary metric.** Total
     non-Glopal ad spend ÷ Online Store revenue, weekly. Needs no
     attribution: Shopify's revenue against the card statement.
     Corrected baseline: **10.54% (5-11 Aug) → 9.95% → 10.44%
     (13-19 Aug)**, i.e. steady around 10%.
     (The earlier 5.27-5.58% series was computed on unfiltered revenue
     and is void.) Report weekly with the week-on-week move. Flag if it
     rises above ~14%.
     Limitation: it measures the whole online business, not incremental
     ad contribution, so it cannot settle individual budget questions.
  2. **Online Store sales** (`FROM sales SHOW orders, total_sales
     TIMESERIES day WHERE sales_channel = 'Online Store'`) as the truth
     source for whether the online business is trading healthily.
  3. **Relative comparison between campaigns** — under-tracking hits all
     campaigns roughly equally, so campaign-vs-campaign ROAS ranking and
     week-on-week direction remain valid even though the absolute level
     does not.
  4. **Non-revenue campaign health**: spend vs budget, CPC, impression
     share, search-term quality, click volume.
  5. **Average order value — ONLINE ONLY.** AOV sets what a click is
     worth, so a lasting drop changes the acceptable cost per
     acquisition. Online AOV runs ~£21-65, averaging ~£45. **The 16-18
     Aug "AOV collapse" was a channel artefact** — blended AOV showed
     £23-26 but online-only was £34.96 / £31.36 / £21.85. Watch
     withdrawn 19 Aug. Only act on a sustained online-only drop below
     ~£30 across a full week.
  Report the **tracked share %** each day as one context line, computed
  against ONLINE STORE revenue. **RESOLVED 19 Aug: tracking has fully
  recovered.** On the corrected denominator the real baseline was 50-65%
  (not 40%), the trough was 12%, and 16-18 Aug ran 65% / 47% / 47% — at
  or above baseline. The three-day test is met and the frozen budget
  decisions are unfrozen. Keep reporting the number, but the incident is
  closed; do not re-litigate it.
  **Superseded note (18 Aug):** an earlier fix here observed that share
  has a moving denominator and that absolute tracked £/day should be
  reported alongside it. Still true and still worth doing — but the
  larger cause was the unfiltered channel denominator, fixed 19 Aug.
  **Keep reporting both the share and the absolute tracked £/day.**
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

- **"Download" in a Loaded Dice listing — INVESTIGATED AND CLOSED
  15 Aug. Not an ads issue; do not re-investigate from the ad account.**
  Graham flagged a listing reading "Download Warhammer 40k 11th Edition
  Starter Sets" on `us.loadeddice.uk`. It is an ORGANIC result, and the
  word is introduced by Glopal's US mirror. Verified: all 344 text assets
  in the account (advertiser + auto-created) contain no "download"; no
  asset targets `us.loadeddice.uk`; there is no US campaign in the
  account. The Shopify source is clean — title tag, meta description and
  article body all free of the word. Open question left with Graham:
  whether Glopal rewrites the tag or Google rewrites Glopal's SERP entry
  (outbound access to the domain is blocked from the analysis
  environment). Full write-up in `reports/2026-08-15.md`.
  - **General lesson: a "Loaded Dice" listing is not automatically our
    ad.** Three surfaces can show our brand and we control only one:
    our ad account, our organic pages, and the Glopal mirrors
    (`us.`/DE/NL/FR), which Glopal rewrites. Check the `Sponsored` label
    and the hostname before treating a screenshot as ad copy.
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
- **PMAX budget — RESOLVED 19 Aug: HOLD at £94.80/day.** On restored
  tracking, matured ROAS 12-17 Aug was **4.14** (£566.32 → £2,344.49)
  with 22.9% impression share. That clears the 2.0 floor and the 3.0
  target but not the 5.0 scale-up bar, and it already spends its budget
  almost exactly (£75-113/day). No further review unless matured ROAS
  leaves the 3.0-5.0 range. Historical note follows:
- **(superseded) PMAX budget — HOLD at £94.80/day, decision suspended 8 Aug.** The
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
