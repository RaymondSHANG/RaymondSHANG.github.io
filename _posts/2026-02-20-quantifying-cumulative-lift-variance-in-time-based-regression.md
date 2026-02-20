---
layout: post
title: "Quantifying Cumulative Lift Variance in Time Based Regression"
subtitle: "variance decomposition"
date: 2026-02-20 00:51:13
header-style: text
catalog: true
author: "Yuan"
tags: [TBR, Lift, Post Experiment, Cumulative Effect, Google]
---
{% include linksref.html %}

# Problem overview
When analyzing a time series experiment (like a geo-experiment), the core logic relies on using historical data to predict what would have happened without the intervention.

The process looks like this:
1. The Pre-Period Model: Before the experiment starts, you estimate a standard frequentist regression model using historical data:$$y = a + bx + \epsilon$$(Where $y$ is the treatment metric, $x$ is the control metric, and $\epsilon$ is the residual noise).
2. The Experiment Lift: During the experiment window, you compute the daily incremental lift by subtracting your model's prediction ($\hat{y}(t)$) from the actual observed value ($y(t)$):$$lift(t) = y(t) - \hat{y}(t)$$
3. The Hypothesis Test: Your stakeholders don't just care about daily fluctuations; they want to know if the total campaign was effective. Therefore, you are testing the Null Hypothesis for the cumulative sum:$$H_0: \sum_{t=1}^{T} lift(t) = 0$$

## The Core Challenge
To calculate a p-value or confidence interval for $H_0$, you need the variance of that cumulative sum. **The challenge is that the cumulative effect variance must account for both parameter uncertainty and residual noise**.

If you simply sum up the daily residual variances, your confidence intervals will be dangerously narrow. As outlined in the paper "Estimating Ad Effectiveness using Geo Experiments in a Time-Based Regression Framework" (Kerman et al., 2017), the variance has two distinct sources that behave very differently over time:
- Parameter Uncertainty (The Model's Error): The coefficients ($\hat{a}, \hat{b}$) are estimates derived from a sample. Because you use the exact same coefficients to predict every single day of the test period, any error in those estimates is systematic. It compounds heavily (quadratically) as the test goes on.
- Residual Noise (The Data's Error): Even a perfect model has daily fluctuations ($\epsilon$). Assuming these are independent day-to-day, this noise simply adds up linearly.

# Step-by-Step Code Walkthrough
Let's trace how the code implements this logic using a simplified 3-day test example.

The Inputs
- lift: The vector of daily differences $[y_1-\hat{y}_1, y_2-\hat{y}_2, y_3-\hat{y}_3]$.
- cntrl_mat ($X$): The design matrix for the test period.
- vsigma ($\Sigma$): The covariance matrix of the coefficients $(X_{pre}^T X_{pre})^{-1} \sigma^2$.
- sigmasq ($\sigma^2$): The Mean Squared Error (MSE) from the pre-test model.
- df_resid: Degrees of freedom from the pre-test model.
Assuming we have 

## Step 1: Time Indexing
We first create a vector representing time $t = 1, 2, ..., T$. This is crucial for scaling the variance later.

```python
len_test = cntrl_mat.shape[0]  # e.g., 3 days
one_to_t = np.arange(1, len_test + 1).reshape(len_test, 1) # [[1], [2], [3]]
```

## Step 2: The Cumulative Mean of Predictors
This is the core of the Kerman method. Instead of summing the variance directly, we compute the variance of the average cumulative predictor and then scale it.

```python
# Calculate cumulative sum of X, then divide by t to get the running average
cntrl_cum_mat = np.array(np.array(cntrl_mat.cumsum(axis=0)) / one_to_t)
```

- If $X = [x_1, x_2, x_3]$, cntrl_cum_mat at $t=3$ is $\bar{x}_3 = \frac{x_1+x_2+x_3}{3}$.

## Step 3: Parameter Variance  (Systematic)
We calculate how the uncertainty in $\hat{\beta}$ propagates to the cumulative sum.

The variance of the cumulative sum up to time $t$ is:
$$\text{Var}(Sum) = t^2 \cdot \text{Var}(\text{Average}) = t^2 \cdot (\bar{x}_t \Sigma \bar{x}_t^T)$$

```python
var_params = []
for t in np.arange(len_test):
    # The quadratic form: x_bar * Sigma * x_bar_transpose
    # This gives the variance of the MEAN lift up to time t
    var_t = (cntrl_cum_mat[t,] @ vsigma @ cntrl_cum_mat[t,].T)
    var_params.append(var_t)

var_params = np.array(var_params).reshape(len_test, 1)

# Scale by t^2 to convert "Variance of Mean" to "Variance of Sum"
var_from_params = var_params * one_to_t**2
```

{{note}}Why $t^2$? Because the parameter error is fully correlated across time. If the intercept is overestimated by 1 unit, the cumulative sum over 10 days is overestimated by 10 units. The variance of that error is $10^2 \times \text{Var}(\text{intercept})$.
{{end}}

## Step 4: Observation Variance (Random)
The residual noise $\epsilon$ is assumed to be i.i.d. (independent and identically distributed). Therefore, the variance of the sum is the sum of the variances.

```python
# Linear scaling: T * sigma^2
sigmasq = self.pre_period_model.scale 
var_from_observations = one_to_t * sigmasq
```

## Constructing the Test Statistic
Finally, we combine the point estimate (cumulative sum of lift) and the total variance to construct a T-distribution.

```python
# Point Estimate: Sum of daily lifts
delta_mean = np.array(np.cumsum(lift)).flatten()

# Total Variance: Parameters + Noise
delta_var = var_from_params + var_from_observations
delta_scale = np.sqrt(delta_var).flatten()

# Degrees of Freedom from the pre-test model
delta_df = self.pre_period_model.df_resid 

# Return a frozen T-distribution for inference
return sp.stats.t(delta_df, loc=delta_mean, scale=delta_scale)

# or 
return final_dist = sp.stats.t(df_resid, loc=delta_mean[-1], scale=delta_scale[-1])
```

# Conclusion
By returning a scipy.stats.t object, this method allows us to perform strictly Frequentist inference.
- P-Value: dist.sf(0) (Probability that the true cumulative lift is $\le 0$).
- Confidence Interval: dist.interval(0.95).This approach ensures that our confidence intervals for the total campaign impact are not artificially narrow, properly reflecting the uncertainty in how we estimated the baseline.

# Reference:

---
