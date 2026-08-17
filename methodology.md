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

## Provenance

Source filings: SEC EDGAR (public). Prices: public market-data APIs. All
derived statistics computed by a pipeline whose full test suite (250+ tests,
including planted-effect recovery and no-false-positive gates on synthetic
data) must pass before any output is written. SEC source material remains
public; what this dataset provides is the derived, documented, validated
research layer on top of it.
