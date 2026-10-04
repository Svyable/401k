# 401k

Regime-aware 401(k) research: a hidden Markov model (HMM) and simple market-timing
rules to make drawdowns shallower, plus interactive HTML visualizations of past and
expected returns. Trades are placed manually; nothing here connects to a brokerage
account.

> **Not financial advice.** This is personal research. Backtests are hypothetical,
> past drawdowns don't bound future ones, and leveraged ETFs can lose most of their
> value quickly.

## The model

The portfolio is split into two sleeves:

| Sleeve | Instruments | Idea |
| --- | --- | --- |
| Market-timing | TQQQ / SQQQ (3x long / 3x short Nasdaq-100) | Lean long in calm regimes, step aside or go short in stressed ones |
| All-weather | SPY, TLT, GLD, USO | Diversified core across stocks, long bonds, gold and oil |

**Regime detection.** A two-state Gaussian HMM on weekly log returns estimates the
probability the market is in a "calm" or "stressed" state. It is refit periodically
on prior data only, and signals use the forward-filtered probability (never the
smoothed one, which peeks at the future).

**Trend rules.** Classic month-end checks such as price vs. its 10-month moving
average, used alongside the HMM to confirm or overrule regime calls.

## Visualizations

Open any file in `viz/` directly in a browser; each page is self-contained.

- [`viz/shallower-holes.html`](viz/shallower-holes.html): five once-a-month 401(k)
  switching rules (including the HMM) tested on S&P 500 and bond index funds,
  1997 to 2026, with drawdown, recovery and regime charts.

## Layout

```
viz/    HTML visualization pages (self-contained, open in a browser)
src/    models and data-feed code (HMM, signals, backtests)
data/   local price caches (gitignored)
```

## Data feeds

Planned sources: the free Quantiacs API, tastytrade market data, and public fund
price history. Copy `.env.example` to `.env` and add your keys; `.env` is gitignored
and must never be committed.

## License

[MIT](LICENSE)
