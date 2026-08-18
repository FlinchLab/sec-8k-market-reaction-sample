# Methodology — Historical SEC 8-K Market-Reaction Dataset (by FlinchLab)

**Dataset version:** 0.1.0 · **Methodology version:** 2026-08-14
**Status: APPROVED FOR SALE — approved by the owner on 2026-08-16.**

This document explains, in plain language first, how every number in this
dataset was produced. It is deliberately honest about limitations. Nothing
in this dataset is investment advice, a forecast, or a trading signal; it is
a historical, descriptive research dataset.

## What the dataset measures

When a US public company discloses material news, it must file a Form 8-K
with the SEC. Each filing carries one or more standardized item codes
(earnings, executive departures, material agreements, and so on). This
dataset measures how stock prices historically behaved around those filings:
how much, in which direction, how much of the move happened overnight versus
during the trading session, and how much happened in the days before the
filing was public.

## Coverage

- **Universe**: ~660 US companies drawn from S&P 100 / 400 / 600 constituent
  lists (large / mid / small size buckets), filtered for price ≥ $5 and
  minimum liquidity. The exact universe is rebuilt from live constituent
  tables at each run, so counts shift slightly between runs; every release
  states its run date.
- **Events**: ~29,300 usable 8-K events over a 48-month window (2026-08-14
  run; window 2022-07 → 2026-08). Amended filings (8-K/A) are excluded;
  same-company same-day filings are collapsed into one event carrying the
  union of their item codes.
- **Item codes**: **26 distinct SEC 8-K item codes ship**, from 1.01 through
  9.01 — essentially the whole form, not the handful of codes most event
  studies restrict themselves to. Rare codes are present rather than dropped:
  1.03 (bankruptcy), 3.01 (listing/standard non-compliance), 4.02
  (non-reliance on prior financials) and 5.01 (change in control) all appear.
  **Sample sizes are wildly uneven and the `n_events` column is not
  decoration**: 2.02 carries 11,241 pooled event-day observations while 12 of
  the 26 codes carry fewer than 100, and 1.03 carries one. Breadth here means
  presence, not equal statistical confidence — read every thin code against
  its own `n_events` and `sample_status`, and note that cells under 10 events
  are suppressed entirely.
- **Prices**: daily adjusted OHLC. Overnight ("gap") and regular-session
  components are computed as log returns: gap = ln(open_t / close_{t-1}),
  session = ln(close_t / open_t).

## How abnormal returns are computed

Raw price moves conflate company news with market-wide movement. Each
event's abnormal return (AR) is the stock's return minus what a market model
predicts for it: an ordinary-least-squares fit of the stock's daily returns
against the market (SPY proxy) over trading days −270 to −21 relative to the
event, requiring at least 60 valid observations. Cumulative abnormal returns
(CAR) sum ARs over the event windows in the data:

| window | meaning |
|---|---|
| [−10,−6] | two weeks before the filing (pre-filing baseline) |
| [−5,−1] | the week before the filing (pre-filing) |
| [0] | the event day |
| [0,+1] | event day plus next trading day |
| [0,+5] | the first week |
| [+2,+10] | later drift, after the initial reaction |

The estimation window ends at day −21, so pre-filing windows never
contaminate the model they are measured against.

### Comparing windows: use `mean_abs_scar`, not the raw percentage

The windows above are different lengths, and a longer window accumulates more
variance simply by being longer. So `mean_abs_car_pct` is **not comparable
across windows**: the ~4.8% at [+2,+10] looks like sustained drift beside the
event day, when most of it is nine days of ordinary volatility.

`mean_abs_scar` removes that. It divides each event's window CAR by that
event's own forecast-error standard deviation before averaging, so the number
is in standard deviations rather than percent. Under a no-reaction null it
averages **√(2/π) ≈ 0.798**, whatever the window length.

Read against that benchmark the picture is clear, and it is not the one the
raw percentages suggest. Across every item code with at least 100 events,
[+2,+10] sits at **0.82–1.13×** its own pre-filing baseline — statistically
indistinguishable from it — while the event day sits at **1.6–2.8×**. **The
[+2,+10] column measures baseline volatility, not post-event drift.** Treat a
non-zero figure there as noise unless its own standardised value clears the
baseline. Codes with fewer than 100 events scatter widely in both directions
(0.59 to 2.09) and should not be read at all.

### The pre-filing baseline is not event-free

[−10,−6] is used throughout as the "quiet" comparison window. It is quieter,
but it is not clean: **19.58% of events (5,725 of 29,239) have another 8-K
from the same company accepted inside that window.** Frequent filers are
common, and this is a property of the data rather than a defect in the
measurement — but any comparison against the pre-filing baseline is a
comparison against a window that one event in five shares with another
filing. That biases the baseline *upward*, which makes it a conservative
anchor for reaction claims and an unreliable one for "nothing happens before
filings" claims.

## Statistical testing, in plain language

For every cell (a bucket × item code × window combination) two separate
questions are tested:

1. **Direction** — does the price move a consistent way? (Boehmer-
   Musumeci-Poulsen standardized cross-sectional t-test, with a
   Kolari-Pynnönen adjustment when many events share calendar dates.)
2. **Size** — does the price move *more than noise*, regardless of
   direction? (A Beaver-style test of mean |standardized AR| against what
   pure noise would produce.)

These are different questions with different answers: earnings filings move
prices enormously (size) but in both directions roughly equally, so their
average direction is near zero.

A third, **distribution-free cross-check on direction** ships alongside them
in the `corrado_statistic` and `corrado_p_value` columns. It ranks each
event's event-window return against that same event's own estimation-window
history, so it does not assume returns are normally distributed — useful
precisely where the parametric test is weakest, in thin cells and in the
presence of outliers.

Two things about it you should know before using it. It is a
**staggered-sample adaptation**, not the textbook Corrado (1989) statistic:
event dates here are spread across four years rather than aligned on one
calendar date, so per-event rank statistics are tested cross-sectionally
rather than pooled under Corrado's common-date variance. And its p-value is
**not FDR-corrected**, unlike `direction_q_value` and `magnitude_q_value` in
the same file. Read it as agreement or disagreement with the BMP result, not
as an independent significance verdict, and do not compare it to 0.05 as
though it had been adjusted.

Because thousands of cells are tested at once, some would look significant
by pure chance. All q-values are Benjamini-Hochberg false-discovery-rate
adjusted at q = 0.10, in **separate correction families** for direction vs
size and for reaction windows vs pre-filing windows. A q-value is only
comparable to q-values in its own family (`fdr_family` column).

## Out-of-sample validation (validated_findings.csv)

FDR control still allows ~10% false discoveries, and any finding located by
searching data can be an artifact of the search. Findings in
`validated_findings.csv` therefore passed a stricter protocol borrowed from
clinical-trials practice:

1. **Discovery** on one slice of the data only.
2. **Pre-registration**: significant findings frozen into a fixed list.
3. **Holdout**: only that frozen list is tested on data the discovery step
   never saw.

This runs on two independent axes — chronological (earliest 75% discovers,
newest 25% tests: does the finding *persist*?) and by-company (no company on
both sides: does it *generalize*?). Only findings that replicated on BOTH
axes appear in the file. Failed and refuted findings are retained in the
research record; an example: an early "CEO departure without successor
causes −8%+ reactions" finding was refuted in holdout as a small-sample
artifact and is reported as such.

### Why holdout effects in this file look *larger* than discovery effects

In `validated_findings.csv` the holdout effect exceeds the discovery effect
in **86.5% of rows**, with a median holdout/discovery ratio of **1.485**.

**This is not evidence that the effects grew, and it should not be read as
strength.** It is survivorship, and the mechanism is arithmetic. Across all
154 hypotheses that were actually tested, the median ratio is **0.461** —
effects shrink out of sample, exactly as the winner's curse predicts, because
a finding is selected for having been large in the discovery slice and part
of that size was luck. Among the 36 that replicated the median is **1.638**.
The file contains only the replicated ones. Conditioning on "replicated" is
what moves the number above 1, not anything about the market.

The selection is sharper than it first looks: holdout slices are roughly a
third the size of discovery slices (mean n 1,578 vs 4,764), so clearing
significance in the holdout mechanically requires a *large* holdout estimate.
Findings that shrank the normal amount did not survive to be published here.

We tested the alternative explanation — that the newest quarter of the sample
was simply a more volatile market regime — and it does not carry the result:
standardising absorbs only about half of a pooled 1.13× gap, leaving ~6–13%
attributable to volatility against ~90% to selection. There is no step change
in volatility at the split boundary.

**What to take from the holdout columns:** they establish that the *direction
and existence* of an effect survived data the discovery step never saw. They
do **not** establish its magnitude. For an unbiased size estimate, the
discovery figure is the conservative one, and the truth is plausibly below
both.

## Sub-type categories

The official SEC item codes are coarse: one code averages together news
types with opposite price effects. Filing texts were additionally classified
into finer categories (for example, earnings releases that *lower guidance*
versus *raise guidance*), each classification independently verified for
structural validity and benchmarked against an independent second
classifier. The bundled earnings code's average day-0 move is roughly
−0.15%; splitting it exposes a spread of several percentage points between
guidance-lowered and guidance-raised filings. Only sub-type findings that
survived the full two-axis validation appear in this dataset.

## Small samples

Cells with fewer than 10 events carry `sample_status = too_few` and blank
statistics. A blank never means "no effect"; it means "not enough data to
say". Validation additionally treats holdout cells with fewer than 30
events as UNTESTABLE rather than failed.

## Known limitations

- **Survivorship bias**: the universe is built from *current* index
  constituents, so companies that failed or were delisted are
  under-represented, more so further back in time.
- **Counts shift between runs**: the universe rebuilds from live constituent
  lists; replication counts are sample-dependent. Effect sizes are the
  stable quantity; counts are always quoted with a run date.
- **Daily resolution**: intraday dynamics beyond the overnight/session split
  are not measured.
- **US single market**, 48-month window as of this version.
- **No consensus-expectations data**: earnings sub-types describe what the
  filing text asserts, not beats/misses versus analyst expectations.
- **Window overlap**: for frequent filers, one event's later windows can
  overlap another event's early windows; this is disclosed, not corrected.
- **Classification error**: label accuracy was benchmarked by independent
  re-labelling (agreement 79–88%, Cohen's kappa 0.68–0.82 on the fields used
  in published findings). Labels with weak agreement are excluded from
  published findings.
- **Filing-time buckets do not sum to the event total.** The `t:pre_open`,
  `t:next_open` and `t:intraday` slices cover 28,947 events, not all 29,331.
  The 384 missing events are not errors and not missing timestamps — every
  filing carries an EDGAR `acceptanceDateTime`. They are event days on which
  one company filed two or more 8-Ks that fall in *different* time buckets,
  so no single bucket describes them. They are labelled `mixed` and excluded
  from filing-time slices rather than assigned arbitrarily. They remain in
  every non-timing cell.
- **The overnight/session components do not sum to the daily CAR.** Each
  component subtracts the same daily market-model intercept α that the daily
  figure already subtracts once, so by construction
  `CAR_gap + CAR_session = CAR_daily − n·α` over an n-day window. The
  **totals are unaffected** — `mean_car_pct` and `mean_abs_car_pct` are
  computed on the daily path and carry no such term — but the two signed
  component columns are each biased by a per-day constant, typically 5–8
  basis points per day. Use the components for *relative* comparisons
  (overnight vs session, one category vs another), which are unaffected; do
  not treat their sum as a decomposition of the total. This is disclosed
  rather than corrected in this version, because correcting it changes
  published numbers and therefore requires a new dataset version.
- **`overnight_share` is a share of component magnitudes, not of the day's
  move.** In `curated_mechanism.csv` it is `|gap| / (|gap| + |session|)`.
  Absolute values are deliberate — signed components cancel and would
  understate both halves — but the denominator is consequently about 1.37×
  the mean |CAR|, so the figure is not "what fraction of the move happened
  overnight". Read it as the overnight fraction of combined component
  magnitudes, and compare it against its own baseline rather than against
  50%: pooled across all events it is **0.4929** on the event day versus
  **0.3602** in the pre-filing baseline window — about 13 points higher when
  news actually arrives.

## Provenance

Source filings: SEC EDGAR (public). Prices: public market-data APIs. All
derived statistics computed by a pipeline whose full test suite (250+ tests,
including planted-effect recovery and no-false-positive gates on synthetic
data) must pass before any output is written. SEC source material remains
public; what this dataset provides is the derived, documented, validated
research layer on top of it.
