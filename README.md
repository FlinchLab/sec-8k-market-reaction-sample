# Historical SEC 8-K Market-Reaction Dataset — by FlinchLab

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.21986316.svg)](https://doi.org/10.5281/zenodo.21986316)

**Know what each kind of company news historically did to the stock —
before you build on top of it.** 29,331 SEC 8-K filings from 660 US
companies, read and re-classified finer than the official item codes, with
each category's price reaction measured by a market-model event study and
every published finding validated on two independent out-of-sample
holdouts.

**Download the free layer** (this repo): [`preview.csv`](preview.csv) —
8 rows, the complete column schema, real figures ·
[`data_dictionary.csv`](data_dictionary.csv) — every field defined ·
[`methodology.md`](methodology.md) — the full method and its limitations.
The complete dataset (1,668 aggregate cells + 104 validated findings, run
2026-08-14) is a one-time **$99** release at
**<https://flinchlab.com/dataset>**. Nothing else is here by design — see
[`NOTICE.md`](NOTICE.md).

## Why this exists when EDGAR is free

SEC EDGAR provides raw filings — public, unstructured, one at a time. This
dataset provides the research layer built on top: ~29,300 8-K events
(~660 US companies, 48 months, run 2026-08-14) normalized, aligned to
market-return windows, converted to market-model abnormal returns,
statistically tested with explicit multiple-testing control, and validated
out-of-sample on two independent axes (chronological persistence and
cross-company generalization). Findings that failed validation are
disclosed as refuted, not deleted.

One example of what the derived layer shows (both files document it): the
official "results of operations" item code averages a −0.15% same-day
abnormal return — near nothing. Splitting the same filings by what the
release says about forward guidance exposes a spread of several percentage
points between guidance-lowered (−5.2% discovery, −6.5%/−5.3% in the two
holdouts) and guidance-raised (+2 to +3%) filings. The official taxonomy
was averaging opposite reactions into mush; the derived taxonomy separates
them, with out-of-sample replication.

## Reading the numbers

- Columns ending `_pct` are percentage points: `-0.09` means −0.09%.
- `sample_status = too_few` (n < 10) rows carry blank statistics; blank
  means "insufficient data", never "zero effect".
- q-values are Benjamini-Hochberg adjusted (q = 0.10) and only comparable
  within the same `fdr_family`.

## Method in one paragraph

Per event: OLS market model on trading days −270..−21 (min 60 obs) →
abnormal returns → CARs over six windows from [−10,−6] to [+2,+10], with
overnight/session decomposition. Tests: BMP standardized cross-sectional t
(direction; Kolari-Pynnönen-adjusted under date clustering), Corrado rank,
and a Beaver-style magnitude test — direction and magnitude corrected as
separate FDR families, pre-filing windows as a third. Validated findings
additionally passed discovery → pre-registration → holdout on two
independent axes. Full details, including all limitations (survivorship
bias, daily resolution, US-only, run-to-run count drift), in
[`methodology.md`](methodology.md).

## Honest caveats (summary — full list in methodology.md)

Survivorship bias toward current index members · daily resolution · single
market (US) · 48-month window · sub-type labels describe what filings
assert, not beats/misses vs expectations · universe rebuilds per run, so
counts are quoted with run dates. Historical and descriptive research
data: not investment advice, not a forecast, and not a trading signal.

## Explore the findings online

Every published finding is a free page with its figures, holdout results
and caveats:

- [All questions the dataset answers](https://flinchlab.com/insights)
- [What happens when a company lowers its guidance?](https://flinchlab.com/answers/stock-reaction-guidance-lowered)
- [How does the market react to a CEO succession?](https://flinchlab.com/answers/stock-reaction-ceo-succession)
- [The full results table, sliceable](https://flinchlab.com)

## Cite this

```bibtex
@misc{flinchlab2026sec8k,
  title   = {Historical SEC 8-K Market-Reaction Dataset},
  author  = {{FlinchLab}},
  year    = {2026},
  doi     = {10.5281/zenodo.21986316},
  url     = {https://flinchlab.com/dataset},
  note    = {Version 0.1.1, run dated 2026-08-14. Quote figures with the run date.}
}
```

The free layer is also on
[Hugging Face](https://huggingface.co/datasets/Flinchlab/sec-8k-market-reaction-dataset)
(interactive table) and archived on
[Zenodo](https://doi.org/10.5281/zenodo.21986316) (DOI).

## The complete dataset

- Full release: **$99 one-time**, via <https://flinchlab.com/dataset> —
  the canonical home of the dataset. This repository is a pointer to it.
- Methodology page: <https://flinchlab.com/methodology>
- Samples, invoices, institutional terms, or questions: **data@flinchlab.com**

Licence for the complete dataset: see [`NOTICE.md`](NOTICE.md).

## Who makes this

Mustafa. I designed the requirements and the honesty rules; AI implemented
them; everything is reproducible from the methodology page at
<https://flinchlab.com>.

## Provenance and versioning

Source filings: SEC EDGAR (public). Prices: public market-data APIs.
Dataset version 0.1.1 · methodology version 2026-08-14 · produced by
a pipeline gated on a 250+-test suite including planted-effect recovery and
no-false-positive controls on synthetic data. This is a bounded historical
release; no update schedule is promised.
