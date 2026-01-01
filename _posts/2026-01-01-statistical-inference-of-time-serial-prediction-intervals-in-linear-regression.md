---
layout: post
title: "Statistical Inference of Time Serial Prediction Intervals in Linear Regression"
subtitle: "Understanding the cihw calculation in Time Based Regression"
date: 2026-01-01 10:10:26
header-style: text
catalog: true
author: "Yuan"
tags: [OLS, Regression, TBR, Time Based Regression, Interval, Time Series]
---
{% include linksref.html %}
This document breaks down the statistical logic behind the confidence interval half-width (`cihw`) formula used in TBR analysis. It explains how the code translates standard OLS prediction variance into a practical business metric.

## 1. The Core Formula
The code calculates the **prediction interval** for the **Total Cumulative Impact**.

The mathematical representation of the code logic is:
$$CIHW_{total} = t_{crit} \times n_{test} \times \sigma \times \sqrt{\underbrace{\frac{1}{n} + \frac{dv}{n}}_{\text{Model Uncertainty}} + \underbrace{\frac{1}{n_{test}}}_{\text{Test Period Noise}}}$$

---

## 2. Component Breakdown

### A. The Variance of the Prediction Error
The goal is to quantify the uncertainty of the "Lift" (Estimate). The Estimate is the difference between what we observed ($\bar{y}_{test}$) and what the model predicted would happen ($\hat{y}_{test}$).

$$Var(\text{Error}) = Var(\hat{y}_{test}) + Var(\bar{y}_{test})$$

*(Note: We assume the test period noise is independent of the training period noise, so covariance is 0)*

### B. Model Uncertainty (The Regression Line)
The first source of error is that our regression line itself is just an estimate. In OLS theory, the variance of a predicted mean at a new point $x_{test}$ is:

$$Var(\hat{y}) = \sigma^2 \left( \frac{1}{n} + \frac{(x_{test} - \bar{x}_{train})^2}{\sum (x_i - \bar{x})^2} \right)$$

**How the code implements this:**
* **Intercept Term:** The $\frac{1}{n}$ term represents the uncertainty of the baseline intercept.
* **Leverage Term (`dv`):** The code calculates `dv` to represent the "distance" of the test period data from the training data mean.
    * Code: `dv = dx**2 / np.var(x, ddof=0)`
    * Math: $\frac{dv}{n} = \frac{(x_{test} - \bar{x}_{train})^2}{\sum (x_i - \bar{x})^2}$

When these are combined, `(1 + dv) / n` in the code perfectly matches the standard OLS prediction variance formula.

### C. Test Period Noise (Sampling Error)
The second source of error is the natural volatility of the test period itself. Even if we had a "perfect" model, the actual data during the experiment would still bounce around.

Since we are looking at the average of `n_test` days, the variance is:
$$Var(\bar{y}_{test}) = \frac{\sigma^2}{n_{test}}$$

**How the code implements this:**
* This matches the `1 / n_test` term inside the square root.

---

## 3. From "Average" to "Total" Impact
The statistical formulas above give us the variance for the **Average Daily Impact**. However, business stakeholders want to know the **Total Impact** over the entire campaign.

* **Average SE:** $SE_{avg} = \sigma \sqrt{\frac{1+dv}{n} + \frac{1}{n_{test}}}$
* **Scaling:** To get the total, we multiply the average standard error by the duration of the test (`n_test`).

**Code Implementation:**
```python
scale = n_test * sigma * np.sqrt(...)
```

## 4. Final Confidence Interval Calculation
Finally, to get the Half-Width (the "$\pm$" range), we multiply the Standard Error by the critical t-statistic.
$$CIHW = t_{crit} \times SE_{total}$$

Code Implementation:
```python
cihw = tq_sig * scale
```
`tq_sig`: The critical t-value (likely for 95% confidence) with $n-2$ degrees of freedom.

## Summary Table: Code to Statistics Mapping

| Code Variable | Statistical Concept | Intuition |
| :--- | :--- | :--- |
| **`sigma`** | $\sigma$ (Residual Std Dev) | The inherent "noisiness" of the market history. |
| **`1/n`** | $\frac{1}{n}$ | Uncertainty in estimating the baseline (Intercept). |
| **`dv/n`** | Leverage: $\frac{(x-\bar{x})^2}{S_{xx}}$ | Uncertainty from the slope. If the test period conditions (`xt`) are very different from history (`xn`), this error grows large. |
| **`1/n_test`** | $\frac{1}{m}$ | Uncertainty in the test data itself (Sampling noise). |
| **`n_test`** | Multiplier | Converts "Daily Average" uncertainty to "Total Campaign" uncertainty. |

## Ref:
1. https://github.com/google/matched_markets/blob/master/matched_markets/methodology/tbrmmdiagnostics.py
2. 'Applied Linear Statistical Models', 5th Edition, Michael H. Kutner,Christopher J. Nachtsheim,John Neter,William Li

---
