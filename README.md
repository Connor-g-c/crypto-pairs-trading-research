# Cryptocurrency Pairs Trading Research

A quantitative research project examining whether cointegration and rolling z-score signals can be used to construct a cryptocurrency pairs trading strategy from historical Binance market data.

The project began as an extension of Chee-Foong's cryptocurrency pairs trading implementation and the Binance data-retrieval approach published by Peter Nistrup. It expands the original framework through a broader cryptocurrency universe, correlation screening, cointegration testing, parameter evaluation and additional trade diagnostics.

> **Research status:** Exploratory academic backtest. The results are not evidence of a production-ready or directly tradeable strategy.

## Research Question

Can a statistically related pair of cryptocurrencies be traded using deviations in its relative price ratio, with opposing positions opened when the ratio moves away from a rolling equilibrium and closed as it converges?

The research process is:

1. Retrieve and align historical daily cryptocurrency prices from Binance.
2. Screen candidate pairs using shared history, correlation and cointegration tests.
3. Calculate short- and long-term moving averages of the selected pair's price ratio.
4. Standardise the divergence as a rolling z-score.
5. Open opposing positions when the z-score crosses an entry threshold.
6. Close the position when the z-score returns towards zero.
7. Compare parameter combinations using the historical training sample.

## Repository Structure

```text
crypto-pairs-trading-research/
├── notebooks/
│   ├── 01_binance_data_collection.ipynb
│   └── 02_crypto_pairs_trading_strategy.ipynb
├── data/
│   └── README.md
├── reports/
│   └── Crypto_Pairs_Trading_Academic_Report.pdf
├── requirements.txt
├── .gitignore
└── README.md
```

## Notebooks

### 01 - Binance Data Collection

[`01_binance_data_collection.ipynb`](notebooks/01_binance_data_collection.ipynb)

Retrieves historical cryptocurrency prices, standardises timestamps and combines individual series into an aligned price matrix.

### 02 - Cryptocurrency Pairs Trading Strategy

[`02_crypto_pairs_trading_strategy.ipynb`](notebooks/02_crypto_pairs_trading_strategy.ipynb)

Screens candidate pairs, evaluates cointegration, engineers rolling ratio and z-score signals, simulates trades and searches across moving-average parameters.

## Academic Report

The [`public academic report`](reports/Crypto_Pairs_Trading_Academic_Report.pdf) provides the wider financial-modelling context, methodology, results and discussion.

University administrative pages and personal candidate identifiers have been removed from the public version.

## Methodology

### Pair Selection

Candidate cryptocurrencies are aligned over a common historical period. The analysis uses correlation as an initial screening tool before applying cointegration tests to identify pairs whose relative relationship may be more stable through time.

The original BTC/ETH specification did not provide sufficient evidence of cointegration at the 5% level. The expanded analysis subsequently identified BTC/XRP as a stronger candidate within the tested universe.

### Signal Construction

For Coin 1 and Coin 2, the relative price ratio is:

```text
Ratio_t = Price_Coin1,t / Price_Coin2,t
```

The rolling signal is:

```text
Z_t = (Near_MA_t - Far_MA_t) / Rolling_Standard_Deviation_t
```

Where:

- `Near_MA` represents the short-term moving average of the ratio.
- `Far_MA` represents the longer-term moving average.
- The rolling standard deviation uses the longer window.

A positive z-score indicates that the short-term ratio is above its longer-term average, while a negative z-score indicates that it is below its longer-term average.

### Trading Rules

When no position is open:

- `Z < -1`: buy Coin 1 and short Coin 2.
- `Z > 1`: short Coin 1 and buy Coin 2.

When a position is open:

- `|Z| < 0.75`: close both legs.

The second leg is sized using the contemporaneous price ratio, producing approximately equal initial notional exposure across the two positions.

### Parameter Search

The implementation evaluates combinations of:

- Near moving-average windows from 5 to 14 observations.
- Far moving-average windows from 50 to 89 observations.

The selected parameters maximise a custom score based on final realised P&L relative to the worst recorded adverse open-position value:

```text
Score = Final P&L / Absolute Worst Recorded Adverse Position Value
```

This is not a Sharpe ratio and should not be interpreted as a complete measure of risk-adjusted performance.

## Selected Findings

- The original BTC/ETH spread failed the reported 5% cointegration test, weakening the theoretical basis for the initial strategy.
- Screening a broader cryptocurrency universe identified BTC/XRP with a reported cointegration p-value of `0.0146`.
- The optimised BTC/XRP specification produced a reported total return of `43.14%` in the academic implementation.
- Despite the improved headline return, the reported Sharpe ratio was only `0.018` and volatility remained extreme.
- The central conclusion is therefore not that the strategy reliably generates profit, but that pair selection, validation and risk controls materially affect apparent performance.

## Observations and Limitations

The results show that statistical significance during pair selection does not guarantee reliable trading performance.

A pair may appear cointegrated over one historical period but fail to maintain the same relationship as market conditions change. The strategy should therefore be treated as an exploratory research model rather than a consistent or immediately deployable source of profit.

### Pair Selection and Training Period

The selected pair depends heavily on the historical period used for cointegration testing.

Using a longer training period may help identify relationships that have persisted across different market conditions. However, a relationship that was stable historically may still weaken or disappear in the future.

Pair selection should therefore be repeated through time using rolling or expanding training windows rather than relying on a single fixed sample.

### Sampling Frequency

The sampling frequency used for the price series can materially affect the results.

A pair that appears cointegrated using minute-level observations may not appear cointegrated using daily data, and vice versa.

Higher-frequency data provide more observations and potentially more trading signals, but also introduce greater market noise, transaction costs and execution risk. More frequent trading opportunities do not necessarily translate into stronger net performance.

### Backtesting and Out-of-Sample Evaluation

The strategy is currently evaluated using a limited number of historical training and testing periods.

A more robust assessment would apply the research process repeatedly through time:

1. Identify candidate pairs using historical training data.
2. Select parameters using a separate validation period.
3. Evaluate the fixed strategy on unseen test data.
4. Roll the estimation window forward and repeat the process.

This walk-forward approach would provide a more realistic assessment of whether the strategy remains effective under changing market conditions.

### Divergence, Adverse Excursion and Holding Period

Pairs trading aims to reduce broad market exposure by taking opposing positions. It does not eliminate risk.

Losses can arise when the relative-price relationship continues to diverge rather than returning towards its historical average. This may occur because:

- The estimated relationship was temporary.
- The cointegration assumption has weakened.
- An asset-specific event has affected one cryptocurrency.
- Market structure or liquidity conditions have changed.

The current implementation records the most negative value of the open spread position as an adverse-excursion measure. This is not the same as conventional portfolio-level peak-to-trough maximum drawdown.

Longer holding periods also increase exposure to funding costs, market events, liquidity changes and structural breaks in the relationship.

### Stability of Cointegration

Cointegrated relationships can change over time.

The strategy should not assume that a pair selected during the training period will remain suitable indefinitely. Cointegration should be reassessed periodically, and pairs should be removed when the statistical relationship no longer meets the required criteria.

Rolling cointegration tests and parameter-stability analysis could help detect when a relationship is weakening.

### Position Sizing and Hedge Ratio

The current strategy trades one unit of Coin 1 and sizes Coin 2 using the contemporaneous price ratio:

```text
n_t = Price_Coin1,t / Price_Coin2,t
```

This produces approximately equal initial notional exposure across the two legs, but it is not necessarily the optimal statistical hedge ratio.

Possible improvements include:

- Estimating the hedge ratio using regression.
- Updating the hedge ratio through a rolling window.
- Scaling positions according to volatility.
- Limiting gross and net portfolio exposure.
- Sizing trades as a proportion of available capital.
- Respecting exchange minimum quantities and rounding requirements.

Introducing an explicit portfolio value would also allow returns to be reported as percentages rather than only as absolute profit and loss.

### Transaction Costs and Execution

The current backtest does not include:

- Exchange fees.
- Bid-ask spreads.
- Slippage.
- Shorting or borrowing costs.
- Perpetual-futures funding payments.
- Liquidity constraints.

These costs could materially reduce performance, particularly where signals generate frequent trades or positions remain open for long periods.

Future testing should report both gross and net strategy performance.

### Risk Controls

The extended BTC/XRP strategy produced improved total returns but remained highly volatile. This indicates that pair selection and parameter optimisation alone are not sufficient to control risk.

Potential additions include:

- Stop-loss rules.
- Maximum holding periods.
- Volatility-adjusted position sizing.
- Limits on exposure to individual assets.
- Filters that suspend trading during unstable market periods.
- Dynamic entry and exit thresholds.

These controls should be selected using training and validation data rather than adjusted after observing test-period losses.

### Trading Multiple Pairs

The framework could be extended to trade several cointegrated pairs simultaneously.

Each pair could use its own moving-average windows, thresholds and hedge ratio. The strategies would then need to be combined within a portfolio subject to:

- Maximum gross exposure.
- Maximum net exposure.
- Asset-level concentration limits.
- Correlation between pair strategies.
- Available capital and liquidity constraints.

Trading multiple pairs may improve diversification, but it can also create hidden concentration where the same cryptocurrency appears in several trades.

## Potential Extensions

Future development could include:

- Rolling or expanding-window pair selection.
- False-discovery-rate adjustment across cointegration tests.
- Regression-based or dynamic hedge-ratio estimation.
- Transaction costs, funding rates and slippage.
- Daily mark-to-market portfolio accounting.
- Stop-loss and maximum-holding-period rules.
- Volatility-adjusted position sizing.
- Portfolio-level gross, net and concentration constraints.
- Simultaneous trading across several independently validated pairs.

## Installation

Clone the repository:

```bash
git clone https://github.com/Connor-g-c/crypto-pairs-trading-research.git
cd crypto-pairs-trading-research
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate the environment.

### Windows

```bash
.venv\Scripts\activate
```

### macOS or Linux

```bash
source .venv/bin/activate
```

Install the required packages:

```bash
pip install -r requirements.txt
```

Launch Jupyter Notebook:

```bash
jupyter notebook
```

Run the notebooks in numerical order.

## Data and Credentials

Historical cryptocurrency market data are retrieved from Binance for research and educational purposes.

The full generated dataset is not required to be stored in the repository.

Never commit API keys, secrets or other credentials. Where authentication is required, credentials should be loaded from local environment variables and excluded through `.gitignore`.

## Attribution

This project was inspired by and initially adapted from:

- [Chee-Foong (EdgeTrader), Pairs Trading Using Co-integrated Cryptocurrency Pairs](https://github.com/edgetrader/crypto-trading/tree/master)
- [Peter Nistrup, Retrieving Full Historical Data for Every Cryptocurrency on Binance and BitMEX Using the Python APIs](https://medium.com/swlh/retrieving-full-historical-data-for-every-cryptocurrency-on-binance-bitmex-using-the-python-apis-27b47fd8137f)
- [Auquan, Pairs Trading Using Data-Driven Techniques](https://medium.com/auquan/pairs-trading-data-science-7dbedafcfe5a)

The notebooks document the extensions, interpretation and limitations introduced in this version.

## Disclaimer

This repository is provided for research, education and portfolio demonstration only.

It does not constitute investment advice, and the historical results should not be interpreted as expected future performance.
