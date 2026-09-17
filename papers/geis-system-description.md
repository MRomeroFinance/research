# GEIS: System Description

### A technical note on the design of a quantitative equity screener

Marc Romero, independent researcher  
Version 1.2, September 2026

*This is a system description rather than a research paper. It covers what GEIS measures, how those measurements combine, and the reasoning behind the design. The empirical work, meaning the fidelity controls and the first measurement of predictive content, is in a separate document, "Fidelity Controls for a Quantitative Equity Screener, and What They Do Not Prove". That one cites this one.*

---

## 1. What this is

GEIS runs once a day over about 800 listed companies. It scores each one on 14 criteria, combines them into a single number between 0 and 100, and sorts.

The question it answers: given a fixed set of criteria that I've written down and can reproduce, which companies in my universe come out on top today?

What it does not do is forecast prices, and that distinction is the one thing to carry out of this document. The two questions get run together constantly and they need completely different kinds of evidence. Whether a system reproduces its own output exactly is an engineering property, and one you can establish by construction. Whether its ranking anticipates returns is an empirical claim that needs realised returns, at the right horizon, across more than one regime. This system is built to answer the first question and to make the second one measurable. It is not built to answer the second, and nothing here claims that it does. The companion document works through what the evidence available so far will and will not support.

Nothing here is investment advice, a recommendation, or a solicitation. Section 9 spells that out.

---

## 2. Universe, horizon and cadence

| Property | Value |
|---|---|
| Universe | ~805 active listings: an AI-ecosystem construction across 20 thematic layers, plus the S&P 500 |
| Scored (13 September 2026) | 774 / 805 (96.1%) |
| Frequency | Daily batch, weekdays, after the US close |
| Target holding horizon | Swing, 2 to 12 weeks, with fundamental backing |
| Persistence | Local SQLite, one dated snapshot per run |
| Runtime | About 12 minutes for a full pass |
| Test suite | 261 tests |

### 2.1 About the universe

The universe is constructed rather than sampled: it covers the segment the system is designed to analyse, not the investable market as a whole.

Two consequences follow from that.

The universe is the current one, so companies delisted along the way are not in it. That bounds what any backward-looking statistic computed over it can mean, and it is one of the reasons the public record is forward-only: a record that begins at publication has nothing that can be selected out of it afterwards.

And because it is built around one economic theme, it is concentrated by construction. A ranking inside it is a ranking inside that theme, not a statement about the market.

### 2.2 Cadence

A native scheduled task fires after the US close, which means the daily series runs when the machine does.

That constrains what can be rebuilt after the fact. Prices can be refetched from the provider; scores cannot, because a score depends on the fundamentals as they stood on that date. The design answer is not to fill the difference but to publish on a cadence that does not depend on it.

So the public record is weekly rather than daily. It is unaffected by the sampling of the underlying series, and it matches the 2 to 12 week horizon the system targets in any case.

---

## 3. The fourteen sub-scores

Each company gets 14 independent scores on a 0 to 100 scale, in three blocks. When an input is missing the sub-score returns nothing at all rather than a default. Section 5 explains why I care about that so much.

### 3.1 CAN SLIM (52% of total weight)

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

Three of these need a note if you're going to read the output.

On N, the stored field is a *signed percentage below* the 52-week high. Zero means at the high, negative means below. The convention is stated explicitly because the sign carries the whole factor.

On L, relative strength anchors in calendar time rather than row counts. I take the most recent close at or before date D, compare it against the close nearest to D minus 365 calendar days, and allow 21 days of tolerance at either end for weekends, holidays and gaps in the data. Ties go to the earlier close. That last bit matters more than it looks. A tie-break that depends on what order the rows arrived in isn't really a definition. Outside the tolerance there's no relative strength and the factor drops out. Every percentile gets stored by date along with both anchors, price and date at each end, so you can check the number instead of trusting it. The companion document sets out why that definition had to live in one place rather than three.

On M, the market-health reading is the same for every company on a given date. It moves all scores up or down together and can't touch the ordering. It's constant by design, so don't read it as a broken factor.

### 3.2 Fundamental quality (30%)

| Factor | Measured as |
|---|---|
| ROIC | Return on invested capital, annualised TTM, using the effective tax rate from the reported tax line instead of an assumed one; plus an asset-growth overlay after Cooper, Gulen and Schill (2008) and a ROIC-trend overlay |
| ROE | Return on equity, annualised TTM |
| FCF | Free cash flow margin, with an accruals overlay after Sloan (1996) comparing operating cash flow against net income |
| Revenue growth | Trailing revenue growth |
| Gross profitability | Gross profit over total assets, after Novy-Marx (2013) |
| Debt safety | Net debt to EBITDA and debt to equity |

Annualisation matters here. A return on capital is a twelve-month flow measured against a balance-sheet stock, so a single quarter's earnings against that same stock understates it by roughly four times. The companion document works through the consequences.

### 3.3 Valuation (18%)

Two yields blended, in the spirit of Greenblatt (2006):

```
EV             = market capitalisation + total debt − cash
earnings_yield = EBIT_TTM / EV
fcf_yield      = FCF_TTM  / EV
```

Each yield gets scored linearly against a fixed band and whatever components exist get averaged. Cheaper ranks higher. The factor returns nothing if market capitalisation is missing, if enterprise value comes out non-positive, or if there's neither earnings nor cash flow to work with.

Currency needs care here. Market capitalisation comes from the price, so it's in the trading currency. Debt and cash come from the accounts, so they're in the reporting currency. For a US company those are the same thing and the expression above is fine. For an ADR they aren't, and what you get is not an error but a plausible number that happens to be nonsense. So the system converts the financial leg explicitly, and where no exchange rate is available it returns nothing rather than a guess. The companion document works through the case in full.

Two notes on how to read this factor. Enterprise value is computed textbook-style, without adjustment for minority interests, capitalised leases or pensions. And the factor prices a company, it does not assess the business behind it: a high yield can reflect a low price or a declining business, and this score alone does not tell the two apart.

---

## 4. Putting the fourteen together

### 4.1 Weights

Base weights sum to exactly 1.0, and an assertion at load time fails the run if they don't.

| Factor | Weight | | Factor | Weight |
|---|---:|---|---|---:|
| Valuation | 0.18 | | Institutional (I) | 0.06 |
| ROIC | 0.10 | | Quarterly EPS growth (C) | 0.05 |
| Relative strength (L) | 0.10 | | FCF margin | 0.05 |
| Annual EPS growth (A) | 0.09 | | Revenue growth | 0.05 |
| Price momentum (N) | 0.08 | | Debt safety | 0.05 |
| Supply/demand (S) | 0.07 | | Gross margin | 0.03 |
| Market health (M) | 0.07 | | ROE | 0.02 |

Valuation carries more weight than anything else, which was a deliberate and fairly aggressive call. It turned GEIS from a growth-and-quality screener into a quality-and-value one. What pushed me there wasn't theory, it was watching expensive companies sit in the top tier week after week because the growth factors had saturated at 100 and the price factor wasn't heavy enough to pull them back down.

The weights are heuristic and deliberately so: not fitted, not optimised against realised returns. Fitting them to the sample currently available would be the fastest route to a model that describes one month of history and nothing else. They are priors, written down in full so that anyone can argue with them and so that any future revision shows up as a change rather than a drift. The companion document sets out the conditions under which a revision would be defensible.

### 4.2 Sector overrides

Nine sectors carry weight overrides, renormalised back to 1.0 afterwards. ROE gets roughly 3.9 times its base weight in Financial Services, for instance, because that's how you evaluate a bank.

Renormalising after an override keeps the total at 1.0, which also means a superseded base weight can remain pinned by an old override without any total looking wrong. An assertion now fails the run when an override matches a base weight that has been revised. The companion paper works through the case.

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

Fundamental inputs get winsorised to physically plausible ranges and aggregated by median instead of mean before anything scores them. This came out of the data rather than out of principle: the provider gave me a net margin of −2,164,493 at one point. A mean does not survive that. A median doesn't notice.

### 5.2 Missing values

This is the design decision I'd defend hardest, so let me be blunt about it.

Filling in a neutral value is not abstaining. It's making a claim and calling it abstention. Give a company with no ownership data a mid-range institutional score and you haven't said *I don't know*, you've said *this company has weak institutional backing*, and the ranking penalises it accordingly. Give a missing return on capital a zero and you've said the business earns nothing on its capital, which is about the worst thing you can say about a business.

The bias that follows isn't noise, it's systematic. Small caps, foreign issuers and ADRs have thinner data, so they sink in the ranking because of coverage rather than quality. What the screener then "discovers" is which companies Yahoo covers well, dressed up as an investment insight.

So any factor without an input returns nothing, drops out, and the remaining weights renormalise.

### 5.3 The floor, and why it is region-aware

You can't renormalise without a limit. A company with nothing but price momentum would get a rescaled momentum number that looks exactly like a full Investment Score and contains one factor's worth of information. So below some minimum fraction of model weight, the company doesn't get scored and doesn't rank.

A single floor applied to everyone reintroduces the bias the exclusion policy exists to remove. At a flat 0.70, 18 of the 28 companies it excluded were ADRs or non-US listings, bunched between 0.40 and 0.48 coverage and missing the same block of fundamentals. That is a threshold calibrated to US disclosure practice being applied to issuers who do not file on it.

The floor is therefore region-aware: 0.70 for US issuers, 0.55 for everyone else, with unknown domicile defaulting to the stricter one. Any row scored below 0.70 carries a low-coverage flag and its actual coverage value, so nineteen ADRs are ranked with the caveat attached rather than dropped for it.

The effect is that exclusion is explicit rather than silent. A company is either absent from the ranking, or present and flagged with the exact fraction of the model that stood behind its score.

### 5.4 Coverage in practice

Most factors are present for over 97% of scored companies. Debt safety is the lowest at around 83%, because a fair number of companies have neither a usable net-debt-to-EBITDA nor a debt-to-equity figure. Gross profitability sits at about 91%.

---

## 6. Data sources

One provider supplies prices, fundamentals, ownership and exchange rates. The controls described in the companion document establish that the system reproduces that data faithfully through the pipeline.

Exchange-traded funds have no fundamentals and are not scored.

---

## 7. Design choices and trade-offs

A linear weighted mean rather than a trained model, because any score decomposes into its parts and can be explained. The trade-off is that the weights are static rather than regime-adaptive, which is the right trade at this sample size: a trained model on one month of forward returns fits noise.

Rank correlation rather than linear correlation when validating, because the sub-scores are bounded with clamps and bonuses, so their scale isn't linear or comparable across factors. Their ordering is.

Exclusion rather than imputation, everywhere. Section 5 covers this. The same principle drives the currency handling in 3.3 and the validation policy in the companion document. A hole beats an invented number.

A native scheduled task rather than a resident process, because it survives reboots and requires nothing kept running. The trade-off is the cadence described in 2.2, which the weekly publication protocol is built around.

Weekly publication rather than daily, for the reasons in 2.2 and Section 8.

---

## 8. Rules for the public record

Written down before the record started rather than after.

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

*Versioned. When the methodology changes materially I publish a new version with a dated changelog, so any ranking can be read against the methodology that produced it.*

v1.2, 17 September 2026. Editorial revision. No changes to methodology, parameters or figures from v1.1.
