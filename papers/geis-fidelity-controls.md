# Fidelity Controls for a Quantitative Equity Screener, and What They Do Not Prove

Marc Romero, independent researcher  
Working paper, version 1.2, September 2026

---

## Abstract

Public quantitative equity work tends to run together two properties that are not the same thing. One is whether a system's output faithfully reflects its inputs. The other is whether that output predicts returns. The first can be established by construction. The second is an empirical claim and needs evidence.

This paper describes a reconciliation control built to establish the first property for a 14-factor equity screener covering roughly 805 listings. The control rebuilds every stored ratio, sub-score, composite and cross-sectional percentile from raw data and compares them against what the system wrote, currently with zero discrepancy across 5,437 company-quarters. I then document three defects the control surfaced. Each produced output that was plausible and wrong, and none of them raised an exception.

The last part of the paper reports a first exploratory measurement of predictive content: a one-month rank information coefficient of approximately zero for the composite, with strongly negative coefficients on both momentum factors. I argue that this measurement does not justify revising the model, and I spend some time on why, because the temptation to act on it is the interesting part. The sample is 14 scoring dates inside one month, with overlapping return windows and a single market regime. The negative momentum coefficients are also what the short-term reversal literature predicts at a one-month horizon, which makes them close to uninformative about the factor itself.

The contribution is the order of operations: make the system auditable first, measure second, and then leave the first measurement alone instead of fitting to it.

**Keywords:** equity screening, data fidelity, reconciliation, information coefficient, short-term reversal, backtest overfitting, reproducibility

---

## 1. Introduction

There are two different ways a quantitative equity system can be wrong, and they have almost nothing to do with each other.

The first is a fidelity problem. The output does not follow from the inputs, because of a unit error, a currency mismatch, a stale read, or a definition that differs between two code paths. That is an engineering defect and you can detect it by reconstruction: recompute everything from the raw data and see whether the numbers match.

The second is not a defect at all. The system says exactly what it means, faithfully and reproducibly, and what it means turns out to have no predictive content. That is the normal condition of most factor models, and the only way to find out is to measure against realised returns.

You can have either without the other. A system can be perfectly faithful and completely uninformative, which is what Sections 6 and 7 describe. It can also be informative and quietly unfaithful, in which case whatever performance it shows is an artifact of a bug that happens to correlate with something, and the model you think you have is not the model you are running.

There is a large literature on the second problem. On the first, as it applies to a system that actually runs rather than to an idealised specification, there is much less, and the gap is worth filling: reconciliation against raw data is where the defects that survive a test suite are actually caught. This paper documents that work on one system, and shows that getting fidelity right licenses no conclusion at all about skill.

The system under study is GEIS, a daily batch screener over approximately 805 listings, scoring each on 14 criteria drawn from O'Neil (2009), a fundamental quality block after Sloan (1996) and Novy-Marx (2013), and a valuation block in the spirit of Greenblatt (2006). Its full specification is documented separately (Romero, 2026); this paper assumes only the outline given in Section 2.

Section 3 describes the reconciliation control. Section 4 reports its results. Section 5 presents three defects it surfaced, each as a short case. Section 6 describes the information-coefficient measurement and Section 7 its results. Section 8 argues why those results should not be acted on, and is the section I would keep if I had to cut the rest. Section 9 sets out the scope of the evidence and Section 10 what follows it.

---

## 2. The system under study

GEIS scores ~805 listings daily on 14 sub-scores in three blocks: seven CAN SLIM criteria (52% of weight), six fundamental quality measures (30%), and one valuation measure (18%). Sub-scores are bounded to [0, 100] and combined as a weighted mean. The target horizon is a swing holding period of 2–12 weeks.

Two properties of the design matter for everything that follows.

Missing inputs are excluded rather than imputed. Each sub-score returns no value when its input is absent; the composite averages only present factors and renormalises their weights. Because base weights sum to exactly 1.0, the renormalisation denominator *is* the fraction of model weight backed by data. A minimum coverage floor prevents companies with very sparse data from receiving a rescaled single-factor number that resembles a full score.

Every score is persisted with its date. A run writes a dated snapshot rather than overwriting state. This is what makes reconstruction possible at all, and it is the precondition for everything in Section 3.

Full specification, including factor definitions, weights, sector overrides and coverage floors, is in the companion technical note (Romero, 2026).

---

## 3. Method: reconciliation as a gate

The control is a reconciliation that runs on every execution of the pipeline and fails the run on breach, returning a non-zero exit status so that a scheduler marks the run as failed rather than silently succeeding.

It operates in three layers, each recomputing from the most raw available representation:

**Layer 0, the cross-sectional percentile.** The stored 12-month relative-strength percentile is recomputed from the price history and compared on three quantities: both anchor points, the derived return, and the resulting percentile. Tolerance is zero. A date with no persisted percentile fails the run rather than raising a warning, because a factor whose input cannot be audited is not auditable even when its value happens to be right.

**Layer 1, the ratios.** Every fundamental ratio is recomputed from the stored statement lines and the stored price, and compared against what the scorer used.

**Layer 2, the sub-scores and the composite.** All 14 sub-scores and the composite are recomputed from the layer-1 ratios and compared against the persisted row.

Alongside the three layers, the gate enforces structural invariants: universe coverage above a floor, per-factor null rates below a ceiling, uniqueness of the market-health reading per date, no duplicate company-date rows, no stale prices among the top-ranked names, and no orphaned forward-return rows.

Two design choices here are worth stating, because neither is obvious until the control is actually run against real history.

No exemption list. An earlier version maintained a set of sub-scores excused from the reproducibility requirement, which in practice meant the two factors that did not reproduce. That set is now empty, and deliberately retained as an empty structure rather than deleted, so that any future exemption must appear in a diff rather than as an absence.

Reconciling the past means reconciling under the past's assumptions. Historical rows were written under a 60-day reporting lag: the assumption about which quarterly filings were public as of a given date. Auditing those rows under a different lag produces discrepancies of around 9% on the composite that are **not** infidelity, they are a different question about what was knowable. The distinction is now a documented operating rule: the audit adopts the parameters under which the data was written.

A second-order version of the same error: if a live re-score is executed *before* the historical re-score rather than after, the most recent date is written with the historical lag and then reconciled against live accessors, producing the same spurious ~9% discrepancy. Order of operations is part of the method.

---

## 4. Results: fidelity

As of 13 September 2026:

| Check | Result |
|---|---|
| Layer 0, relative strength reproducibility | 0 discrepancies |
| Layer 1, ratios | 0.00% |
| Layer 2, sub-scores and composite | 0.00% |
| Company-quarters reconciled | 5,437 |
| Companies scored | 774 / 805 (96.1%) |
| Cross-sectional percentile stability, adjacent dates | max 4.9 points (threshold 10) |
| Full gate | **PASS, 0 warnings** |
| Test suite | 261 passing |

The stability check in the penultimate row deserves a note, because it tests something the reproducibility check cannot. A percentile can be perfectly reproducible from stored data and still move erratically if the *definition* is unstable. The check asserts that a company whose own 12-month return did not move (by less than one point) cannot shift more than 10 percentile points between adjacent dates. Measured maximum after the rebuild described in Section 5.4: 4.9 points.

Before that rebuild the same measurement produced a median discrepancy of 3.5 points, a 90th percentile near 17, and a worst case of 71. One company moved from the 74th to the 16th percentile in a single week without any corresponding change in its own return.

What this establishes: Every number the system publishes is a faithful function of the data it stored. Any figure can be recomputed by a third party holding the same database and the same code, and it will match to the last decimal.

What it does not establish is whether the ranking predicts anything. Fidelity is a precondition for doing research, not evidence of skill, and the rest of the paper is about that second question.

---

## 5. Three defects, and how they were found

Each of the following produced output that was plausible and wrong. None raised an exception. All three were found and corrected during the audit described above, before the public record began. I found them by looking at the distribution of a sub-score across the universe rather than by reading code, and that is the methodological finding of this section: a defect that survives a test suite is usually visible in a histogram.

### 5.1 Two currencies in one subtraction

Enterprise value is computed as market capitalisation plus total debt minus cash. Market capitalisation derives from price and is denominated in the **trading** currency. Debt and cash derive from financial statements and are denominated in the **reporting** currency.

For a domestic US issuer these coincide and the expression is correct. For an ADR or foreign issuer they do not: a company reporting in Taiwan dollars and trading in US dollars, or reporting in Swedish kronor, Danish kroner, or Chinese renminbi while trading in US dollars. The subtraction then mixes units.

Nothing raises an exception. What you get is a plausible number.

What I noticed was four foreign issuers sitting at exactly 100 on the valuation factor, all looking like extraordinary bargains. The obvious explanation is saturation: a linear scoring band topping out at genuinely cheap valuations. That happens in this model and it is not a bug. So my first instinct was to widen the band.

What did not fit that explanation was a fifth name, which produced a negative enterprise value, returned no score for the factor, and dropped out of it with no signal that anything had happened. Saturation does not do that. Something was wrong with the denominator itself, and once you look at the denominator with the currencies in mind it is obvious.

This was the heaviest-weighted factor in the model, at 18%.

The fix converts the financial leg through a retrieved exchange rate, keying on the reporting currency recorded for the issuer, and where no rate is available it returns no value instead of estimating one. That second half is the part that matters. Afterwards the same four names score 81.3, 80.3, 61.0 and 18.8 against a prior uniform 100, and the factor resolves into 619 distinct values across 753 companies, which makes it one of the most discriminating things in the model.

### 5.2 A flow measured against a stock

Return on invested capital and return on equity were computed from a single quarter's earnings against a balance-sheet stock. The correct construction is a trailing-twelve-month flow against the same stock.

The error understated returns on capital by roughly a factor of four. One large-capitalisation technology company read 9.8% where its trailing-twelve-month figure is 30.2%; another read 47.7% against 119.9%; a consumer-staples name read 7.1% against 39.6%.

This is harder to catch than the currency defect, because uniformly low numbers always have an innocent explanation available: strict thresholds. The quality ranking sat compressed toward the bottom of its range, which is equally consistent with a demanding model and with a broken one.

The downstream effect was larger than the direct one. A secondary classifier assigns each company a profile, and its compounder rule tests return on capital against a threshold of 20%. Reading the quarterly figure meant that rule was, in practice, testing for roughly 80% annualised. Correcting the read raised the count of companies classified as compounders from 9 to 54. A sixfold change in a categorical output, all of it from a unit error upstream.

The part worth generalising is that a wrong unit does not just shift a number. It quietly moves every threshold defined against that number, and those thresholds do not announce that they have moved.

### 5.3 Renormalisation concealing an inert rebalance

The weight on quarterly earnings growth was reduced from 0.15 to 0.05 in a deliberate rebalance away from trailing growth. Sector-level overrides, written earlier and not revisited, continued to pin that weight at 0.18 for the largest sector in the universe.

Because overrides are renormalised back to a unit sum, the effective weight came out at 0.1488, roughly three times the new base, and everything still summed to 1.0. No assertion fired. No total looked wrong, because no total was wrong.

Reading the configuration does not help here either. The base weight says 0.05 and the override says 0.18, and both are doing exactly what they say. The failure only becomes visible if you compute the effective weight per sector after renormalisation, which is not an obvious thing to check, because the arithmetic is correct at every step.

For 52% of the scored universe, the rebalance had not happened at all. In the affected sector, the combined weight on the two growth factors stood at 0.2645, almost exactly the concentration the rebalance had been designed to remove.

This is the most consequential defect of the three and the hardest to see, because renormalisation is a *correct* mechanism whose correctness is precisely what hides the failure. A quantity that always sums to one offers no signal that one of its components never changed.

The control now in place is an assertion that fails when an override equals a base weight that has been superseded. It checks intent rather than arithmetic, which is unusual for an assertion but is what the failure required.

### 5.4 A fourth case: one concept, two definitions

Not a defect in output so much as in construction, and the reason Layer 0 of the gate exists.

Relative strength never reproduced. The cause was two definitions of "12-month return" living in two code paths: the live daily run anchored on a fixed number of **rows** of price history; the historical re-score anchored on 365 **calendar days**. Both are defensible. Holding both is not.

The row-anchored definition had a further property that made it irreproducible in principle rather than merely in practice: the window shifted whenever the price history gained or repaired a row, so the same company on the same date could yield different answers depending on when the question was asked. A tie-breaking rule dependent on row arrival order compounded it.

The rebuild put the definition in a single module used by all three call sites: the live run, the historical re-score and the backfill. The far anchor is the close nearest to D − 365 calendar days, ties resolving to the earlier close: a fixed rule, deliberately not one contingent on data arrival. Tolerance is 21 calendar days at each end; outside it there is no relative strength and the factor is excluded under the standard missing-data policy. Each percentile is persisted with both anchors, price and date at each end, so the figure is auditable rather than merely assertable.

A definition existing in two places is not one definition. Reproducibility failures are frequently definitional rather than arithmetic, and this one is a clear instance.

---

## 6. Method: information coefficient

Once fidelity is established, the second question can be asked.

The information coefficient is the Spearman rank correlation between a score observed at date D and the realised return over a subsequent window. It is computed per date and then averaged with weights proportional to sample size, which is to say walk-forward, rather than pooling all observations into a single correlation. Pooling would allow a few dates with large cross-sections to dominate, and would obscure regime dependence.

Rank correlation is used rather than linear correlation because the sub-scores are bounded with clamps and bonuses: their scale is neither linear nor comparable across factors, while their ordering is. Where a date yields fewer than a minimum number of paired observations, the coefficient returns no value rather than a number, the same policy the scoring layer applies to missing inputs.

Forward returns are computed from the same price history that fed the scores, at horizons of one, three and six months with proportionate tolerances. Using the same price source is deliberate: the return is measured with the prices that produced the score. The cost is that operational gaps in the price history are also gaps in the measurement.

### 6.1 Reconstructed history, and why it is not a record

The historical scores used here were produced by re-scoring the past with current code under a 60-day reporting lag, chosen to cover the statutory filing deadline without discarding data that was in fact public.

This removes coarse look-ahead. It does not remove three residual biases, and I regard the distinction as decisive:

1. **Restatement look-ahead, irreducible.** The fundamentals store retains only the current version of each quarter. Where a company has restated, the reconstruction uses corrected figures that nobody held at the time. Eliminating this requires a point-in-time archive of financial statements, which does not exist here.
2. **Current exchange rates applied to past dates.** The valuation factor requests live rates; re-scoring an earlier date applies a later rate. This affects only the 61 dual-currency listings, and the magnitude over two months is small, but it is look-ahead.
3. **Stale market-health readings on six dates**, where no contemporaneous reading existed and the most recent prior one was carried forward, in one case by 29 days. Since that factor is uniform across the universe, it shifts the level without affecting the cross-section, but it should be known when reading those dates.

So the reconstructed history is a research artifact and not a track record. It is not labelled as picks, not presented as realised performance, and not published as a historical result. The public record is forward-only, beginning at first publication, with zero look-ahead by construction because there is nothing to reconstruct.

---

## 7. Results: first measurement

**Sample: 14 scoring dates, 9 July – 8 August 2026, approximately 730 paired observations per date, one-month horizon.**

| Factor | IC (1m) |
|---|---:|
| Relative strength (L) | **−0.2293** |
| Price momentum (N) | **−0.1809** |
| Debt safety | +0.1154 |
| FCF | +0.0783 |
| Valuation | +0.0684 |
| **Composite** | **−0.0234** |

Two things about this measurement before reading anything into it. It sits at a one-month horizon, where the documented effect on relative strength is reversal rather than momentum, and it rests on 14 dates inside a single month. Section 8 takes both apart.

Read at face value: over this window the two momentum factors, which are the core of the CAN SLIM half of the model, ordered companies in the wrong direction, and on large cross-sections. The quality and safety factors ordered them correctly, modestly. The composite came out at approximately zero because the two halves of the model cancelled each other.

Market-health across the window stood near 79 on a 0–100 scale, i.e. a uniformly favourable regime. Three- and six-month horizons have no computable sample: the first score is dated 9 July 2026, so the earliest three-month return becomes available on approximately 7 October 2026 and the earliest six-month return in approximately January 2027. That is arithmetic, not a deficiency.

---

## 8. Why these results do not support revising the model

The conclusion of this section is negative, which is the point of including it.

Not a single weight has been changed in response to the results above. There are five reasons and any one of them would be enough.

### 8.1 The sample is approximately one observation

The 14 dates span 9 July to 8 August, which is one month. The return windows are 30 days and therefore overlap almost entirely. Adjacent dates share nearly all of their return period, so these are not 14 independent observations; in terms of effective sample size they approximate one. López de Prado (2018) treats this directly as the problem of sample uniqueness under overlapping outcomes, and the standard correction, weighting observations by uniqueness, would here reduce the effective sample to near unity.

A single month of mean reversion produces a momentum coefficient of −0.23 without carrying any information about whether momentum works.

### 8.2 The negative momentum result is what the literature predicts

This is the part that most changes how the numbers should be read, and it cuts against over-interpreting my own result.

The momentum literature (Jegadeesh & Titman, 1993) documents positive returns to relative strength at horizons of three to twelve months. At one month the documented effect has the opposite sign: short-term reversal, established independently by Jegadeesh (1990) and Lehmann (1990).

The measurement here is at one month. A negative one-month coefficient on relative strength is therefore the expected result under the standard literature, not evidence against the factor. Had the coefficient come out strongly positive at this horizon, that would have been the anomalous finding requiring explanation.

The system targets a 2–12 week horizon. One month sits at the bottom edge of that band, in precisely the region where the reversal effect dominates and the momentum effect has not yet asserted itself. The measurement sits at the wrong horizon for the question, and fixing that means waiting for data rather than changing the model.

### 8.3 A single regime

All 14 dates fall in one favourable market regime. Whatever the coefficients describe, it is not behaviour in a drawdown, which is the regime in which a momentum-weighted model would be expected to fail differently and in which any conclusion would matter most.

### 8.4 Survivorship

The universe is the present universe. Companies delisted or removed are absent from the historical scores, inflating every backward-looking statistic above by an unmeasured amount.

### 8.5 Fitting to this would be textbook overfitting

Revising weights against a single month of a single regime, at the wrong horizon, on an effectively unitary sample, is data-snooping in the sense Harvey, Liu and Zhu (2016) and Bailey, Borwein, López de Prado and Zhu (2014) describe, and it would be self-inflicted, since the same data would then serve as both the basis for the revision and the evidence for it.

### 8.6 The defensible conclusion

What this exercise establishes is that the measuring instrument now exists. It does not establish that momentum fails, that the model is miscalibrated, or that the composite is worthless.

If the weights are ever revised, three conditions apply: the revision is walk-forward, on data the model has not seen; it is published before the period over which it is to be assessed; and the pre-revision specification stays on the public record. Those conditions are stated here so that departing from them later is visible.

One more thing, since it is the part I found hardest in practice. Generating a result is not the difficult step. Not acting on the first result that arrives is, especially when it is your own system and the result is interesting. Section 7 is the most interesting output this project has produced so far, and the right response to it is to change nothing and wait until October.

---

## 9. Scope of the evidence

**What the system does.** GEIS evaluates companies against a fixed set of criteria that are published in full and reproducible from the stored data, and reports where each company falls in that ranking. It is not a return predictor. Section 4 establishes the first property; nothing in this paper establishes the second, and Section 8 sets out why the measurement in Section 7 does not speak to it in either direction. That distinction is the limitation to carry away from this paper. The notes below bound the Section 7 measurement, not the system.

**Horizon.** The target horizon is 2–12 weeks. The only horizon computable so far is one month, which sits at the bottom edge of that band and inside the region where short-term reversal dominates. Per Section 8.2, this is a property of when the series began rather than of the model, and it resolves with elapsed time rather than with any change to the system.

**Continuity of the sample.** The scheduled run fires when the machine is powered, and a score for a date that was not run cannot be rebuilt afterwards, since it depends on the fundamentals as they stood. This bounds the number of dates available to Section 7. The published record is weekly for exactly this reason and is unaffected by it.

**The portfolio question.** An information coefficient describes whether an ordering carries information, not whether a top-N selection outperforms a benchmark. Those are different questions with different designs, and the second is set out as item 4 of Section 10.

**What the universe is.** The universe is constructed around one economic theme rather than sampled from the market, so a result inside it is a result about that theme. This is a statement of what the ranking ranks, not a defect in the ranking.

---

## 10. Further work

1. **Measure at three months** (available from approximately October 2026) and six months (approximately January 2027). This is the horizon the system targets and the only one at which Section 7 becomes interpretable.
2. **Accumulate dates across regimes.** Any conclusion drawn within a single favourable regime is a conclusion about that regime.
3. **Move the forward-return computation into the daily pipeline**, so the measurement series is maintained by the same process that generates the scores.
4. **Portfolio-level test** against a broad and a technology-weighted benchmark, to address the question the information coefficient cannot.
5. **Turnover and rank-persistence statistics**, to characterise how the ordering behaves across the horizon the system targets.
6. **Sensitivity analysis on the thresholds.** Measuring how far the ranking moves as the coverage floors, the ranking floor and the valuation saturation bands are varied, to put numbers under parameters currently set by reasoning.
7. **Point-in-time fundamentals archive**, the condition for removing the restatement look-ahead described in Section 6.1.

Publication of the forward record began in September 2026 and is weekly, benchmarked, and without post-hoc selection, under the protocol set out in the companion note (Romero, 2026).

---

## Disclosures

- **Author and capacity.** Marc Romero. Independent individual researcher. Not an investment firm, an investment adviser, an analyst at a regulated entity, or a fund manager; holding no licence to provide investment advice.
- **Nature of the content.** This is methodological research. It is not investment advice, not a personal recommendation, and not a solicitation.
- **No performance claims.** No claim, express or implied, regarding past or future returns. The system ranks companies against disclosed criteria and does not forecast prices; no edge has been demonstrated or is claimed.
- **Conflicts of interest.** None relating to any security named. Any position held is disclosed at the point of publication, as is any net position exceeding 5% of a company's capital.
- **Compensation.** No sponsorship or compensation from any issuer.
- **Risk.** Equity investment carries risk of total loss. Nothing here accounts for any individual's circumstances, objectives or risk tolerance.
- **Data.** All figures derive from a commercial market data provider accessed through a public library.

---

## References

Bailey, D. H., Borwein, J. M., López de Prado, M., & Zhu, Q. J. (2014). Pseudo-mathematics and financial charlatanism: The effects of backtest overfitting on out-of-sample performance. *Notices of the American Mathematical Society*, 61(5), 458–471.

Cooper, M. J., Gulen, H., & Schill, M. J. (2008). Asset growth and the cross-section of stock returns. *Journal of Finance*, 63(4), 1609–1651.

Greenblatt, J. (2006). *The little book that beats the market*. Wiley.

Grinold, R. C., & Kahn, R. N. (2000). *Active portfolio management* (2nd ed.). McGraw-Hill.

Harvey, C. R., Liu, Y., & Zhu, H. (2016). …and the cross-section of expected returns. *Review of Financial Studies*, 29(1), 5–68.

Jegadeesh, N. (1990). Evidence of predictable behavior of security returns. *Journal of Finance*, 45(3), 881–898.

Jegadeesh, N., & Titman, S. (1993). Returns to buying winners and selling losers: Implications for stock market efficiency. *Journal of Finance*, 48(1), 65–91.

Lehmann, B. N. (1990). Fads, martingales, and market efficiency. *Quarterly Journal of Economics*, 105(1), 1–28.

López de Prado, M. (2018). *Advances in financial machine learning*. Wiley.

Novy-Marx, R. (2013). The other side of value: The gross profitability premium. *Journal of Financial Economics*, 108(1), 1–28.

O'Neil, W. J. (2009). *How to make money in stocks* (4th ed.). McGraw-Hill.

Romero, M. (2026). *GEIS: System description. A technical note on the design of a quantitative equity screener*. Version 1.2.

Sloan, R. G. (1996). Do stock prices fully reflect information in accruals and cash flows about future earnings? *The Accounting Review*, 71(3), 289–315.

---

v1.2, 17 September 2026. Results as of 13 September 2026. Editorial revision; no change to method, parameters or figures from v1.1. A revision reporting the three-month horizon is expected once that sample becomes available.
