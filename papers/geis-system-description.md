# GEIS: System Description

### A technical note on the design of a quantitative equity screener

Marc Romero, independent researcher  
Version 1.0, 27 September 2026

*This is a system description, not a research paper. It covers what GEIS measures, how those measurements combine, and why I made the calls I made. The empirical work, the fidelity controls and the first measurement of predictive content, sits in a separate document, "Fidelity Controls for a Quantitative Equity Screener, and What They Do Not Prove". That one was written against an earlier build of this system and says so at the top.*

---

## 1. What this is

GEIS runs once a day over about 800 listed companies. It scores each one on 13 criteria, combines them into a single number between 0 and 100, and sorts.

The question it answers: given a fixed set of criteria that I've written down and can reproduce, which companies in my universe come out on top today?

What it does not do is forecast prices, and that distinction is the one thing to carry out of this document. The two questions get run together constantly and they need completely different kinds of evidence. Whether a system reproduces its own output exactly is an engineering property, and one you can establish by construction. Whether its ranking anticipates returns is an empirical claim that needs realised returns, at the right horizon, across more than one regime. This system is built to answer the first question and to make the second one measurable. It is not built to answer the second, and nothing here claims that it does. The companion document works through what the evidence available so far will and will not support.

Nothing here is investment advice, a recommendation, or a solicitation. Section 9 spells that out.

---

## 2. Universe, horizon and cadence

| Property | Value |
|---|---|
| Universe | ~805 active listings: an AI-ecosystem construction across 20 thematic layers, plus the S&P 500 |
| Scored (25 September 2026) | 778 / 805 (96.6%), of which 123 above the ranking floor |
| Frequency | Daily batch, weekdays, after the US close |
| Target holding horizon | Swing, 2 to 12 weeks, with fundamental backing |
| Persistence | Local SQLite, one dated snapshot per run |
| Runtime | About 12 minutes for a full pass |
| Test suite | 285 tests |

### 2.1 About the universe

I built the universe rather than sampling one. It covers the segment the system is designed to analyse, and it makes no attempt to represent the investable market.

Two consequences follow from that.

The universe is the current one, so companies delisted along the way are not in it. That bounds what any backward-looking statistic computed over it can mean, and it is one of the reasons the public record is forward-only: a record that begins at publication has nothing that can be selected out of it afterwards.

And because it is built around one economic theme, it is concentrated by construction. A ranking inside it is a ranking inside that theme. Treat it as a statement about the market and you will be wrong.

### 2.2 Cadence

A native scheduled task fires after the US close, which means the daily series runs when the machine does.

That constrains what can be rebuilt after the fact. Prices can be refetched from the provider; scores cannot, because a score depends on the fundamentals as they stood on that date. The design answer is not to fill the difference but to publish on a cadence that does not depend on it.

So the public record is weekly rather than daily. It is unaffected by the sampling of the underlying series, and it matches the 2 to 12 week horizon the system targets in any case.

---

## 3. The thirteen sub-scores

Each company gets 13 independent scores on a 0 to 100 scale, in two blocks. When an input is missing the sub-score returns nothing at all rather than a default. Section 5 explains why I care about that so much.

### 3.1 CAN SLIM (70% of total weight)

After O'Neil (2009), turned into explicit metrics with explicit thresholds.

| Code | Criterion | Measured as |
|---|---|---|
| C | Current quarterly earnings | EPS growth versus the year-ago quarter, with turnarounds on a separate scale |
| A | Annual earnings | Multi-year EPS growth, with a dilution overlay against net-income growth |
| N | New highs | Price relative to the 52-week high |
| S | Supply and demand | Share count dynamics and volume behaviour |
| L | Leader or laggard | 12-month relative-strength percentile across the universe |
| I | Institutional sponsorship | Institutional ownership level |
| M | Market direction | One market-health reading, identical for the whole universe |

Three of these need a note before you read any output.

The stored field behind N is a signed percentage *below* the 52-week high, so zero means sitting at the high and negative means under it. I spell the convention out every time, because the sign carries the whole factor and reading it backwards inverts the ranking.

L is the odd one out structurally. It is the only factor scored on a percentile of the universe instead of against a fixed band, because that is what O'Neil's criterion is: each company's twelve-month return ranked against every other company's. At 0.13 it is also the heaviest single weight, so one factor in thirteen is relative and the other twelve are not. Anyone comparing two weeks should know which is which, and everything 3.2.1 says about absolute bands covers the quality block and the rest of CAN SLIM, not this.

Its anchors sit in calendar time. Row counts would be easier and would also mean something different every time the history had a gap. I take the most recent close at or before date D and compare it against the close nearest D minus 365 calendar days, with 21 days of tolerance at either end for weekends, holidays and gaps in the data. Ties go to the earlier close, which sounds like a detail and is not: a tie-break that depends on what order the rows arrived in is not a definition at all. Outside the tolerance there is no relative strength and the factor drops out. Every percentile is stored by date with both anchors, price and date at each end, so you can check the number.

M is constant. The market-health reading is identical for every company on a given date, which lifts or drops all the scores together and cannot touch the ordering. If it looks like a broken factor, that is what it is supposed to look like.

### 3.2 Fundamental quality (30%)

| Factor | Measured as |
|---|---|
| ROIC | Return on invested capital, annualised TTM, using the effective tax rate from the reported tax line instead of an assumed one; plus an asset-growth overlay after Cooper, Gulen and Schill (2008) and a ROIC-trend overlay |
| ROE | Return on equity, annualised TTM |
| FCF | Free cash flow margin, with an accruals overlay after Sloan (1996) comparing operating cash flow against net income |
| Revenue growth | Trailing revenue growth |
| Gross profitability | Gross profit over total assets, after Novy-Marx (2013) |
| Debt safety | Net debt to EBITDA and debt to equity |

Annualisation matters here. A return on capital is a twelve-month flow measured against a balance-sheet stock, so a single quarter's earnings against that same stock understates it by roughly four times. Section 5.2 of the companion paper works through what that cost before it was caught.

#### 3.2.1 Where the bands sit, and why they moved in 2.0

Every factor in this block scores a magnitude linearly between a floor and a ceiling, and the ceiling is an absolute number written into the configuration. It is not a percentile of the universe. That is a deliberate constraint: a percentile band would make a company's score depend on what else happened to be in the universe that week, so the same company with the same accounts could score differently for reasons internal to my own data collection. The number a factor produces has to mean the same thing in every snapshot or the weekly series is not comparable with itself.

What an absolute band buys in stability it can lose by sitting in the wrong place. Three of these ceilings started out too low and I moved them before publishing anything, because when a fifth or a quarter of the universe ties at 100 on a factor, that factor has stopped separating the companies it exists to separate. Measured over the 25 September universe:

| Factor | Ceiling | Would have tied at 100 under the ceiling I started with |
|---|---:|---:|
| Gross profitability (GP/assets) | 90% | 29.3%, at a ceiling of 35% |
| FCF margin | 35% | 19.9%, at 25% |
| ROE | 45% | 20.4%, at 30% |
| Revenue growth | consistency bonus capped at 95 | the bonus could reach 100 on its own |

Each ceiling has to stand on its own terms, and a distribution alone is not terms. A 35% free cash flow margin and a 45% return on equity both sit near the ninetieth percentile here. Gross profit worth 90% of assets is higher, around the ninety-eighth, and I set that one by judgement instead of by the data: earning gross profit equal to 90% of your asset base in a year is exceptional in any sector, and I would rather a quality factor keep its maximum than give it away. The cost shows up immediately. The median score on that factor is 25.5, so it does most of its work in the lower half of its range.

One number is worth stating precisely because it is easy to get backwards. The median company here has gross profit equal to 22.9% of assets, which is *below* the 35% ceiling I started with, not above it. A ceiling can sit well above the median and still blunt a factor, as long as enough of the distribution piles up at the top.

One guard goes with them. A return on equity stops measuring profitability when the denominator produces the number instead of the business, and that happens two ways. Negative equity leaves no denominator at all, so a loss divided by a negative book comes out positive. And an equity base ground down to a sliver gives a ratio in the hundreds of percent that says more about a decade of buybacks than about this year's business.

The guard always fires on the first case. On the second it needs both conditions together: average equity below 10% of assets and a return above 100%. Both, because a bank is leveraged by definition. Testing the denominator on its own threw out 23 of the 69 financials in this universe, every one of them with a return between 8% and 13% that anybody can measure, and because those readings sat below the universe median, dropping them and renormalising pushed the composites up. A guard that rewards the companies it cannot measure is worse than no guard.

It reads the same denominator the ratio uses, the average of opening and closing equity rather than the closing figure. One company in this universe closes the window with equity at 21.7% of assets, a perfectly ordinary balance sheet, and opens it at minus $453m. The average is $857m and the return comes out at 371.5%. A guard looking at the closing row waves that through.

When it fires, the factor returns nothing and the company is scored on its other twelve with the weights renormalised.

### 3.3 What is not in here: valuation

There is no valuation factor, and the omission is deliberate enough to need a section of its own.

An earlier build of this screener had one. Two yields blended in the spirit of Greenblatt (2006), carrying 0.18 of the weight, which made it the heaviest single factor in the model. It went in for an observed reason: expensive companies kept sitting in the top tier week after week, and a price factor heavy enough to pull them back down looked like the answer.

What I had not checked was whether the framework this model is built on supports such a factor at all. It does not. CAN SLIM has seven criteria and none of them prices the company. O'Neil argues the opposite case outright: the leaders of a cycle look expensive on the way up, and screening on a multiple is how you miss them. So the model carried, as its heaviest weight, a criterion that contradicted the framework named on its front page. That is not a calibration problem. It is an incoherent specification, and it does not survive anyone reading the paper and the weights table in the same sitting.

The factor was also covering for two defects that had nothing to do with price. One was saturation: three quality ceilings that a fifth to a quarter of the universe reached, which 3.2.1 sets out. The other was a broken ratio, the return on equity that runs to the hundreds of percent once the equity base has been ground down. Both are fixed where they live.

So the seven CAN SLIM criteria carry 0.70 of the model, which is what a model claiming to implement CAN SLIM ought to look like.

One consequence is the point and not a side effect: the ranking surfaces companies that look expensive. That is the framework doing what it says on the cover. Whether momentum and quality leadership anticipates returns is an open question, and it is not settled by bolting on a factor that makes the output look more sensible.

The two yields are still computed and stored as a diagnostic. Nothing is published from them and they do not touch the ranking.

---

## 4. Putting the thirteen together

### 4.1 Weights

Base weights sum to exactly 1.0, and an assertion at load time fails the run if they don't.

| Factor | Weight | | Factor | Weight |
|---|---:|---|---|---:|
| Relative strength (L) | 0.13 | | ROIC | 0.10 |
| Annual EPS growth (A) | 0.12 | | FCF margin | 0.05 |
| Price momentum (N) | 0.11 | | Revenue growth | 0.05 |
| Market health (M) | 0.10 | | Debt safety | 0.05 |
| Supply/demand (S) | 0.09 | | Gross profitability | 0.03 |
| Institutional (I) | 0.08 | | ROE | 0.02 |
| Quarterly EPS growth (C) | 0.07 | | | |

The seven CAN SLIM criteria take 0.70 and the six quality factors take 0.30. Relative strength is the heaviest single factor, which is both the ranking the framework supports most directly and the easiest one for anyone to reproduce from the stored prices.

The weights are heuristic and deliberately so. Nothing here is fitted or optimised against realised returns. Fitting them to the sample currently available would be the fastest route to a model that describes one month of history and nothing else. They are priors, written down in full so anyone can argue with them and so any future revision shows up as a change and not as a drift.

### 4.2 Sector overrides

Nine sectors carry weight overrides, renormalised back to 1.0 afterwards. ROE gets roughly 3.9 times its base weight in Financial Services, for instance, because that's how you evaluate a bank.

Renormalising after an override keeps the total at 1.0, which also means a superseded base weight can remain pinned by an old override without any total looking wrong. An assertion now fails the run when an override matches a base weight that has been revised. Section 5.3 of the companion paper has the case that produced it.

### 4.3 Composition

The composite averages only the factors that are present, and renormalises their weights:

```
present   = [(s, w) for s, w in weight_map if subscore[s] is not None]
wsum      = sum(weights[w] for _, w in present)
if wsum < coverage_floor:
    return None
composite = sum(subscore[s] * weights[w] for s, w in present) / wsum
```

Since the base weights sum to exactly 1.0, `wsum` is literally the fraction of the model that had data behind it. Coverage comes out of the arithmetic for free, and gets stored per row alongside the score.

### 4.4 Tiers, the ranking floor, and the classifier

Tiers by lower bound: Elite 85, Strong 70, Moderate 55, Weak 40, Avoid 0.

The ranking floor is 60 and the Moderate tier starts at 55, so companies scoring between the two are scored but not ranked. The thresholds answer different questions: the tier describes the score, the floor decides what enters the published ranking.

A second classifier tags each company as compounder, growth, turnaround, value or cyclical, on the first rule that matches. It reads the annualised return on capital, which is the only construction under which a threshold expressed in annual terms means what it says. The companion document works through why that distinction moves a categorical output further than it moves the number behind it.

---

## 5. What happens to bad data and missing data

### 5.1 Corrupt values

Fundamental inputs get winsorised to physically plausible ranges and aggregated by median instead of mean before anything scores them. This came out of the data, not out of principle: the provider gave me a net margin of −2,164,493 at one point. A mean does not survive that. A median doesn't notice.

### 5.2 Missing values

This is the design decision I'd defend hardest, so let me be blunt about it.

Filling in a neutral value is not abstaining. It's making a claim and calling it abstention. Give a company with no ownership data a mid-range institutional score and you haven't said *I don't know*, you've said *this company has weak institutional backing*, and the ranking penalises it accordingly. Give a missing return on capital a zero and you've said the business earns nothing on its capital, which is about the worst thing you can say about a business.

The bias that follows isn't noise, it's systematic. Small caps, foreign issuers and ADRs have thinner data, so they sink in the ranking because of coverage rather than quality. What the screener then "discovers" is which companies Yahoo covers well, dressed up as an investment insight.

So any factor without an input returns nothing, drops out, and the remaining weights renormalise.

### 5.3 The floor, and why it is region-aware

You can't renormalise without a limit. A company with nothing but price momentum would get a rescaled momentum number that looks exactly like a full Investment Score and contains one factor's worth of information. So below some minimum fraction of model weight, the company doesn't get scored and doesn't rank.

A single floor applied to everyone reintroduces the bias the exclusion policy exists to remove. The companies a flat 0.70 excluded were mostly ADRs and non-US listings, clustered together and missing the same block of fundamentals, because a threshold calibrated to US disclosure practice was being applied to issuers who do not file on it.

The floor is therefore region-aware: 0.70 for US issuers, 0.55 for everyone else, with unknown domicile defaulting to the stricter one. Any row scored below 0.70 carries a low-coverage flag and its actual coverage value, so it is scored with the caveat attached instead of dropped for it. On 25 September sixteen rows carry that flag, fourteen of them non-US, and three of the sixteen score high enough to appear in the ranking.

A second condition joined the floor in 2.1, and the reason is worth setting out because it is a consequence of removing valuation. The seven CAN SLIM criteria carry 0.70 of the model, which is exactly the US floor. In the sectors that carry no weight override, a company with all seven criteria and not one quality factor lands on 0.7000 and clears the floor. Its Investment Score would be a statement about price behaviour wearing the same clothes as a full one, and "70% of the model was present" would be true and misleading at once. So the composite must also carry at least half of the quality block's weight; below that, the row is flagged. It is the same caveat-not-exclusion policy as the regional floor, applied to the shape of what is missing and not only to how much.

The effect is that exclusion is explicit and never silent. A company is either absent from the ranking, or present and flagged with the exact fraction of the model that stood behind its score.

### 5.4 Coverage in practice

Measured over the 778 companies scored on 25 September 2026:

| Factor | Present | | Factor | Present |
|---|---:|---|---|---:|
| Market direction (M) | 100.0% | | FCF margin | 92.9% |
| New highs (N) | 100.0% | | ROE | 90.6% |
| Institutional (I) | 99.7% | | Gross profitability | 89.1% |
| ROIC | 99.7% | | Revenue growth | 86.5% |
| Annual EPS (A) | 99.6% | | Quarterly EPS (C) | 79.3% |
| Supply/demand (S) | 99.6% | | | |
| Relative strength (L) | 99.0% | | | |
| Debt safety | 98.6% | | | |

Quarterly earnings is the worst covered factor in the model, absent for one company in five, and the reason is mechanical rather than a gap in the feed. The provider populates a year-over-year growth figure only on a company's most recently reported quarter, and a point-in-time read treats a quarter as public only after the reporting lag in 2.2. A company whose fiscal quarter ended inside that window therefore has no growth figure the model can see at all: the newest row is not yet visible and the older ones carry nothing. It falls hardest on issuers whose fiscal year does not follow the calendar.

Two consequences follow, and both are in the published output rather than hidden in it. The coverage fraction is written to every row, so a score built on 88% of the model says so. And a reader comparing two weeks should expect this factor to move for reasons that are about the reporting calendar and not about the companies.

---

## 6. Data sources

One provider supplies prices, fundamentals, ownership and exchange rates. The controls described in the companion document establish that the system reproduces that data faithfully through the pipeline.

Exchange-traded funds have no fundamentals and are not scored.

---

## 7. Design choices and trade-offs

The composite is a linear weighted mean and not a trained model, because a score built that way decomposes into its parts and can be explained line by line. The cost is that the weights are static and cannot adapt to a regime. At this sample size that is the right trade: a model fitted to one month of forward returns fits noise and nothing else.

Validation uses rank correlation. The sub-scores are bounded, clamped and carry bonuses, so their scale is neither linear nor comparable across factors, while their ordering is both.

Missing inputs are excluded and never imputed, everywhere in the system. Section 5 covers the mechanism. A hole beats an invented number.

The daily pass runs as a native scheduled task and not as a resident process, so it survives a reboot and needs nothing kept running. The cost is the cadence described in 2.2, which the weekly publication protocol is built around.

Publication is weekly for the reasons in 2.2 and Section 8.

---

## 8. Rules for the public record

Written down before the record started, which is the only time writing them down counts for anything.

1. Forward only. The record starts at the first public snapshot. Reconstructed history is research and never gets presented as results. The companion document explains why it can't be a record even in principle.
2. Weekly, at Friday's close. The operational gaps in 2.2 would otherwise leave holes, and a record with holes looks identical to a curated one from outside. Weekly also fits a 2 to 12 week horizon.
3. Fixed N, declared in advance, and everything that comes out goes out. No choosing after the fact.
4. Benchmarked. Any performance statement gets measured against a broad index and a technology-weighted one over the same window. A return without a benchmark isn't a result.
5. Dated and immutable. Each snapshot carries its run date and time and goes into a public repository, so it has a timestamp and a hash instead of being an editable post. Nothing gets edited after publication; corrections are new items that reference the original.
6. Failures get published in place of the snapshot, with the reason.
7. Losses get the same prominence as gains.
8. No return promises. This is a filter and a prioritisation, not a fund and not a recommendation.

---

## 9. Disclosures

I'm Marc Romero, an independent researcher. I'm not an investment firm, an adviser, an analyst at a regulated entity or a fund manager, and I hold no licence to give investment advice.

Everything I publish is educational and methodological. It describes a system and what it produces. It is not investment advice, not a personal recommendation, not a solicitation, and not an offer of any financial product or service.

I make no claim, express or implied, about past or future returns. The system ranks companies against the criteria set out above; it does not forecast prices, and no edge has been demonstrated or is claimed. Methodology, assumptions, horizon and risks are in Sections 1 to 7 above, and this document is the standing reference for every ranking I publish.

Where I hold a position in a security I discuss, I disclose it in the item itself at the time of publication, along with any net position above 5% of a company's capital. Nothing I publish is sponsored or paid for by an issuer, and if that ever changes it will say so in the item.

Equity investment carries risk of total loss. Nothing here takes account of anyone's circumstances, objectives or tolerance for risk. If you're acting on public information, talk to someone properly authorised.

---

## References

Cooper, M. J., Gulen, H., & Schill, M. J. (2008). Asset growth and the cross-section of stock returns. *Journal of Finance*, 63(4), 1609–1651.

Greenblatt, J. (2006). *The little book that beats the market*. Wiley.

Novy-Marx, R. (2013). The other side of value: The gross profitability premium. *Journal of Financial Economics*, 108(1), 1–28.

O'Neil, W. J. (2009). *How to make money in stocks* (4th ed.). McGraw-Hill.

Sloan, R. G. (1996). Do stock prices fully reflect information in accruals and cash flows about future earnings? *The Accounting Review*, 71(3), 289–315.

---

*Versioned. When the methodology changes materially I publish a new version with a dated changelog, so any ranking can be read against the specification that produced it.*

v1.0, 27 September 2026. First published specification, and the one the forward record in `track-record/` starts from. Rule 5 of Section 8 applies from that first snapshot onward.
