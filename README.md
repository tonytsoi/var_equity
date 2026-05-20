# Basic Estimation Methods of Value at Risk (VaR) and Expected Shortfall for Equity Portfolios

Value at Risk (VaR) and Expected Shortfall are measures broadly adopted across the financial industry to assess the level of risk in assets and portfolios. This project illustrates three basic estimation methods — Historical Simulation, Parametric Method, and Monte Carlo Simulation — applied to an example equity portfolio of MSFT, AAPL, and GOOG.

For a detailed walkthrough, read the full article on [Medium](https://medium.com/@tsoiyingkit/basic-estimation-methods-of-value-at-risk-var-and-expected-shortfall-for-equity-portfolios-2646b00ab338).

---

## Value at Risk (VaR) and Expected Shortfall

**Value at Risk (VaR)** estimates the potential loss of an asset or portfolio over a defined period at a given confidence level. For example, a 1-day VaR of $10m at 99% confidence means there is a 1% chance the portfolio loses more than $10m in one day.

```text
Pr(x ≤ VaR_c(X)) = 1 - c
```

**Expected Shortfall (ES)** quantifies the expected loss given that the loss exceeds the VaR threshold — it captures the severity of losses in the tail of the distribution.

```text
ES_c(X) = E[X | X ≤ VaR_c(X)]
```

ES is a **coherent** risk measure satisfying the subadditivity axiom (`R(X+Y) ≤ R(X) + R(Y)`), whereas VaR is not. This makes ES a more consistent measure of combined portfolio risk, particularly under market stress when positions are positively correlated.

---

## Portfolio Setup

The example portfolio consists of three stocks with the following weights:

| Stock | Weight |
| --- | --- |
| Microsoft (MSFT) | 20% |
| Apple (AAPL) | 20% |
| Alphabet (GOOG) | 60% |

Two years of daily historical price data are downloaded from Yahoo Finance using `yfinance`. Daily portfolio returns are calculated as the weighted sum of individual stock returns.

---

## Method 1: Historical Simulation

Historical Simulation is a non-parametric approach that uses the empirical distribution of historical returns directly, without assuming any particular distribution.

- **VaR** is the return at the chosen percentile of the ranked historical return distribution (e.g. the 5th percentile for 95% confidence).
- **ES** is the average of all returns that fall below the VaR threshold.

No distributional assumptions are required, making this method straightforward and intuitive.

![hist_var](https://github.com/tonytsoi/var_equity/blob/main/hist_var.png?raw=true)

---

## Method 2: Parametric (Variance-Covariance) Method

The Parametric Method assumes that portfolio returns are normally distributed and uses the variance-covariance matrix of individual stock returns to derive closed-form formulas for VaR and ES.

The **variance-covariance matrix** Σ is constructed from historical returns, and the **portfolio variance** and **mean return** are computed as:

```text
σ_p² = w Σ wᵀ
μ_p  = w μ
```

where `w` is the vector of portfolio weights and `μ` is the vector of individual mean returns.

Under the normal distribution assumption, VaR and ES are:

```text
VaR_c = μ_p + Φ⁻¹(c) · σ_p
ES_c  = μ_p + [φ(Φ⁻¹(c)) / (1 - c)] · σ_p
```

where `φ` is the standard normal PDF and `Φ⁻¹` is the inverse normal CDF.

---

## Method 3: Monte Carlo Simulation

Monte Carlo Simulation generates a large number of random samples of potential future returns based on the statistical characteristics of the portfolio. VaR and ES are then calculated from the simulated return distribution. This approach is more flexible than the historical or parametric methods, as various distributional assumptions can be adopted.

For a portfolio with multiple assets, a **copula** is used to model the joint distribution of returns, and the **Cholesky Decomposition** is used to incorporate the correlation structure.

### Cholesky Decomposition

Cholesky Decomposition factorises the covariance matrix into lower and upper triangular matrices:

```text
Σ = L Lᵀ
```

This decomposition is used to generate correlated random variables from independent ones. For example, with two assets:

```text
x₁ = σ₁ z₁
x₂ = ρ₁₂ σ₂ z₁ + σ₂ √(1 - ρ₁₂²) z₂
```

where `z₁` and `z₂` are independent standard normal variables. The covariance matrix of any portfolio is guaranteed to be positive semi-definite, so Cholesky Decomposition always applies.

### Copula

A copula is a joint distribution function that links the marginal distributions of individual assets into a multivariate distribution, regardless of the form of each marginal distribution. This allows each stock's return to follow a different distribution while still modelling the dependence between positions.

### Gaussian Copula (Normal Marginals)

The Gaussian Copula uses a multivariate normal distribution to model the dependence between assets:

```text
C(u₁, u₂, ..., uₙ) = Φₙ(Φ⁻¹(u₁), Φ⁻¹(u₂), ..., Φ⁻¹(uₙ); Σ)
```

Independent normal random variables are generated and then combined using the Cholesky matrix `L` to produce correlated returns. This is the simplest Monte Carlo approach but does not capture fat tails in the return distribution.

### Gaussian Copula with Student's t Marginals

Empirical evidence shows that equity returns exhibit **fat tails** — extreme returns occur more frequently than the normal distribution would predict. The Student's t distribution, with its additional degrees-of-freedom parameter `ν`, provides a better fit: lower degrees of freedom correspond to heavier tails.

**Maximum Likelihood Estimation (MLE)** is used to fit the degrees of freedom to the historical returns of each stock. For the example portfolio:

| Stock | Degrees of Freedom (ν) |
| --- | --- |
| AAPL | ≈ 4.94 |
| GOOG | ≈ 4.07 |
| MSFT | ≈ 6.54 |

Correlated normal variables are converted to Student's t variables via the probability integral transform. The resulting VaR and ES estimates are more extreme than those from the Gaussian-only approach.

### t-Copula (Tail Dependence)

During extreme market stress, correlations between positions tend to increase — a phenomenon known as **tail dependence**. The Gaussian Copula does not account for this. The t-Copula addresses tail dependence by using a multivariate Student's t distribution at the portfolio level:

```text
C(u₁, u₂, ..., uₙ) = Tᵥ(tᵥ⁻¹(u₁), tᵥ⁻¹(u₂), ..., tᵥ⁻¹(uₙ); Σ)
```

The degrees of freedom `ν` are estimated from portfolio-level historical returns using MLE. The t-Copula produces the most conservative VaR and ES estimates as it captures both the fat tails of individual return distributions and the tail dependence between positions.

---

## Comparison of Monte Carlo Results

| Method | VaR (95%) | ES (95%) |
| --- | --- | --- |
| Gaussian Copula | -1.94% | -2.46% |
| Gaussian Copula + Student's t marginals | -2.04% | -2.49% |
| t-Copula | -2.26% | -2.67% |

The t-Copula produces the largest loss estimates because it accounts for both fat-tailed marginal distributions and increased tail dependence under stress scenarios.

---

## Visualisation

![var_example](https://github.com/tonytsoi/var_equity/blob/main/var_example.png?raw=true)

---

## Requirements

- `yfinance`
- `numpy`
- `pandas`
- `scipy`
- `matplotlib`
- `seaborn`
