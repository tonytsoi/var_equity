# Basic Estimation Methods of Value at Risk (VaR) and Expected Shortfall for Equity Portfolios
Value at Risk (“VaR”) and Expected Shortfall are measures broadly adopted across the financial industry to assess the level of risks in assets and portfolios.

In <a href="https://medium.com/@tsoiyingkit/basic-estimation-methods-of-value-at-risk-var-and-expected-shortfall-for-equity-portfolios-2646b00ab338">this article</a>, I introduce the concepts of VaR and Expected Shortfall and illustrate the three basic estimation methods, i.e. the Historical Simulation, the Parametric Method and the Monte Carlo Simulation for equity portfolios.

For the Monte Carlo Simulation, I discuss how copula and matrix decomposition are used in the simulation process to incorporate correlation between the individual positions in the portfolio. In particular, the Gaussian Copula and t-Copula, together with the Cholesky Decomposition method are introduced.

I also discuss how the Maximum Likelihood Estimation (“MLE”) method is used to parameterise the assumed distribution and copula given the historical returns data.

![var_example](https://github.com/tonytsoi/var_equity/blob/main/var_example.png?raw=true)
