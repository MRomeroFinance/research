# Weekly notes — 25 September 2026

First file of the forward record. The ranking below is what the screener produced at Friday's US close, and the CSV that goes with it sits in `track-record/`. This file explains what the system measured. It is commentary, not the record, and no return is ever computed from it.

778 companies scored out of 805 active. 123 cleared the ranking floor of 60, and 16 rows carry a low-coverage flag. Market health read 87.3, with SPY 1.51% above its 50-day average and 7.91% above its 200-day, VIX at 14.87. That reading is identical for every company in the universe, so it lifts the level of every score and cannot touch the order.

## What the model is, in one paragraph

Thirteen criteria, each scored 0 to 100. Seven of them are O'Neil's CAN SLIM criteria and carry 0.70 of the weight; six are fundamental quality measures and carry 0.30. Twelve of the thirteen score a magnitude against a band that is a fixed number, so the same accounts produce the same score in any week. The thirteenth is relative strength, which is a cross-sectional percentile of the universe by construction, because that is what O'Neil's criterion is; at 0.13 it is also the heaviest single weight, so it is worth knowing that one factor in the model is relative and twelve are not. The Investment Score is the weighted mean of the criteria that have data, with the missing ones excluded and the remaining weights renormalised. Nine sectors carry weight overrides, so the exact weight on a factor depends on the company's sector. Nothing is imputed: a factor with no input produces no number, and the coverage figure on every row says how much of the model stood behind that score.

## How to read the numbers

Everything called a score below is a sub-score on that 0 to 100 scale. It says where a company sits on one criterion, and nothing about what the company is worth.

Every figure quoted as a magnitude is the one the sub-score was computed from, not a similar figure from the same report. That distinction did real work this week. Return on equity is the annualised trailing-twelve-month figure over average equity, not the per-quarter number stored beside it: across these ten the two differ by factors running from 2.4× to 6.2×, and for MeiraGTx the annualised figure is the lower of the two. The free-cash-flow margin is the median of up to four quarters, not the latest one. And growth figures are shown after the model's own clamps, so a quarterly earnings jump the provider reports at 1,381% appears here as the 500% the model actually scored.

## Three things about the list as a whole

**Nine of the ten failed the supply-and-demand criterion.** Aya Gold scored 0.96 on it, Fortinet 2.91, Comfort Systems 5.68, TSMC 6.58, NVIDIA 8.95, Seagate 9.06, Micron 10.49, Arista 13.66 and Western Digital 15.14. Only MeiraGTx traded above its own 30-day average volume on Friday.

O'Neil put S in CAN SLIM to catch accumulation, and accumulation is what is missing. The composite carries them anyway, because S is worth 9% and the other twelve factors outvote it. I do not think that is wrong, but a top ten that collectively fails one of the seven criteria the model is named after is the kind of thing somebody was going to notice, so it goes at the top of the file.

One caveat, and it is not a small one. A single day's volume against a 30-day average is a noisy measure. This describes Friday. It is not evidence about any trend in these stocks' volume, and nothing here should be read that way.

**Six of the ten are one trade.** Micron, Seagate and Western Digital are memory and storage. NVIDIA, TSMC and Arista are the silicon and the networking underneath the same demand. Fifteen of the top twenty-five are classified Technology, against 30% of the universe: the concentration in the list is roughly double the concentration in what it was drawn from. Nothing in the model penalises correlation, and it is not equipped to: it scores companies one at a time and sorts. A reader building anything from this list has to do the diversification work themselves, because the screener has not done any of it.

**The accrual readings split the list, and not where you would expect.** The free-cash-flow factor carries a Sloan overlay comparing trailing net income against trailing operating cash flow, scaled by assets. Positive means reported profit is running ahead of cash. Western Digital is at +38.7% of assets, the largest of the ten by a distance, MeiraGTx +16.5% and NVIDIA +13.1%. At the other end, Comfort Systems is at −13.1% and Fortinet −11.7%, the conservative case: more cash than the income statement shows. Note the ranks. The three positive readings are at 10, 9 and 6; the two most conservative are at 3 and 2. This is one of the few things in the model that looks at the quality of reported earnings rather than their size, and it is pointing at the bottom of the list, not the top.

## The ten

### 1 · MU — Micron Technology · $1,082.28 · score 81.50

Memory and storage. Relative strength is the number that stops you: the 99.7th percentile of the universe on a 591.21% twelve-month gain from $156.58. It is the highest of the ten, though not of the universe, where AXT and Moderna sit above it.

Quarterly earnings and revenue growth both score 100, and both are clamped: the provider reports quarterly EPS growth of 1,381% against the year-ago quarter, and the model scores the 500% its own bounds allow. Return on invested capital is 58.21% on EBIT over average invested capital, return on equity 66.64% annualised, net margin 68.1%, and net debt is −0.52× EBITDA, so it holds more cash than debt.

The weak legs are supply and demand at 10.49, with Friday's 19.5m shares against a 25.6m average, then gross profitability at 54.28 on gross profit equal to 48.85% of assets, and the free-cash-flow margin at 64.67 on a four-quarter median of 22.63%. The latest quarter's margin was 42.36%, so the median is doing what a median is for. Institutions hold 79.91%, down 1.69% on the quarter. Coverage 1.0.

### 2 · FTNT — Fortinet · $173.46 · score 81.29

Network security. The quality block is close to as good as this model measures: return on invested capital 77.36%, return on equity 117.44%, a free-cash-flow margin of 40.03% with accruals at −11.7% of assets, and net debt at −3.34× EBITDA, the deepest net cash position of the ten. Annual earnings score 100 on three years of EPS growth at 7.5%, 55.1% and 36.1%.

It closed 4.36% below its 52-week high, second closest of the ten, with relative strength at the 92.5th percentile on a 108.46% year. The soft readings are quarterly earnings at 65.61, on EPS growth of 45.61% that is unremarkable only on a screen full of triple digits, and supply and demand at 2.91: 2.50m shares against a 4.36m average. Coverage 1.0.

### 3 · FIX — Comfort Systems USA · $1,658.91 · score 80.58

Mechanical and electrical contracting: heating, ventilation, air conditioning, plumbing and electrical installation for commercial, industrial and institutional customers. The only industrial name in the ten. Quarterly EPS up 91.74%, revenue growth at a 50.26% median, both scoring 100, and three years of annual EPS growth at 97.6%, 62.1% and 32.0%. Return on invested capital 49.73%, net debt −2.54× EBITDA, debt to equity 0.10. Institutions hold 95.96%, the highest reading of the ten.

Gross profitability is its weakest quality factor at 37.72, on gross profit equal to 33.95% of total assets. The margin on sales behind it is 25.85%, which is what a contracting business looks like, and it is the reason the factor is measured over assets and not over sales: a 25% margin on a fast-turning asset base is a different business from the same margin on a slow one.

It closed 19.94% below its 52-week high and traded 227,000 shares against a 354,000 average. Accruals are −13.1% of assets, the most conservative reading in the ten. Coverage 1.0.

### 4 · STX — Seagate · $916.83 · score 80.36

Hard drives and storage, based in Singapore. Up 320.47% over twelve months, the 98.8th percentile, with quarterly EPS up 150% and revenue growth at a 48.49% four-quarter median. Return on invested capital 67.01%.

This is the one row of the ten where a factor was thrown out by a guard rather than by absent data, and it is worth walking through because it is exactly what the guard is for. The annualised return on equity computes to 371.53%. The model divides trailing net income by *average* equity, and Seagate's equity at the start of that window was negative $453m against $2.17bn at the end, so the denominator is an $857m average on a $9.97bn balance sheet. A ratio produced by averaging across a book that was underwater for part of the year is not a measurement of profitability, so the factor returns nothing and the remaining twelve renormalise. Coverage 0.98.

What the balance sheet does say is scored separately and not ignored: debt to equity 1.78 and net debt at 1.32× EBITDA give it 66.87 on debt safety, its lowest quality reading. Gross profitability 61.93 on gross profit equal to 55.74% of assets. Supply and demand 9.06.

### 5 · ANET — Arista Networks · $206.55 · score 79.30

Data-centre and AI networking. It closed 3.88% below its 52-week high, the closest of the ten, on a 44.38% year that puts relative strength at the 79.9th percentile, the third lowest here after NVIDIA at 69.2 and MeiraGTx at 73.4. The free-cash-flow margin is the highest of the ten at a 51.44% median, revenue growth 37.69%, return on invested capital 28.68%, return on equity 31.48%.

Two readings need a footnote. Gross profitability is 31.11, the lowest quality score in the ten, on gross profit equal to 28.00% of assets, against a margin on sales of 62.93%. The gap between those two figures is the size of the asset base, which grew from $14.0bn at the end of FY-2024 to $23.7bn by the June quarter; what that base is made of, the database does not say.

And the debt-safety score of 91.36 rests on the FY-2022 annual statement. That is the last statement carrying a debt line, and it shows $44m of debt against $4.89bn of equity; the provider has reported no debt figure on any quarter since. The model is built to fall back to the most recent statement that carries the ratio, which here is nearly four years old. Worth stating, because an absent field is not the same as an absent liability.

Coverage 1.0.

### 6 · NVDA — NVIDIA · $225.07 · score 77.26

Coverage 0.88, the only row of the ten below 1.0, though not the lowest in the top twenty-five: Dell is at 0.86 and NetApp ties at 0.88.

Two factors produce no score, quarterly earnings and revenue growth, and the reason is a property of the system rather than of the company. Both are computed from year-over-year growth figures that the data feed populates only on a company's most recently reported quarter. NVIDIA's most recent quarter ended on 31 July, and the reconstruction described at the end of this file treats a quarter as public only 75 days after it closes, so at 25 September that row is not visible to the scorer and the older rows it can see carry no growth figures at all. Both factors drop out, the remaining eleven renormalise, and 12% of the model is not standing behind this score. That is what the coverage figure exists to say. The same mechanism makes quarterly earnings the worst-covered factor in the model, absent for one company in five.

What is measured: return on invested capital 90.57%, return on equity 114.29%, annual EPS growth of 66.0%, 145.5% and a clamped 500% over three years, a free-cash-flow margin of 45.01% with accruals at +13.1% of assets, and a balance sheet at −0.02× EBITDA.

Relative strength is the surprise. The raw percentile is 69.2, and the factor subtracts 15 points below the 70th percentile, so it scores 54.20. On a 26.83% twelve-month gain, NVIDIA ranks below two thirds of this universe.

### 7 · AYA.TO — Aya Gold & Silver · C$40.34 · score 77.01

Silver producer, listed in Toronto, reporting in US dollars. Quarterly EPS up 299.79% against a base quarter of $0.06, and revenue growth at a 150.66% median, both at the ceiling. Relative strength sits at the 97th percentile on a 211.99% year, and it closed 6.45% below its 52-week high.

The quality block is where it separates from the rest: return on invested capital 21.58% and return on equity 25.80%, scoring 71.33 and 52.01, the lowest readings on both returns on capital anywhere in the ten. Debt to equity is 0.18 and the free-cash-flow margin 20.51%, with accruals at −8.8% of assets. Institutional ownership is 59.12% and rose 16.51% on the quarter, the largest increase of the ten.

Supply and demand is 0.96, the lowest reading in the table: 605,000 shares against a 1.15m average. This is also the row where the sector overrides bite hardest: in Basic Materials the model puts 0.1188 on return on invested capital and only 0.0297 on quarterly earnings, so one of its two ceiling readings on the growth factors counts for less than half what it would elsewhere. Coverage 1.0.

### 8 · TSM — Taiwan Semiconductor · $450.61 · score 76.52

Foundry. Quarterly EPS up 77.41%, revenue growth at a 36.05% median, return on invested capital 31.63%, return on equity 40.26%, a free-cash-flow margin of 26.13% and net debt at −2.14× EBITDA. It closed 5.67% below its 52-week high on a 64.16% year, which puts relative strength at the 86.4th percentile. Annual EPS growth over three years reads 46.4%, 39.9% and −17.5%, and that negative year costs it ten points on the A factor.

Institutional sponsorship reads 15.48% and scores 27.36, the lowest institutional figure of the ten by a distance. That is the figure the provider reports for this listing. What it covers, and whether the shares that trade in Taipei are inside or outside it, is not something the database records, so the factor is scored on a number whose scope I cannot verify. Reported because using it quietly would be worse.

Fundamentals are in New Taiwan dollars and the price in US dollars, and the model converts explicitly wherever the two would otherwise be mixed. Coverage 1.0.

### 9 · MGTX — MeiraGTx · $10.91 · score 76.09

Gene therapy, $1.05bn market capitalisation, 393 employees.

The accounting figures are extraordinary. Revenue growth is at the model's 500% clamp, which means the underlying figure is larger still. Gross profit equals 98.21% of total assets. Return on invested capital is 84.28% and return on equity 72.28%. All four score at or near the ceiling.

Two readings in the same table complicate them. The accrual ratio is +16.5% of assets, the second highest of the ten: reported profit is running well ahead of the cash the business generated. And the quarterly-earnings factor did not score the 466.67% the raw growth would suggest; the year-ago base quarter was a loss of $0.48 per share, so the model classified it as a turnaround and assigned the fixed turnaround score of 55.00 instead of letting a percentage off a negative base run to the ceiling.

What produced the revenue figure, I do not know. The database holds a total, not its composition, and I am not going to guess at a cause I cannot verify.

It is the only name of the ten that traded above its 30-day average volume on Friday, at 1.55m against 886,000, and institutional ownership rose 9.59% on the quarter to 70.48%, the second largest increase here after Aya Gold. It closed 28.93% below its 52-week high, the second furthest of the ten. Coverage 1.0.

### 10 · WDC — Western Digital · $456.81 · score 75.97

Storage. Up 326.75% over twelve months, the 99th percentile on relative strength. Quarterly earnings and revenue growth both score 100, quarterly EPS at the 500% clamp. Return on invested capital 43.39%, return on equity 129.10%, a free-cash-flow margin of 25.48%.

Two readings pull the other way, and both are the model's own. It closed 42.87% below its 52-week high, which scores 28.55 on new highs, the lowest reading of the ten by a wide margin: under CAN SLIM that is not a minor blemish, because O'Neil's framework wants leaders near their highs and this one is not. And the accrual ratio is +38.7% of assets, the largest in the ten by some distance: trailing net income is running far ahead of trailing operating cash flow. Neither stops it ranking tenth, because eleven other factors outvote them.

Institutions hold 91.92%, down 9.28% on the quarter, the largest institutional decrease of the ten. Coverage 1.0.

## How this snapshot was produced, and why it is the only one like it

This entry is a reconstruction, and that has to be on the record because every entry after it will be produced differently.

The live Friday run scores the universe with the data the provider has at that moment. A re-score of a past date cannot do that, because the point of re-scoring a past date is to avoid using anything that was not available then. So the system reads the date point-in-time: prices, fundamentals, ownership, market health and relative strength as they stood on 25 September, with fundamentals treated as public only 75 days after the quarter closes. That reporting lag is a proxy, since the database stores the end of each period and not the date it was reported, and it is deliberately set to exclude too much and never too little.

The effect is measurable and goes in one direction. The live run on Friday produced a quarterly-earnings score for 684 of the 774 companies it scored. The reconstruction produces one for 617 of 778, so the number with no score on that factor rises from 90 to 161. Average coverage across the universe falls from 0.978 to 0.966. NVIDIA is the visible case in the ten above.

Nothing is wrong with either number; they answer different questions. The live run asks what the screener saw on the day, the reconstruction asks what it could defensibly have seen. This snapshot had to be reconstructed because the model was revised after the date, which is a thing that happens before a record starts and will not happen again: from 2 October onward every entry comes straight out of the live Friday run with no re-scoring of any kind.

The prices in the CSV are unaffected. They were sealed on Friday night and the reconstruction cannot alter them, which is the property that makes the forward record measurable regardless of any of this.

## What is not here

No target prices, no ratings, no position sizing, and no view on whether any of these should be bought or held. Where a figure looks extreme, this file says what produced it and stops there; it does not tell you what happens next, because the model has no opinion about that and neither does the record until it has one.

The screener ranks companies against criteria written down in full and reproducible from the CSV and the papers in this repository. It does not forecast prices, it has not been shown to anticipate returns, and the record that would let anyone judge that starts with this file.

Not investment advice. Independent researcher, not a regulated adviser. No positions in any company named.
