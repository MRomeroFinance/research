# Research

Quantitative equity research by Marc Romero, published with the method attached.

I build systems that rank equities, and I publish the method alongside the output. Right now that means one screener, its documentation, and a weekly record of what it produces.

The screener evaluates companies against a fixed set of criteria that are written down here in full and reproducible from the stored data. It is a filter over a universe, not a forecast of prices, and the documents are explicit about which of those two claims the evidence supports.

## papers/

`geis-system-description.md` covers what the screener measures, how the thirteen sub-scores get combined, what happens when data is missing, and why I made the calls I made. It's the reference document everything else points at, and its changelog is where any change to the method shows up.

`geis-fidelity-controls.md` and its PDF are the working paper. It describes a control that rebuilds every stored number from raw data and compares it against what the system wrote, three defects that control surfaced and how each was found, and a first measurement of predictive content. Section 8 is the part worth reading: it sets out what a measurement like that can and cannot support, and why the answer was to change nothing.

## track-record/

One CSV per week, committed on the Friday it refers to.

Each file has the top 10 by score at that Friday's close, including the reference price, so anyone can check later what the list did rather than taking my word for it. Next to it sits a `universe-*.csv` with every company scored that day, which exists so nobody has to wonder what I left out.

The rules are in `track-record/README.md`. They went up before the first file did.

## notes/

One file per week, in English, on what the screener produced: what each company in the top ten does, the seven CAN SLIM criteria and the six quality factors with the magnitude behind each one and not the sub-score alone, and which factors came back with no data at all.

These notes are not the record. The record is `track-record/`, and performance is computed from those CSVs and from nothing else. The separation is deliberate: a record you can reinterpret afterwards is not a record, so the two live in different folders and the rule is written here rather than remembered.

What is not in them: price targets, forecasts, or a view on whether anything is worth owning.

## Version

The screener is at version 1.0, dated 27 September 2026, and everything in `track-record/` was produced under it. The specification and the reasoning behind it are in `papers/geis-system-description.md`; when the method changes materially, a new version goes up with a dated changelog so any past ranking can be read against the specification that produced it.

The working paper on the fidelity controls was written against an earlier build of the system, which carried a fourteenth factor scoring valuation. Its scope note says so at the top and its addendum sets out what changed.

## Scope

What this repository holds is the method and the output. The screener's source code and its database stay private.

## Disclaimer

I'm an independent researcher. Not an investment firm, not an adviser, not a fund, and not authorised to give investment advice.

Everything here is educational. It describes a system and what it produces. It is not investment advice, not a personal recommendation, and not a solicitation.

I make no claims about past or future returns. The system ranks companies against disclosed criteria; it does not forecast prices, and no edge has been demonstrated or is claimed. The working paper reports the one measurement made so far and sets out what it does and does not support.

Equity investment carries risk of total loss. Nothing here takes account of anyone's circumstances or objectives.
