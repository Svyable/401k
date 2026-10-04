# Twelve papers — second pass

This review extracts the pieces that can be falsified or implemented: equations, portfolio construction, timing conventions, parameter choices, empirical tests, and failure modes. It is deliberately stricter than a summary. A result is not promoted into the 401(k) model merely because it is statistically interesting.

## 1. Odean — *Are Investors Reluctant to Realize Their Losses?*

**Role:** behavioral guardrail.

**Question.** Do investors sell winners more readily than losers, even when taxes, rebalancing, or subsequent returns do not justify the asymmetry?

**Construction.** Odean defines

[
PGR = \frac{\text{realized gains}}{\text{realized gains}+\text{paper gains}},\qquad
PLR = \frac{\text{realized losses}}{\text{realized losses}+\text{paper losses}}.
]

The core test is whether PGR exceeds PLR. The data cover 10,000 discount-brokerage accounts.

**Key result.** For the full year, PLR is about 0.098 and PGR about 0.148; the difference is highly significant. The pattern reverses in December, consistent with tax-loss selling. Account-, share-, and dollar-weighted robustness checks preserve the basic disposition effect.

**Transfer to this repo.** The December tax mechanism is largely irrelevant inside a tax-advantaged 401(k), but the behavioral mechanism is directly relevant to manual switching. Entry price and current P&L must not enter an override decision unless they are explicit model variables. A rule should not become harder to exit simply because it is losing.

**Research control to add.** Log every discretionary override with the model state, proposed trade, current unrealized P&L, and reason. Later test whether overrides conditioned on losses are systematically different from overrides conditioned on gains.

**Do not infer.** The paper does not provide an alpha signal and does not say "always cut losses."

Source: Terrance Odean, 1998, *Journal of Finance*.  
PDF: https://faculty.haas.berkeley.edu/odean/papers%20current%20versions/areinvestorsreluctant.pdf

---

## 2. Brunnermeier & Pedersen — *Market Liquidity and Funding Liquidity*

**Role:** risk architecture.

**Question.** How do traders' funding constraints interact with market liquidity?

**Mechanism.** Traders supply market liquidity using scarce capital. Margins/haircuts determine how much capital a position consumes. When losses reduce capital or margins rise, constrained traders cut positions. That can move prices farther from fundamentals, worsen volatility, raise margins again, and create a feedback loop.

The paper's important state variable is not merely price volatility. It is the interaction of market illiquidity, margin requirements, and the shadow cost of funding.

**Key result.** The model produces both a **loss spiral** and a **margin spiral**. Liquidity can become fragile exactly when capital is scarce, and common funding conditions can create commonality in liquidity across otherwise different assets.

**Transfer to this repo.** This is especially relevant because the README contemplates TQQQ/SQQQ. A daily-reset 3x ETF is not a futures position financed exactly as in the paper, but it is still a leveraged exposure whose behavior deteriorates in volatile, discontinuous markets. Backtests should not assume crisis execution is identical to calm execution.

**Research controls to add.**
- Report results under crisis slippage/spread shocks, not just a constant cost.
- Forbid an increase in leveraged exposure in synthetic stress tests unless the rule still passes after adverse execution assumptions.
- Report performance conditioned on high realized volatility and large gaps.
- Keep an unlevered version of every timing rule as a benchmark.

**Do not infer.** The paper is not a VIX trading rule and does not supply a threshold for switching a 401(k).

Source: Markus Brunnermeier and Lasse Heje Pedersen, 2009, *Review of Financial Studies*.  
NBER: https://www.nber.org/papers/w12939  
Author PDF: https://markus.scholar.princeton.edu/document/98

---

## 3. Lo, Mamaysky & Wang — *Foundations of Technical Analysis*

**Role:** research control / signal-methodology evidence.

**Question.** Can chart patterns be converted from visual stories into objective statistical events?

**Construction.** Prices are smoothed with nonparametric kernel regression. Local extrema in the smoothed path define patterns such as head-and-shoulders, double tops/bottoms, and related formations. In the main implementation the rolling pattern window uses (l=35) and lag (d=3), producing a 38-trading-day window. The lag is important: it prevents a detected pattern from using the same observations whose subsequent return is being evaluated.

The paper compares conditional and unconditional return distributions using Kolmogorov-Smirnov tests and bootstrapping rather than declaring a pattern useful because a fitted chart looks persuasive.

**Data/test design.** Daily CRSP NYSE/AMEX and Nasdaq data from 1962–1996 are split into subperiods. Stocks are sampled across market-cap quintiles. Post-pattern returns begin after the detection lag.

**Key result.** Several patterns change the conditional return distribution, with stronger evidence in the Nasdaq sample. The authors explicitly do **not** equate statistical predictability with a profitable trading strategy after costs.

**Transfer to this repo.** The important contribution is procedural:
- Define the 10-month moving-average and 12-month momentum rules as exact functions.
- Freeze the timestamp convention.
- Separate pattern detection from the future return window.
- Bootstrap or otherwise quantify whether conditional outcomes differ from the baseline.
- Reject visual re-labeling of a failed period after the fact.

**Do not infer.** This paper does not validate arbitrary chart reading, and its 38-day pattern machinery is not a reason to add short-horizon chart patterns to a monthly 401(k) model.

Source: Andrew Lo, Harry Mamaysky, Jiang Wang, 2000, *Journal of Finance*.  
PDF: https://web.mit.edu/people/wangj/pap/LoMamayskyWang00.pdf

---

## 4. Barber & Odean — *Trading Is Hazardous to Your Wealth*

**Role:** turnover control / behavioral guardrail.

**Question.** Does more household trading produce better net investment results?

**Data.** 66,465 households at a large discount broker, 1991–1996.

**Key result.** The most active households earned about 11.4% net annually while the market earned about 17.9%; the least active group earned about 18.5%. The average household turned over roughly 75% of the portfolio per year. In that era, a typical large round trip could cost roughly 3% in commissions plus about 1% in bid-ask spread.

**Important distinction.** The historical commission numbers are not portable to today's liquid ETFs or plan funds. The structural lesson is: a new trade starts with a burden of proof. Gross signal quality is not net strategy quality.

**Transfer to this repo.**
- Count switches and one-way turnover.
- Run every strategy with a no-trade baseline.
- Show gross and friction-adjusted results.
- Require an incremental switch to clear a predefined hurdle rather than reacting to every tiny change in a continuous score.
- In a 401(k), ignore taxable-account capital-gains costs but keep spreads, plan restrictions, stale NAV timing, and behavioral costs.

**Do not infer.** "Trading is harmful" does not mean a low-frequency risk-control rule is automatically harmful; it means the rule must earn its turnover.

Source: Brad Barber and Terrance Odean, 2000, *Journal of Finance*.  
DOI: https://doi.org/10.1111/0022-1082.00226

---

## 5. Gârleanu & Pedersen — *Dynamic Trading with Predictable Returns and Transaction Costs*

**Role:** execution / turnover control.

This is the most directly implementable execution paper in the set.

**Model.** Expected returns depend on predictors (f_t):

[
r_{t+1}=Bf_t+u_{t+1},
]

and predictors mean-revert dynamically. Trading a change (Delta x_t) incurs quadratic cost

[
TC(\Delta x_t)=\tfrac12\Delta x_t'\Lambda\Delta x_t.
]

The dynamic solution separates the current position, an **aim portfolio**, and the speed of trading toward that aim.

A useful simplified representation is

[
x_t=(1-\kappa)x_{t-1}+\kappa\,\text{aim}_t,\qquad 0<\kappa<1,
]

where (kappa) falls as trading costs rise and changes with risk aversion and predictability. The paper's "aim in front of the target" result can be represented as a weighted average of current and expected future Markowitz targets:

[
\text{aim}_t=z\,\text{Markowitz}_t+(1-z)E_t[\text{aim}_{t+1}],
]

so more persistent predictors receive more weight.

**Empirical illustration.** Their commodity example includes predictors based on roughly five-day, 12-month, and five-year past returns, deliberately spanning fast and slow decay. The optimized dynamic strategy improves net performance relative to static reactions to the signals.

**Transfer to this repo.** The exact continuous quadratic-cost solution does not map one-for-one onto a discrete 401(k) menu. The principle does:
- a score is not a target;
- a target is not necessarily an immediate full switch;
- persistent signals justify more movement than ephemeral signals;
- hysteresis or partial adjustment should be tested against all-in/all-out switching.

For a monthly manual system, practical approximations are (a) partial adjustment, (b) a no-trade band around the current regime probability, or (c) confirmation persistence before a full switch. These are hypotheses to test, not parameters supplied by the paper.

**Do not infer.** The paper does not tell us the optimal (kappa) for a 401(k), and fitting (kappa) on the same sample used for evaluation would simply move the overfitting problem.

Source: Nicolae Gârleanu and Lasse Heje Pedersen, 2013, *Journal of Finance*.  
PDF: https://pages.stern.nyu.edu/~lpederse/papers/DynamicTrading.pdf

---

## 6. Tetlock — *Giving Content to Investor Sentiment: The Role of Media in the Stock Market*

**Role:** context only unless independently revalidated.

**Construction.** Tetlock applies the General Inquirer to the *Wall Street Journal* "Abreast of the Market" column from 1984–1999. Each day is represented by counts in 77 Harvard psychosocial dictionary categories. Principal-components analysis produces a first media factor strongly related to Negative, Weak, Fail, and Fall categories.

A particularly good anti-look-ahead detail: factor loadings for year (t) are estimated from year (t-1), then applied during year (t). The yearly factors are highly stable (average pairwise correlation about 0.96).

**Return test.** VARs include lags of market returns, volume, volatility controls, day-of-week effects, January, and the 1987 crash dummy. High pessimism predicts short-horizon downward pressure followed by reversal; extreme pessimism in either direction predicts high volume; weak market returns also predict more pessimistic language.

**Trading benchmark.** A deliberately simple rule uses the Negative-word measure relative to the prior year's distribution. Bottom-third days produce a one-day long Dow trade; top-third days produce a one-day short trade after the timing gap. The gross annualized return is reported as 7.3%, but the average edge is only about 4.4 bp per trade and the paper explicitly notes that impact/spread/tax frictions can eliminate it.

**Transfer to this repo.** The robust lesson is timestamp discipline for alternative data. The daily 1984–1999 text effect is a poor fit to monthly 401(k) switching and should not be added to the core model without a modern, out-of-sample replication that shows incremental value over price, trend, and volatility.

**Do not infer.** A generic modern sentiment score is not the same variable, and a relationship at daily frequency need not survive monthly aggregation.

Source: Paul Tetlock, 2007, *Journal of Finance*.  
PDF: https://www.columbia.edu/~pt2238/papers/Tetlock_Media_Sentiment_JF.pdf

---

## 7. Moskowitz, Ooi & Pedersen — *Time Series Momentum*

**Role:** direct signal evidence, with market/instrument caveats.

**Data.** 58 liquid instruments: 24 commodity futures, 12 currency pairs, nine developed equity-index futures, and 13 government-bond futures. The broad raw data span 1965–2009; the main diversified factor is evaluated from 1985–2009 to ensure broad, liquid coverage.

**Canonical strategy.** For instrument (s), use the sign of its own trailing 12-month excess return and scale the position to 40% ex-ante annualized volatility:

[
r^{TSMOM,s}_{t,t+1}
=
\operatorname{sign}(r^s_{t-12,t})
\frac{40\%}{\sigma^s_t}
r^s_{t,t+1}.
]

The diversified factor equal-weights the resulting strategy returns across instruments available at time (t). The 40% instrument target is a normalization choice; diversification brings the realized portfolio volatility to roughly 12% annually in the 1985–2009 sample.

**Key result.** Own past returns predict future returns over roughly one to 12 months and partially reverse at longer horizons. All 58 contracts have positive 12-month TSMOM returns; 52 are statistically positive at the 5% level. The diversified factor has an annual Sharpe above one in the reported sample and performs particularly well during large market moves.

**Transfer to this repo.** The existing 12-month momentum rule has a strong literature analogue. The appropriate replication here is intentionally simpler:
- month-end total return over the prior 12 months;
- decision made only after that month-end close;
- next executable period receives the position;
- compare sign-only momentum with the 10-month moving-average rule and HMM;
- test optional volatility scaling separately, because it changes the strategy.

**Critical caveat.** The paper studies liquid futures/forwards, not daily-reset 3x ETFs. Its result does not establish that TQQQ/SQQQ timing has the same return distribution or crisis behavior.

Source: Tobias Moskowitz, Yao Hua Ooi, Lasse Heje Pedersen, 2012, *Journal of Financial Economics*.  
PDF: https://pages.stern.nyu.edu/~lpederse/papers/TimeSeriesMomentum.pdf

---

## 8. Embrechts, McNeil & Straumann — *Correlation and Dependence in Risk Management: Properties and Pitfalls*

**Role:** risk architecture.

**Question.** When is ordinary linear correlation a safe description of dependence?

**Core point.** Pearson correlation is natural for multivariate normal and, more broadly, elliptical models. Outside that world it can be a poor summary of joint risk. Dependence and marginal distributions can be separated with a copula; rank measures such as Kendall's tau and Spearman correlation are functions of dependence structure and are invariant to strictly increasing transformations in ways Pearson correlation is not.

**Why it matters.** Two assets or strategies can have modest full-sample correlation and still fail together in the states we care about. "Diversified on average" is not equivalent to "diversified in a drawdown."

**Transfer to this repo.** Add non-Pearson and conditional diagnostics:
- Spearman rank correlation alongside Pearson.
- Correlation and beta during the worst 10% of stock months.
- Joint drawdown frequency and conditional loss of the defensive sleeve when stocks are down.
- A simple lower-tail co-exceedance table.
- Stress scenarios in which historically low-correlated sleeves move together.

These diagnostics are an implementation extension of the paper's dependence warning; they are not a claim that a particular copula family is optimal here.

**Do not infer.** Fitting a sophisticated copula to a short sample can create another estimation-error problem. The first objective is to expose dependence hidden by a single full-sample correlation.

Source: Paul Embrechts, Alexander McNeil, Daniel Straumann, 1999/2002 chapter version, *Risk Management: Value at Risk and Beyond*.  
PDF mirror: https://citeseerx.ist.psu.edu/document?doi=bda8f92ff000a7636e4e8879bfb77f5347894934&repid=rep1&type=pdf

---

## 9. Gorton, Hayashi & Rouwenhorst — *The Fundamentals of Commodity Futures Returns*

**Role:** mechanism evidence / context for the all-weather commodity sleeve.

**Data.** 31 commodity futures with physical-inventory data, 1969–2006.

**Mechanism.** The Theory of Storage predicts that scarcity raises the convenience yield. The paper documents a decreasing, nonlinear relationship between inventories and convenience yield. Futures basis, prior futures returns, and prior spot returns contain information about the inventory state and expected risk premium.

**Portfolio construction.** At each month end, commodities are split into halves using price signals such as basis or prior performance. Portfolios are equal-weighted, held for the next month, then re-sorted and rebalanced. Futures momentum is measured using the prior 12-month futures return; spot momentum uses the year-on-year spot-price move.

**Selected results.**
- The high-basis portfolio minus low-basis portfolio averages about 10.23% annualized in the full sample (t ≈ 3.73).
- High spot-momentum minus low spot-momentum averages about 13.85% annualized (t ≈ 4.95).
- The high-signal portfolios tend to contain lower-inventory commodities.
- The authors reject the simple Keynesian hedging-pressure story as the main determinant of these risk premiums.

**Transfer to this repo.** The useful lesson is causal discipline: a price signal is more believable when it can be tied to a slow-moving economic state. For the current USO sleeve, curve shape/inventory conditions may eventually be useful research variables, but this paper does not justify adding a commodity timing feature to the equity regime model.

**Do not infer.** Results for cross-sectional commodity-futures portfolios do not transfer directly to USO, whose roll mechanics and concentration differ materially.

Source: Gary Gorton, Fumio Hayashi, K. Geert Rouwenhorst, 2013, *Review of Finance*.  
Author PDF: https://depot.som.yale.edu/icf/papers/fileuploads/2605/original/07-08.pdf  
NBER: https://www.nber.org/papers/w13249

---

## 10. Burnside, Eichenbaum & Rebelo — *Carry Trade and Momentum in Currency Markets*

**Role:** context / factor-combination lesson.

**Carry construction.** A long foreign-currency position has payoff

[
z^L_{t+1}=(1+i_t^*)\frac{S_{t+1}}{S_t}-(1+i_t).
]

The carry trade chooses its sign from the interest differential:

[
z^C_{t+1}=\operatorname{sign}(i_t^*-i_t)z^L_{t+1}.
]

Under covered interest parity the same direction can be implemented using forwards.

**Momentum construction.**

[
z^M_{t+1}=\operatorname{sign}(z^L_t)z^L_{t+1},
]

so the prior month's currency return sets next month's direction. The portfolio equally weights the currency-level trades.

**Sample/result.** In the 1976–2010 sample of 20 currencies, the paper reports approximate annualized payouts/volatilities of 4.6%/5.1% for carry (Sharpe 0.89) and 4.5%/7.3% for momentum (Sharpe 0.62). A 50/50 combination has similar mean payout but lower volatility, producing a Sharpe near 0.98. Direction is correct only about 57% of the time for each standalone rule; diversification matters more than a spectacular hit rate.

**Interpretation.** Standard risk factors do not fully explain the returns. The paper considers disaster/peso explanations and price pressure. The 2008 episode is informative because carry performs badly while momentum performs well, making a simple "both are one hidden crash premium" story difficult.

**Transfer to this repo.** Two useful ideas transfer, not the FX signals themselves:
1. evaluate combinations because differently behaving weak predictors can be more useful together;
2. always ask what adverse state a strategy is implicitly short.

The HMM + trend combination therefore deserves evaluation as a portfolio of decision rules, but it must beat each component out of sample and after added turnover.

Source: Craig Burnside, Martin Eichenbaum, Sergio Rebelo, 2011, *Review of Financial Studies*.  
PDF: https://www.kellogg.northwestern.edu/faculty/rebelo/htm/carry.pdf

---

## 11. DeMiguel, Garlappi & Uppal — *Optimal Versus Naive Diversification*

**Role:** benchmark / research control.

**Question.** Do sophisticated estimated portfolio optimizers reliably beat equal weighting out of sample?

**Benchmark.** The naive portfolio allocates (1/N) to each available risky asset at each rebalance and requires no estimates of expected return or covariance.

**Models/test.** The paper compares 14 formulations/variants, including sample mean-variance, Bayes-Stein, data-and-model Bayesian portfolios, minimum variance, factor/missing-factor models, constrained versions, and mixtures. Seven empirical monthly datasets include US sectors/industries, international equity indexes, and Fama-French portfolio/factor sets. Estimation windows are generally 60 or 120 months; evaluation is out of sample and considers Sharpe, certainty-equivalent return, and turnover.

**Key result.** No optimizing model consistently beats 1/N across the datasets and criteria. Constraining portfolios helps turnover and stability, but not enough to make optimization reliably superior. Their calibration implies roughly 3,000 months of estimation data for a 25-asset sample mean-variance portfolio and more than 6,000 months for 50 assets to reliably clear the 1/N benchmark under their assumptions, versus the 60–120 months commonly used.

A vivid sensitivity example in the paper: with two nearly identical assets at 0.99 correlation, changing one estimated annual mean from 8% to 9% can drive unconstrained mean-variance weights to roughly +635% and -535%.

**Transfer to this repo.**
- The all-weather sleeve needs a simple equal-weight benchmark.
- The HMM/trend timing layer needs simple buy-and-hold and single-rule benchmarks.
- Any future optimizer must show stable weights and materially better out-of-sample behavior, not just a better in-sample frontier.
- Parameter count itself is a cost.

**Do not infer.** The authors explicitly use 1/N as a benchmark, not as a theorem that equal weighting is optimal.

Source: Victor DeMiguel, Lorenzo Garlappi, Raman Uppal, 2009, *Review of Financial Studies*.  
DOI: https://doi.org/10.1093/rfs/hhm075

---

## 12. Dao et al. — *Tail Protection for Long Investors: Trend Convexity at Work*

**Role:** direct mechanism evidence for trend; tail-risk architecture.

**Core identity.** Start with a deliberately simple trend position proportional to cumulative price change:

[
\Pi_t=\lambda A_t(S_t-S_0),
]

with (A_t=1) for the simple derivation and daily price change (D_t). The next-period gain is

[
G_t=\Pi_{t-1}D_t.
]

Aggregating and rearranging yields the key identity

[
G_T=\frac{\lambda}{2}
\left[(S_T-S_0)^2-\sum_{t=1}^{T}D_t^2\right].
]

In expectation, aggregate trend P&L is proportional to the difference between long-horizon and short-horizon realized variance. The paper extends the result to EMA-style trend filters. For an EMA implementation, averaged trend P&L can again be written as a long-timescale variance term minus a short-timescale variance term.

**Empirical illustration.** In one S&P 500 futures illustration the authors use a trend scale around (	au=180) trading days and choose the risk coefficient so daily P&L volatility is around 1%. The exact number is an illustration, not a universal optimal lookback.

**Interpretation.** Over the horizon on which the trend rule has time to establish a position, trend P&L becomes positively convex in the underlying long-horizon move. This helps explain why diversified CTAs can look like crisis protection.

**Critical limitation.** Trend is not the same as owning a put. A discontinuous gap can occur before the signal has moved the position. Convexity emerges over the strategy's reaction horizon and depends on path.

**Transfer to this repo.** This gives a mechanism for why the simple 10–12 month trend rules can reduce deep, persistent drawdowns. It also explains why a slow trend filter can fail at an abrupt crash and why the HMM/volatility state might complement rather than duplicate trend.

**Do not infer.** "Trend is convex" does not guarantee protection in every crash, and the paper's futures replication does not validate leveraged ETF path behavior.

Source: Tung-Lam Dao, Trung-Tu Nguyen, Cyril Deremble, Yves Lempérière, Jean-Philippe Bouchaud, Marc Potters, 2017, *Journal of Investment Strategies*.  
arXiv: https://arxiv.org/abs/1607.02410

---

# Cross-paper synthesis

The set is more useful as a research architecture than as a bag of signals.

**Signal layer.** Time-series momentum provides the cleanest direct evidence for an implementable trend rule. Lo–Mamaysky–Wang provides the discipline for defining and timestamping such a rule. Dao et al. gives a mechanism for its long-horizon convexity.

**Decision layer.** Gârleanu–Pedersen says that a forecast should not automatically become a full immediate trade. Barber–Odean says turnover must earn its costs. Odean says manual overrides can be systematically contaminated by the current gain/loss frame.

**Portfolio layer.** DeMiguel–Garlappi–Uppal forces every optimizer to clear a simple benchmark. Embrechts–McNeil–Straumann says full-sample correlation is insufficient evidence of diversification.

**Crisis layer.** Brunnermeier–Pedersen explains why funding/liquidity relationships can become nonlinear under stress. Dao explains why trend can be convex over its response horizon but still miss a gap.

**Mechanism/context layer.** Gorton–Hayashi–Rouwenhorst shows how price trends can encode a slow physical state. Tetlock shows how non-fundamental sentiment can move prices temporarily, with unusually strict attention to look-ahead. Burnside–Eichenbaum–Rebelo shows that combining differently behaving return sources can matter more than a high hit rate.

For this repository, the strongest next step is therefore not to add twelve features. It is to make the existing HMM/trend project harder to fool: cleaner timestamps, stronger baselines, explicit turnover, partial-adjustment variants, tail-dependence diagnostics, and crisis execution stress tests.
