# Implementation spec from the 12-paper review

Status: **research plan, not production trading logic**.

The purpose of this document is to convert the literature review into falsifiable changes to the backtest and reporting stack. It deliberately keeps new ideas behind experiments until they beat simpler controls.

## 1. Freeze the decision timestamp

Every monthly strategy should use one common convention:

1. Observe data through the final close of month (t).
2. Compute all features and state probabilities using data available at that close.
3. Generate the decision after the close.
4. Apply the new position only at the first modeled executable price in month (t+1).
5. Attribute no return from month (t) to a signal that required month (t)'s closing data.

For the HMM, only forward-filtered probabilities are admissible. Smoothed state probabilities are diagnostic-only because they use future observations.

Add unit tests that intentionally perturb future prices and confirm historical signals do not change.

**Paper basis:** Lo–Mamaysky–Wang; Tetlock.

---

## 2. Establish a benchmark ladder before adding features

Every experiment should report the same baselines:

| ID | Baseline |
| --- | --- |
| B0 | Buy and hold the stock sleeve |
| B1 | Equal-weight all-weather sleeve where the eligible menu makes this meaningful |
| B2 | 10-month price / moving-average rule |
| B3 | 12-month sign-of-total-return momentum |
| B4 | HMM regime rule |
| B5 | HMM + trend combination |

Any optimizer or new overlay has to beat the relevant simpler strategy **out of sample**, not merely improve the in-sample fit.

For the all-weather sleeve, 1/N is a benchmark, not a recommendation. For the market-timing sleeve, buy-and-hold and single-rule trend are the primary naive controls.

**Paper basis:** DeMiguel–Garlappi–Uppal; Moskowitz–Ooi–Pedersen.

---

## 3. Add a 12-month TSMOM replication that matches the repo frequency

Start with the least-parameterized version:

$
m_t = \operatorname{sign}\left(\frac{P_t}{P_{t-12}}-1\right).
$

At month-end (t):
- (m_t>0): risk-on for month (t+1);
- (m_t<0): defensive for month (t+1).

Use total-return series when possible.

Test three variants separately:

**M0: sign only.** No volatility targeting.

**M1: sign + capped volatility scaling.** Estimate volatility using only prior data. Cap leverage at 1x for the unlevered research version so volatility targeting is not confounded with leverage.

**M2: sign + existing sleeve mechanics.** Only after M0/M1 are understood, map the signal onto the repo's actual instruments.

The original TSMOM paper targets each futures position to 40% ex-ante volatility before equal-weighting across 58 contracts. Do **not** copy that target into this 401(k) project: the instrument set and leverage mechanics are different.

**Paper basis:** Moskowitz–Ooi–Pedersen.

---

## 4. Separate score, target, and trade

The code should represent these as different objects or columns:

- **score**: model evidence, e.g. HMM stressed probability or trend distance;
- **target**: desired risk exposure implied by the score;
- **trade**: actual change from current position to target after turnover controls.

This prevents a change in model confidence from silently becoming an all-in/all-out order.

### Partial-adjustment experiment

For a continuous target (a_t), test

$
x_t=(1-\kappa)x_{t-1}+\kappa a_t,
$

with a small predeclared grid such as (kappa\in\{0.25,0.5,1.0\}). The point of the grid is stability analysis, not selecting the best backtest point.

### Hysteresis experiment

For the HMM stressed probability (p_t), compare:
- single threshold: risk-off if (p_t>0.5);
- two-threshold band: exit risk-on only above an upper threshold and re-enter only below a lower threshold.

Threshold pairs must be fixed before the final evaluation window. Report how much apparent improvement comes merely from reducing turnover.

**Paper basis:** Gârleanu–Pedersen; Barber–Odean.

---

## 5. Make turnover a first-class metric

For every strategy report:

$
\text{one-way turnover}_t = \frac12\sum_i |w_{i,t}-w_{i,t-1}|.
$

Also report:
- switches per calendar year;
- median and maximum holding period;
- fraction of switches reversed within 1, 2, and 3 months;
- gross return;
- friction-adjusted return.

### Friction sensitivity

Do not pretend one cost estimate is "the" answer. Run a simple sensitivity grid of round-trip equivalent costs, for example 0, 5, 10, 25, and 50 bp, plus a separate crisis-stress scenario.

These are research stress assumptions, not claims about the user's actual plan costs. If plan-specific exchange restrictions or redemption fees are later modeled, keep them in a separate configuration file.

**Paper basis:** Barber–Odean; Gârleanu–Pedersen; Tetlock.

---

## 6. Add crisis execution stress tests

A backtest that assumes identical execution in March 2020 and a quiet month is too optimistic for leveraged exposures.

For each timing strategy rerun with synthetic adverse execution on months meeting predeclared stress definitions, such as:
- top-decile realized volatility;
- monthly gap/return beyond a fixed percentile;
- HMM stressed probability above a fixed threshold.

Stress scenarios should combine, rather than isolate:
- worse execution price/slippage;
- delayed execution by one additional modeled period;
- lower allowable leveraged exposure.

The point is not to forecast a particular spread. It is to test whether the strategy's conclusion survives plausible nonlinear deterioration.

**Paper basis:** Brunnermeier–Pedersen.

---

## 7. Add dependence diagnostics beyond Pearson correlation

For each pair of sleeves/strategies report:

- Pearson correlation;
- Spearman rank correlation;
- correlation conditional on the stock sleeve being in its worst 10% of months;
- average defensive-sleeve return in those worst stock months;
- fraction of stock drawdown months in which the defensive sleeve is also negative;
- maximum simultaneous drawdown.

A simple lower-tail co-exceedance statistic is useful:

$
C_q=P(R_A<Q_A(q),\ R_B<Q_B(q)),
$

evaluated at a fixed (q), e.g. 10%. Compare the observed joint frequency with the product of marginal frequencies as a sanity check.

Do not fit a complicated copula unless these simple diagnostics reveal a decision-relevant gap that justifies the extra parameter risk.

**Paper basis:** Embrechts–McNeil–Straumann; DeMiguel–Garlappi–Uppal.

---

## 8. Measure trend's crisis behavior by path, not label

Create event studies around large equity drawdowns. For each event store:
- date of prior peak;
- date trend rule first became defensive;
- drawdown already incurred at that signal;
- maximum drawdown after the signal;
- date of re-entry;
- missed rebound after re-entry lag;
- recovery date.

This directly tests the Dao et al. distinction between **long-horizon convexity** and **instantaneous gap insurance**.

At minimum, break out the 2000–02 bear market, 2008, 2020, and 2022 in the current 1997+ sample.

**Paper basis:** Dao et al.; Moskowitz–Ooi–Pedersen.

---

## 9. Keep leveraged ETF validation separate

The literature set contains strong evidence on futures trend, but it does not validate daily-reset leveraged ETFs.

Before TQQQ/SQQQ results are treated as evidence for the regime model, add a separate report containing:

- unlevered benchmark using QQQ or a Nasdaq-100 total-return proxy;
- 3x ETF result over the live ETF history;
- a synthetic 3x daily-reset series over earlier periods, clearly labeled synthetic;
- realized volatility drag relative to three times the unlevered cumulative return;
- gap/crash sensitivity;
- result with no short/inverse sleeve, to separate timing skill from inverse-leverage mechanics.

A model should first demonstrate value on the underlying risk exposure. Leverage should amplify an already-defensible process, not rescue a weak signal.

**Paper basis:** Brunnermeier–Pedersen for nonlinear stress; transfer-limit warning from Moskowitz–Ooi–Pedersen and Dao et al.

---

## 10. Treat alternative data as quarantined research

### Media sentiment

Do not add a generic sentiment API to the core feature set because Tetlock found a daily text effect. A valid replication would need:
- a precisely timestamped corpus;
- a frozen text model/dictionary;
- training/factor estimation using only past data;
- a monthly aggregation rule fixed before the test;
- incremental predictive value beyond returns, trend, volatility, and the HMM;
- turnover after the added feature.

### Commodity inventory / futures basis

For the USO sleeve, inventory or curve information may eventually be tested as a commodity-specific overlay. It should not modify the equity HMM unless a separate economic and empirical case is established.

### FX carry

No FX carry feature belongs in the current 401(k) model. The paper is useful as an example of combining strategies and diagnosing hidden crash exposure.

**Paper basis:** Tetlock; Gorton–Hayashi–Rouwenhorst; Burnside–Eichenbaum–Rebelo.

---

## 11. Add a manual-override audit

Because trades are placed manually, every deviation from a generated allocation should be logged:

| Field | Purpose |
| --- | --- |
| timestamp | Establish what information was available |
| model version / git SHA | Reconstruct the recommendation |
| generated target | What the system said |
| executed allocation | What was actually done |
| current P&L since last switch | Detect disposition-effect behavior |
| override reason | Human-readable hypothesis |
| planned review date | Prevent indefinite "temporary" overrides |

The analysis should later compare override outcomes with rule-following counterfactuals. Do not use cost basis as a model feature unless it has a predeclared economic purpose.

**Paper basis:** Odean; Barber–Odean.

---

## 12. Acceptance gates for future model changes

A proposed change should not enter the default strategy merely because it raises CAGR. It should satisfy a written set of gates.

### Required evidence

- no look-ahead under an automated future-data perturbation test;
- improvement survives a walk-forward or untouched evaluation period;
- result is not concentrated in one decade/event;
- performance is shown against the relevant simple benchmark;
- turnover and friction sensitivity are reported;
- max drawdown and recovery time are reported;
- lower-tail co-movement is reported for diversified sleeves;
- parameter perturbations show a plateau rather than a single lucky optimum;
- unlevered result is reported before leveraged implementation;
- effect has a plausible mechanism or a strong cross-market empirical precedent.

### Default decision rule

If a more complex model is statistically ambiguous relative to a simpler one, keep the simpler one.

That is the common implication of this literature set: estimation error, trading frictions, behavior, and crisis dependence all punish unnecessary complexity.

---

## Suggested implementation order

1. Timestamp/unit-test harness.
2. Standard benchmark report.
3. 12-month sign momentum replication.
4. Turnover/friction metrics.
5. HMM hysteresis / partial-adjustment experiment.
6. Tail-dependence dashboard.
7. Crisis event-study report.
8. Leveraged-ETF validation report.
9. Manual-override log schema.
10. Only then consider quarantined alternative-data research.

This order intentionally improves the **research process** before expanding the feature set.
