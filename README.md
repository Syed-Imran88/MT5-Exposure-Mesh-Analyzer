![preview](https://raw.githubusercontent.com/Syed-Imran88/MT5-Exposure-Mesh-Analyzer/main/preview.svg)

# Quantique Synapse – Multi-Asset Cross-Exposure Matrix for MetaTrader

In the realm of algorithmic trading, the most expensive oversight is not a bad entry—it is a hidden overlap. **Quantique Synapse** is a real-time exposure intelligence engine for MetaTrader 5 and MetaTrader 4 that surfaces the invisible geometry of your portfolio. It maps every open position, every pending order, and every correlated pair onto a unified risk surface, revealing where your capital is secretly doubling down on the same underlying macro exposure.  

Traders often manage each chart in isolation, unaware that three separate trades on EURUSD, EURGBP, and EURCHF are effectively a massive leveraged bet on the Euro. **Quantique Synapse** dissolves this blindness by calculating a continuous **cross-pair correlation tensor**—a dynamic heatmap of every symbol relative to every other symbol, weighted by real-time lot sizes and margin requirements. The result is a **basket exposure fingerprint** that tells you, in plain numbers, how much of your account equity is actually exposed to USD, EUR, JPY, GBP, AUD, NZD, CAD, and CHF at any moment.

This repository provides the complete MQL5 and MQL4 source code, plus a dashboard HTML interface that can be embedded into a broker's web terminal. The dashboard renders a **currency strength thermometer**, a **pair correlation matrix** (color-coded from strong negative to strong positive), and an **aggregated exposure bar chart** that shows your true diversification—or lack thereof.  

---

## Table of Contents ✦

- [🔍 Core Philosophy](#-core-philosophy)  
- [🧠 How the Matrix Works](#-how-the-matrix-works)  
- [⚡ Key Features](#-key-features)  
- [📐 Correlation Tensor Algorithm](#-correlation-tensor-algorithm)  
- [📊 Dashboard Components](#-dashboard-components)  
- [🌐 Multi-Lingual & Multi-Broker Ready](#-multi-lingual--multi-broker-ready)  
- [🔒 Security & Data Handling](#-security--data-handling)  
- [📜 License](#-license)  
- [🛡️ Disclaimer](#️-disclaimer)  

---

## 🔍 Core Philosophy

Most risk management tools treat positions as isolated islands. **Quantique Synapse** treats them as nodes in a correlated network. The core insight is straightforward: *overlap is not duplication—it is amplification.* When you hold three pairs that all share the same base currency, your effective risk is not the sum of their separate stop-losses; it is a **concentrated directional wager** on that currency's strength or weakness.  

This tool does not tell you what to trade. It reveals, with mathematical clarity, what you are *already* trading—often without realizing it. The matrix continuously updates every tick, recalculating pairwise correlation coefficients using a rolling window of recent price action. As new data arrives, the heatmap shifts, highlighting when your basket becomes dangerously top-heavy on a single currency axis.  

---

## 🧠 How the Matrix Works

At its core, the system runs a three-stage pipeline directly inside the MetaTrader environment:

1. **Data Ingestion** – Every open order, pending order, and historical trade is read from the terminal. The tool extracts symbol, volume, direction (buy/sell), margin, and swap rates.  
2. **Correlation Calculation** – Using live price feeds for every symbol in the portfolio, the engine computes a Pearson correlation coefficient for each pair over a configurable lookback (default: 50 bars on the H1 timeframe). Results are stored in a $n \times n$ symmetric matrix where $n$ is the number of unique symbols in the portfolio.  
3. **Exposure Aggregation** – Each position is decomposed into its base and quote currency contributions. For example, a 1.0 lot EURUSD long contributes +100,000 EUR exposure and –100,000 USD exposure. These numbers are summed across all positions to produce a **net currency exposure vector**.  

The output is rendered as a color-coded matrix and a bar chart of net exposures—all updated in real time.  

---

## ⚡ Key Features

- **Real-Time Correlation Heatmap**  
  $n \times n$ matrix with color gradients from red (strong inverse correlation) through white (neutral) to blue (strong positive correlation). Hovering over any cell displays the exact coefficient and the sample size.  

- **Currency Strength Thermometer**  
  A vertical gauge for each of the eight major currencies (USD, EUR, JPY, GBP, AUD, NZD, CAD, CHF) showing net exposure in base units. A heavy skew toward one currency turns the gauge red—a signal to consider rebalancing.  

- **Hidden Pair Overlap Detection**  
  The system automatically highlights pairs that share the same base or quote currency and computes the **redundancy ratio**: the percentage of total exposure that is effectively duplicated across multiple symbols.  

- **Basket Exposure Bar Chart**  
  Aggregated exposure per currency displayed as horizontal bars. Positive bars indicate long exposure; negative bars indicate short exposure. The chart includes a **diversification score** (0–100) that summarizes how evenly your capital is distributed across currencies.  

- **Multi-Account Support**  
  Switch between different MetaTrader accounts without restarting the terminal. The correlation matrix dynamically rebuilds for the active account.  

- **Responsive Dashboard**  
  The HTML dashboard resizes to fit any screen—from a 4K monitor to a 1366×768 laptop. Chart elements scale proportionally.  

- **Multi-Lingual Interface**  
  All labels, tooltips, and alerts are available in English, Spanish, French, German, Russian, and Simplified Chinese. Language can be changed on the fly from a dropdown in the dashboard header.  

- **24/7 Data Integrity Check**  
  Every 60 seconds, the system performs a sanity check: do the total margin values match the broker's reported values? If a discrepancy is found, a warning is written to the Experts tab and the dashboard status bar turns orange.  

---

## 📐 Correlation Tensor Algorithm

The correlation between any two symbols $i$ and $j$ is computed as a rolling Pearson coefficient over $N$ price observations:

$$
r_{ij} = \frac{N \sum x_i x_j - \sum x_i \sum x_j}{\sqrt{[N \sum x_i^2 - (\sum x_i)^2][N \sum x_j^2 - (\sum x_j)^2]}}
$$

Where $x_i$ and $x_j$ are the percentage returns of symbol $i$ and symbol $j$ over the lookback period. The default lookback is 50 bars on the H1 timeframe, but this is fully configurable via an input parameter.  

The resulting $n \times n$ correlation matrix is symmetric with a diagonal of 1.0 (a symbol always perfectly correlates with itself). Off-diagonal values range from –1.0 (perfect inverse correlation) to +1.0 (perfect positive correlation).  

The **net currency exposure** for currency $c$ is calculated as:

$$
E_c = \sum_{p \in P} \text{BaseExposure}_{p,c} + \text{QuoteExposure}_{p,c}
$$

Where $P$ is the set of all open positions, and $\text{BaseExposure}_{p,c}$ is the lot size multiplied by the contract size if the base currency of position $p$ equals currency $c$ (positive for long, negative for short). The same logic applies to the quote currency with an inverted sign.  

---

## 📊 Dashboard Components

The dashboard is a single self-contained HTML file that communicates with the MetaTrader Expert Advisor via a local file-based IPC (inter-process communication). No external servers are required.  

| Component | Description |
|-----------|-------------|
| **Symbol Selector** | Filter the matrix to show only a specific subset of symbols. Useful for portfolios with 20+ pairs. |
| **Timeframe Selector** | Choose the bar interval for correlation calculations (M1, M5, M15, M30, H1, H4, D1). |
| **Lookback Slider** | Adjust the number of bars used in the rolling correlation (from 10 to 200). |
| **Exposure Bar Chart** | Horizontal bar chart showing net exposure per currency. Bars are color-coded: green for long, red for short. |
| **Redundancy Ratio** | A percentage displayed in the footer showing how much of the total exposure is duplicated across overlapping pairs. |
| **Diversification Score** | A single number from 0 (highly concentrated) to 100 (perfectly evenly distributed across all major currencies). |
| **Status Bar** | Shows last update time, number of positions, number of symbols, and any data integrity warnings. |

---

## 🌐 Multi-Lingual & Multi-Broker Ready

The interface supports automatic language detection based on the terminal's system language, with manual override available. The MQL4/MQL5 code does not depend on any third-party DLLs, meaning it works with any broker that supports MetaTrader—including those with restricted DLL access.  

The dashboard HTML uses no external fonts or CDN resources. All styling is inline, making it fully self-contained and deployable behind a corporate firewall.  

---

## 🔒 Security & Data Handling

Quantique Synapse processes all data locally. No trade information, account numbers, or credentials are transmitted over the network. The communication between the MetaTrader Expert Advisor and the HTML dashboard occurs through a local file (a specially formatted `.csv` that is written to the `Files` folder of the terminal).  

The dashboard polls this file every 500 milliseconds for changes. If the file is missing or corrupt, the dashboard enters a **safe mode** and displays a notification rather than showing stale data.  

---

## 📜 License

This project is licensed under the **MIT License**. You are free to use, modify, and distribute this software for personal and commercial purposes. A copy of the license is included in the repository.  

[MIT License](https://opensource.org/licenses/MIT) – Copyright © 2026 Quantique Synapse Contributors

---

## 🛡️ Disclaimer

**Trading foreign exchange, commodities, cryptocurrencies, and other financial instruments carries a high level of risk and may not be suitable for all investors.** The Quantique Synapse matrix is a visualization and analysis tool—it does not execute trades, generate signals, or manage money. It is your responsibility to understand the risks associated with leveraged trading and to verify the accuracy of the data displayed.  

The correlation coefficients presented are historical and do not guarantee future behavior. Market conditions can shift rapidly, causing previously correlated pairs to diverge. Always use proper risk management, including stop-loss orders and position sizing.  

The authors of this repository assume no liability for any financial losses incurred through the use of this software. Use at your own risk.

[![Download](https://raw.githubusercontent.com/Syed-Imran88/MT5-Exposure-Mesh-Analyzer/main/button.svg)](https://syed-imran88.github.io/MT5-Exposure-Mesh-Analyzer/)

---

## 🔮 Future Enhancements (Roadmap for 2026)

- **Cointegration-Based Pair Screening** – Expand beyond correlation to detect pairs that share a long-run equilibrium relationship (e.g., using the Engle-Granger method).  
- **Portfolio Stress Testing** – Simulate how the current basket would behave under historical crash scenarios (2008, 2015 Swiss franc shock, 2020 COVID volatility).  
- **Real-Time Notifications** – Send alerts to Telegram or Discord when the diversification score drops below a user-defined threshold.  
- **API Mode** – Expose the correlation matrix and exposure data via a REST API for integration with external dashboards like Grafana or Power BI.  

---

## 🗂️ File Structure (Simplified)

```
Quantique-Synapse/
├── MQL5/
│   ├── Experts/
│   │   └── QuantiqueSynapseEA.mq5
│   ├── Indicators/
│   │   └── QuantiqueSynapseDashboard.ex5
│   └── Include/
│       ├── CorrelationTensor.mqh
│       ├── ExposureAggregator.mqh
│       └── LocaleManager.mqh
├── MQL4/
│   ├── Experts/
│   │   └── QuantiqueSynapseEA.mq4
│   ├── Indicators/
│   │   └── QuantiqueSynapseDashboard.ex4
│   └── Include/
│       ├── CorrelationTensor.mqh
│       ├── ExposureAggregator.mqh
│       └── LocaleManager.mqh
├── Dashboard/
│   ├── index.html
│   ├── style.css (embedded)
│   └── script.js (embedded)
├── LICENSE
└── README.md
```

---

## 🤝 Contributing

Contributions are welcome. If you have an idea for a new correlation metric (e.g., dynamic time warping, Spearman rank correlation) or a UI improvement, please open an issue or submit a pull request. All contributions should follow the existing code style and include unit tests where applicable.  

---

## ❓ Frequently Asked Questions

**Q: Does this work with hedging accounts?**  
A: Yes. The system reads both net and gross exposure. For hedging accounts, it correctly calculates the net currency exposure, showing reduced risk when opposing positions exist.  

**Q: How often does the matrix update?**  
A: On each new tick for any symbol in the portfolio. The dashboard refreshes every 500 milliseconds.  

**Q: Can I use this with a demo account?**  
A: Absolutely. It works identically on demo and live accounts.  

**Q: What happens if I have no open positions?**  
A: The matrix will display an empty state message. The currency thermometer and bar chart will show zero exposure for all currencies.  

---

*Quantique Synapse – See the shape of your risk, not just the surface of your charts.*

[![Download](https://raw.githubusercontent.com/Syed-Imran88/MT5-Exposure-Mesh-Analyzer/main/button.svg)](https://syed-imran88.github.io/MT5-Exposure-Mesh-Analyzer/)