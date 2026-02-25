---
layout: post
title: "Minimum Detectable Effect in Time Series Data"
subtitle: "Explain in code and formula"
date: 2026-02-19 23:04:34
header-style: text
catalog: true
author: "Yuan"
tags: [MDE, TBR, Google, Regression, Time Series, F-test, CUPED,Variance Reduction]
---
{% include linksref.html %}

When designing a Time-Based Regression (TBR) in a geo-experiment, the most critical question is: "How much of a change do I need to see to be sure it wasn't just noise?" This is the Minimum Detectable Effect (MDE). It is the smallest true causal impact that an experiment has a high probability (power) of detecting at a specific significance level.

This framework of TBR fits within a simple linear regression model:
$$y = a + bx + \epsilon$$
In this context, $y$ represents the treatment series, and $x$ represents the control series. The Minimum Detectable Effect (MDE) is the smallest true causal impact that an experiment has a high probability (power) of detecting at a specific significance level.


# 1. The Statistical Foundation: Prediction of Mean
The statistical logic for MDE in time series often draws from the prediction of the mean of $m$ new observations, as seen in classical regression theory.As defined in standard statistical texts, the $1 - \alpha$ prediction limits for a new mean $\bar{Y}_{h(new)}$ are:

$$\hat{Y}_h \pm t(1 - \alpha/2; n - 2)s\{\text{predmean}\}$$

The variance of this prediction, $s^2\{\text{predmean}\}$, is the core of the MDE calculation:

$$s^2\{\text{predmean}\} = MSE \left[ \frac{1}{m} + \frac{1}{n} + \frac{(X_h - \bar{X})^2}{\sum (X_i - \bar{X})^2} \right]$$

In your experiment, $m$ represents the number of test periods ($n_{test}$), and $n$ represents the number of historical pre-test periods.


# 2. Part 2: MSE and Variance Reduction (The CUPED Approach)
In the provided code, the "Residual Noise" is calculated by adjusting the raw volatility of the treatment series by its correlation with the control. This is the MSE of the regression model and serves the same purpose as CUPED (Controlled-experiment Using Pre-Experiment Data).

**Why this matters**: Traditional experiments often suffer from "noise" inherent in the metric itself. CUPED reduces this noise by using a covariate (in our case, the control time series $X$) to explain away pre-existing variance.

```python
# Calculate Residual Standard Deviation (root of MSE)
# sigma = raw_std * sqrt(1 - rho^2)
sigma = np.std(self.y, ddof=2) * np.sqrt(1 - corr ** 2)
```

# 3. Part 3: Deriving the $SQ$ Term as an F-Statistic
## The Geometry of the "Leverage" Term

The term $\frac{(X_h - \bar{X})^2}{\sum (X_i - \bar{X})^2}$ measures "leverage"—how far the control values during the test period ($X_h$) deviate from the pre-test average ($\bar{X}$). Since we don't know the future values of $X_h$ exactly when planning, we treat this ratio as a random variable to find a conservative bound.

### 3.1. The Distribution of the Numerator
To find the distribution of the numerator, we first look at the difference between the future mean and the historical mean:
- Let $\bar{X}_h$ be the mean of $m$ new observations and $\bar{X}$ be the mean of $n$ historical observations.
- The difference $(\bar{X}_h - \bar{X})$ is normally distributed with mean 0 and variance $\sigma_x^2 (\frac{1}{m} + \frac{1}{n})$.
- Therefore, the standardized variable $Z = \frac{\bar{X}_h - \bar{X}}{\sqrt{\sigma_x^2 (\frac{1}{m} + \frac{1}{n})}}$ follows a standard normal distribution $N(0, 1)$.
- The square of a standard normal variable follows a Chi-square distribution with 1 degree of freedom:$$\frac{(\bar{X}_h - \bar{X})^2}{\sigma_x^2 (\frac{1}{m} + \frac{1}{n})} \sim \chi^2(1)$$
- This gives us the distribution of the numerator: $$(\bar{X}_h - \bar{X})^2 \sim \sigma_x^2 (\frac{n+m}{nm}) \chi^2(1)$$

### 3.2. The Distribution of the Denominator
The denominator is the sum of squared deviations from the pre-test period:
- Based on the property of sample variance, $\frac{\sum (X_i - \bar{X})^2}{\sigma_x^2} \sim \chi^2(n-1)$.
- Thus, the denominator follows: $$\sum (X_i - \bar{X})^2 \sim \sigma_x^2 \chi^2(n-1)$$

### 3.3 Creating the F-Statistic
An F-statistic is the ratio of two independent $\chi^2$ variables, each divided by their respective degrees of freedom:$$F(1, n-1) = \frac{\chi^2(1) / 1}{\chi^2(n-1) / (n-1)}$$

By substituting our distributions into the leverage ratio:
$$\frac{(\bar{X}_h - \bar{X})^2}{\sum (X_i - \bar{X})^2} = \frac{\sigma_x^2 (\frac{n+m}{nm}) \chi^2(1)}{\sigma_x^2 \chi^2(n-1)} = \frac{n+m}{nm} \cdot \frac{\chi^2(1)}{\chi^2(n-1)}$$

To transform this into the F-distribution, we multiply and divide by the degrees of freedom $(n-1)$:
$$\frac{(\bar{X}_h - \bar{X})^2}{\sum (X_i - \bar{X})^2} = \frac{n+m}{nm(n-1)} \cdot \frac{\chi^2(1)/1}{\chi^2(n-1)/(n-1)} = F(1, n-1) \cdot \frac{n+m}{nm(n-1)}$$

## Python code:
```python
# F-distribution quantile for model variance
phi = stats.f(dfn=1, dfd=n - 1).ppf(flevel)

# Orignal Code in https://github.com/google/matched_markets/blob/master/matched_markets/methodology/tbrmmdiagnostics.py
sq = np.sqrt(phi * (n + 1) / (n * n_test * (n - 1)) + 1 / n + 1 / n_test)

# Corrected one,  https://github.com/google/matched_markets/issues/16#issue-3966610639 raised by me
sq = np.sqrt(phi * (n + n_test) / (n * n_test * (n - 1)) + 1 / n + 1 / n_test)

```

{{important}}
In a typical setting, n=90, n_test = 56, flevel = 0.9
</br>
Then, we have: $\phi = stats.f(dfn=1, dfd=89).ppf(0.9) \approx 2.7628$

</br>
the orignal one will give sq = $sq1 \approx 0.17184$
</br>
The new one will give sq = $sq1 \approx 0.17282$
</br>
The difference is minor, only 0.6%. The original one underestimate MDE by 0.6%.

{{end}}

# 4. Part 4: The Statistical Multiplier (Thresholding)
Once we have calculated the standard error of the prediction (sq and sigma), we must determine how many multiples of that error the observed signal must reach to be considered significant.

## The T-Distribution Quantiles
We use the t-distribution because we are working with estimated variances from a finite sample size $n$. 
The degrees of freedom are $n-2$ because the linear regression model $y = a + bx$ estimates two parameters.

### 4.1 tq_sig (The Significance Hurdle):
- This is the critical value for your significance level (e.g., $\alpha = 0.05$).
- It defines the boundary of the "Null Hypothesis" region. If your observed effect is smaller than this, you cannot distinguish it from random noise.
- Formula: $t_{1-\alpha, n-2}$.
### 4.2 tq_pow (The Power Hurdle):
- This represents the desired statistical power (e.g., $1-\beta = 0.80$).
- While tq_sig protects you from "False Positives," tq_pow protects you from "False Negatives". It ensures that if a real effect exists, the experiment is sensitive enough to detect it.
- Formula: $t_{1-\beta, n-2}$.

## Scaling to the Test Period: n_test * sq
The MDE represents the total cumulative impact required over the entire test duration.
- sq: As derived in Part 3, this is the normalized standard error per time point.
- n_test: Multiplying by the number of test points scales the per-period detection threshold up to the total volume of the experiment window.

## The Combined "Design Threshold"
When you add the two t-quantiles together, you are essentially calculating the distance between the center of the Null Hypothesis distribution and the center of the Alternative Hypothesis distribution required to avoid overlapping beyond your error tolerances.
$$\text{Total Multiplier} = (t_{\alpha} + t_{\beta}) \cdot n_{test} \cdot SQ$$


## Python code:
```python
tq_sig = stats.t.ppf(sig_level, df=n - 2)
tq_pow = stats.t.ppf(power_level, df=n - 2)
term = (tq_sig + tq_pow) * n_test * sq
```

# 5. Putting it All Together

The final result of your code is the product of this Design Threshold and the Residual Noise (derived via the CUPED/MSE approach in Part 2):
$$MDE = \underbrace{(t_{sig} + t_{pow}) \cdot n_{test} \cdot SQ}_{\text{Part 4: Statistical Hurdle}} \cdot \underbrace{\sigma_y \sqrt{1 - \rho^2}}_{\text{Part 2: Residual Noise}}$$

This ensures that your MDE accounts for the probability of error, the duration of the test, the precision of the regression, and the strength of your control group.

```python
impact = term * sigma
mde = (term * sigma / y[-n_test:].sum())*100
```

# Reference
Deng, A., Xu, Y., Kohavi, R., & Walker, T. (2013). "Improving the Sensitivity of Online Controlled Experiments by Utilizing Pre-Experiment Data." Proceedings of the fifth ACM international conference on Web search and data mining (WSDM).

https://github.com/google/matched_markets/blob/master/matched_markets/methodology/tbrmmdiagnostics.py


---
