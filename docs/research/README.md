# Research

This directory holds evidence notes that inform the 401(k) model without silently turning every published result into a production signal.

## Current review

- [Second pass: 12 papers on behavior, trend, execution, diversification, liquidity, and tails](12-paper-second-pass.md)
- [Implementation spec derived from the 12-paper review](implementation-spec.md)

## Evidence roles

The papers are tagged by the job they can legitimately do here:

| Role | Meaning |
| --- | --- |
| **Signal evidence** | Direct empirical evidence for a rule close enough to test in this repo. |
| **Execution / turnover control** | Changes how a signal should become a trade, not whether the signal exists. |
| **Benchmark / research control** | A test the model should have to beat. |
| **Risk architecture** | A failure mode the backtest or stress suite should explicitly measure. |
| **Behavioral guardrail** | A reason to precommit rules and audit manual overrides. |
| **Context only** | Interesting evidence from a different market/frequency that should not be imported as a feature without a new test. |

## Research doctrine

1. Use only information available at the decision timestamp. A month-end signal belongs to the next executable period, never the month that produced it.
2. Keep **signal**, **target**, and **trade** separate. A forecast can be valid while an immediate full-position switch is not.
3. Compare every sophisticated rule with simple baselines: buy-and-hold, equal weight where meaningful, and a simple trend rule.
4. Report turnover, drawdown, recovery time, tail behavior, and results after plausible frictions. Sharpe alone is not an acceptance criterion.
5. Prefer walk-forward or otherwise genuinely out-of-sample evaluation. Parameter sweeps are diagnostics for stability, not a license to select the prettiest backtest.
6. Treat findings from futures, FX, commodities, or daily text data as **context** until the mechanism and implementation survive a 401(k)-appropriate test.
7. Do not infer that these papers validate the repo's HMM. The HMM remains a separate hypothesis that must earn its place against simpler rules.
8. Do not infer that futures trend evidence automatically validates daily-reset 3x ETFs such as TQQQ or SQQQ. Leverage, path dependence, financing, and rebalance drag require separate tests.

The original PDFs are linked from the notes rather than copied into this public repository.
