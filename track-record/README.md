# The record

One CSV per week. Each holds the top 10 companies by Investment Score at that Friday's US close, committed the same day.

The rules below went up before the first file did.

## The rules

Forward only. The record starts with the first file in this folder. I do have historical scores from re-running the current code over past dates, and that's research, not results. It can't be a record even in principle: the fundamentals table only keeps the current version of each quarter, so any company that has restated gets scored on numbers nobody had at the time. The paper sets out the reasoning.

Weekly, at Friday's close. A weekly cadence is unaffected by the daily sampling constraints described in the system description, and it matches a 2 to 12 week holding horizon.

Ten names, fixed in advance and unchanged. A variable N would make the selection discretionary.

Everything that comes out goes out. The `universe-*.csv` files hold every company scored that day, so whatever isn't in the top 10 is still here.

If a run fails, the failure gets published instead of the snapshot, with the reason.

Benchmarked. Anything I say about performance gets measured against SPY and QQQ over the same window. A return without a benchmark isn't a result.

Nothing gets edited. Corrections are new commits that reference what they're correcting. That's why this lives in git and not in a post.

Losses get the same prominence as gains.

No return promises. This is a filter over a universe. Not a fund, not a recommendation, not advice.

## Columns

| Column | What it is |
|---|---|
| `date` | The close the ranking refers to |
| `rank` | 1 to 10 |
| `ticker`, `name` | The company |
| `close` | Closing price on that date, in `currency`. This is what the record gets measured from |
| `currency` | Trading currency |
| `investment_score` | Composite, 0 to 100 |
| `tier` | Elite 85+, Strong 70+, Moderate 55+, Weak 40+, Avoid below |
| `company_type` | compounder, growth, turnaround or value when the classifier's rules fire; cyclical is the residual bucket and means none of them did, so it carries no claim about the business |
| `factor_coverage` | Fraction of model weight that actually had data behind it |
| `low_coverage` | 1 when the composite rests on less of the model than a full row: either total coverage below the 0.70 US floor, admitted under the 0.55 floor for non-US issuers, or less than half the quality block present |

The closing price is the column that carries the record. Without it, three months from now nobody can check what the list did without rebuilding the prices themselves, and a record you can't check isn't one.

## Reading a file

The score ranks a company inside my universe on that day, against the criteria set out in the system description. It is not a forecast and it isn't offered as one. I publish the ranking because it is what the system produced, and I publish it here rather than in a post so that the claim can be checked instead of argued about.

`universe-*.csv` is every company scored that date, in score order, flags included. A company that falls below its region's coverage floor is not scored at all and so is not in the file; one that clears the floor but rests on less of the model than a full row is in it, with `low_coverage` set to 1 and its exact `factor_coverage` beside it. The system description explains both.

Companies in the active universe that produce no score at all are in neither file. On 25 September 2026 that is 27 of 805, most of them symbols the data feed never resolved.

## Disclaimer

Not investment advice, not a personal recommendation, not a solicitation. I'm an independent researcher and I'm not authorised to give investment advice. Equity investment carries risk of total loss.
