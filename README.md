<div style="overflow-x: auto; white-space: nowrap;">
  <img src="images/workflow.png" width="260" />
  <img src="images/iv_surfaces.png" width="260" />
  <img src="images/oosc.png" width="260" />
  <img src="images/param_landscape.png" width="260" />
  <img src="images/Theory.png" width="260" />
  <img src="images/dynamic_thresholds.png" width="260" />
</div>

<p align="center">
  <img src="images/workflow.png" width="22%" />
  <img src="images/iv_surfaces.png" width="22%" />
  <img src="images/param_landscape.png" width="22%" />
  <img src="images/oosc.png" width="22%" />
</p>


# Modeling Market Geometry  
### An Engineering Approach to Implied Volatility in Options

This repository contains the code accompanying the research project *Modeling Market Geometry: An Engineering Approach to Implied Volatility in Options*.  
The purpose of this README is to give a concise overview of the project, its motivation, and how the methodology is organized. It is not a substitute for the full paper; instead, it provides enough structure for someone who will not read the paper to understand what the project is doing and why.

---

## 1. Project Context and Motivation

The work began as an engineering optimization project originally developed and tuned for a research module I conducted in CEE 251L under the academic supervision of Prof. Henri P. Gavin at Duke University. In that early stage, the goal was to design a price-based decision rule built around a weighted signal (the "quality equation") and to optimize its parameters using stochastic search and penalty-based objective functions.

As the project evolved, the focus shifted from stock-level price dynamics to the geometry of the entire implied volatility (IV) surface. Options markets provide a richer, higher-dimensional structure: volatility is not a scalar, but a surface $IV(m,\tau)$ depending on moneyness $m$ and days-to-expiration $\tau$. This perspective allowed the project to merge ideas from engineering optimization, applied mathematics, and quantitative finance into a single framework.

The central premise is that the IV surface contains structured information, and that the way this surface deforms over time can be used as a predictive signal for short-term volatility movements.

---

## 2. High-Level Methodology

The pipeline implemented in this repository follows four main steps. The technical details, proofs, and full rationale are left for the paper; here only the essential components are summarized.

### 2.1 Surface Construction

For each trading day, raw option-chain data is cleaned, filtered for liquidity, and projected onto a smooth, continuous implied-volatility surface using a penalized spline fit.  
The surface is modeled as

$$
IV_t(m,\tau) \approx f(m,\tau),
$$

where smoothing penalties suppress irregularities arising from bid/ask noise or sparse strikes.

### 2.2 Dimensionality Reduction via PCA

To study how the IV surface evolves, daily slices at fixed maturities are differenced and stacked. Principal Component Analysis (PCA) is then applied to these deformations.  
The first three components consistently correspond to:

- Level shifts  
- Skew (tilt)  
- Smile or curvature changes  

If $S_t(m)$ is the cross-section, the PCA scores are

$$
PC_i^t = e_i^\top (S_t - S_{t-1}).
$$

Their time differences, $\Delta PC_i^t$, serve as the core volatility-momentum signals.

### 2.3 Feature Engineering

In addition to PCA deltas, the model incorporates:

- Local curvature changes near at-the-money  
- The implied–realized volatility spread  
- Short-horizon rolling averages of $\Delta PC_1^t$  

These features capture both instantaneous and smoothed information about surface geometry.

### 2.4 Volatility Positioning

The strategy trades **delta-neutral option structures**:

- **Long volatility** when signals indicate upward volatility pressure (ATM straddle)  
- **Short volatility** when signals indicate compression (slightly OTM strangle)  

Trade exits follow strict rules: profit targets, stop-losses, maximum duration, and delta re-hedging.  
The aim is not to forecast direction, but to express a clean view on volatility itself.

---

## 3. Summary of Results

The strategy was tested across multiple regimes:

- A multi-year in-sample period  
- Several out-of-sample windows  
- Stress periods resembling COVID-19 and 2008  
- One week of live trading  

Performance metrics include annualized return, Sharpe ratio, drawdown, and PSR (Probabilistic Sharpe Ratio).  
These results show that the model tends to perform well during sustained volatility regimes, while single-day shock events remain challenging.

A full breakdown, including tables and backtest plots, is presented in the paper.

---


---

## 4. Origins: The Quality Equation

The initial stage of this research involved a price-based signal

$$
Q = q_1 \dot{\bar{p}} + q_2 \ddot{\bar{p}} + q_3 v,
$$

where $\dot{\bar{p}}$ is smoothed momentum, $\ddot{\bar{p}}$ is acceleration, and $v$ is local volatility.  
A stochastic optimizer was used to select the weights $(q_1,q_2,q_3)$ under penalty terms discouraging fragile or overfit parameter combinations.

Although the volatility-surface model ultimately replaced this framework, many of its lessons—particularly smoothing rugged landscapes and emphasizing robustness—directly influenced the final design.

---

## 4. Reproducing the Pipeline

1. Load daily option chains for the chosen underlying.  
2. Fit the IV surface using the spline module.  
3. Extract fixed-maturity slices and compute PCA.  
4. Generate feature vectors and apply the decision rule.  
5. Run a backtest or deploy inside QuantConnect.

The implementation assumes access to option-level historical data.

---

## Closing Remarks

This repository represents an attempt to view volatility not simply as an input to option pricing, but as a geometric object evolving on a manifold. By blending engineering optimization techniques with volatility-surface analysis, the project aims to provide an interpretable and data-driven approach to volatility trading.  
**Readers interested in the mathematical development and empirical evaluation should refer to the full paper, which is included in the repository.**




