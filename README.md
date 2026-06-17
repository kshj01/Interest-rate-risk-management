**Interest rate risk management using short rate models.**

## Overview
This project implements an Interest rate risk management framework using two stochastic models - Vasicek and Cox-Ingersoll-Ross model 
calibrated to real US Treasury Bill Data. The framwork cover the full pipeline from parameter estimation to risk metric computation.

## Data
- **Source** - 3-Month Treasury Bill Secondary Market Rate (TB3MS)
- **Provider** - Federal Reserve Economic Data (FRED), St Louis Fed
- **Sample** - January 2000 - April 2026 (Montly, 317 observations)

## Methodolgy

**Phase 1 - Data and Parameter Estimation** - 
Historical monthly 3-Month US Treasury Bill rates (TB3MS) from FRED were used as a proxy for the theoretical short rate r(t), 
covering January 2000 to April 2026 (317 observations). 
Why : The 3-Month US Treasury Bill is a standard academic proxy for the short rate - risk free, short term and market-determined. The sample period was restricted to 2000 onwards to avoid the structurally different monetary regime of the 1970s-80s (extreme inflation) which would distort long run parameter estimates.

VASICEK PARAMETERS - OLS:
The Euler-Maruyama discretisation of the Vasicek SDE yields a linear regression of form, Δrt​=a+b⋅rt​+ϵt​. OLS was applied using statsmodels

CIR PARAMETERS - Method of Moments:
CIR's non-constant variance violates OLS regression. Method of Moments was used instead - matching theoretical stationary mean, variance and autocorrelation.

Limitations : The two models use different estimation methods (OLS vs MoM), so parameter differences partly reflect estimation method rather than purely model
structure.

**Phase 2 - Monte Carlo Simulation** - 
10000 short rate paths were simulated over a 1-year horizon (12 monthly steps, dt = 1/12) using Euler-Maruyama discretisation for both models.
Why 10000 paths : Sufficient for stable VaR estimates at the 99th percentile (requiring atleast 1000 observations in the tail).
Why Euler-Maruyama : Simple and sufficiently accurate for small dt = 1/12
Limitations : Euler-Maruyama introduces discretisation error. CIR rates were floored to zero via np.maximum(r,0) within the diffusion term to prevent numerical
instability in edge cases near the Feller boundary.

**Phase 3 - Zero Coupon Bond Pricing** - 
A 1-year ZCB (100 face value) was priced using the closed-form affine formula: P(t,T) = 100. exp( a(t,T) - b(t,T)*rt ) where a(t,T) and b(t,T) are model specific 
functions of the paramters.
Why closed form : Both Vasicek and CIR admit exact analytical bond pricing formulas avoiding the need for nested Monte Carlo Simulation. Prices were computed across all 13 time points (0,1,2,...,12) with (T-t) shirnking from 1 year to 0 as t advances.

**Phase 4 - Duration and Convexity** - 
Modified Duration and Convexity were computed numerically using the central difference (bump) method with dr = 0.0001 (1 basis point)
Why numerical bumping : Works for any pricing formula without requiring analytical differentiation. Consistent across both models - any differnce in Duration or 
Convexity is attributable to model structure and parameters, not the method.

**Phase 5 - Value at Risk** - 
1-month VaR was computed from the Monte Carlo PnL distributon, d(Pi) = Pi(t=1) - P0(t=0) for i=1

Why 1-month horizon : The standard regulatory and risk management horizon from market VaR.
Limitation : This is a parametric simulation VaR, not historical VaR - results are model dependent. The 1-month PnL includes both rate risk and time decay.




## Results
Parameters -

Vasicek (OLS) : Kappa (mean reversion) = 0.1023, mu (long run mean) = 1.27%, sigma (volatility) = 0.65%


CIR (MoM) : Kappa (mean reversion) = 0.0577, mu (long run mean) = 1.92%, sigma(volatility) = 4.90%


Risk Metric - 

Vasicek : Duration (years) = 0.9506, Convexity  = 0.9036, VaR 95% (1-month) = -0.024267, VaR 99% (1-month) = +0.071920

CIR : Duration (years) = 0.9713, Convexity = 0.9435, VaR 95% (1-month) = +0.076355, VaR 99% (1-month) = +0.234643

VaR estimates may vary everytime when code is run due to use of numpy.random

## Key Findings 
The large difference in 1-month VaR between models is primarily driven by the differnece in sigma estimates (0.65% vs 4.73%) arising from different estimation methods. This highlights that **model choice and estimation method jointly determine perceived risk**.


# Requirements
numpy, pandas, matplotlib, statsmodel

# Usage
Download "TB3MS.csv" 

Place in the same directory as the notebok

Run "Interest_Rate_Risk_Management.ipynb" end to end.








