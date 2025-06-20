---
layout: post
title: "Power Analysis for ANOVA"
subtitle: "Post hoc ANOVA type 2 and type 3 analysis"
date: 2025-06-19 15:29:22
header-style: text
catalog: true
author: "Yuan"
tags: [Power Analysis, Python, R, statsmodels, car, ANOVA, Type II SS (sums of squares), Type III SS, FTestPower, TTestIndPower]
---
{% include linksref.html %}
>Beyond the F-Test: Power's Echo in Post-Hoc Truths
{{note}} Python codes can be found here(<a href="https://github.com/RaymondSHANG/Accelerated-Midlife-Endocrine-and-Bioenergetic-Brain-Aging-in-APOE4-Females/code/power_analysis_python_readingFromExcel.ipynb" target="_blank" rel="noopener noreferrer">
  power_analysis_python_readingFromExcel.ipynb
</a>) {{end}}

{{note}} [Anova – Type I/II/III SS explained](<a href="https://md.psych.bio.uni-goettingen.de/mv/unit/lm_cat/lm_cat_unbal_ss_explained.html" target="_blank" rel="noopener noreferrer">
  is R based blog, explaining detailed differences between type I/II/III ANOVA tests
</a>) {{end}}

# ANOVA Type I/II/III SS
At its core, ANOVA (Analysis of Variance) quantifies how various factors influence the variability observed in a dataset. Pioneered by R. A. Fisher in 1925 for balanced experimental designs, its application becomes more nuanced with unbalanced data—a common occurrence. To address this, statisticians employ different methods for calculating sums of squares, typically referred to as Type I, Type II, and Type III. Though this nomenclature became popular through the SAS statistical package, these distinct approaches are crucial because they evaluate different hypotheses embedded within the data. Deciding which type is appropriate continues to be a point of discussion in the statistical community, as explored by Heer's paper "On the History of ANOVA in Unbalanced, Factorial Designs: The First 30 Years".

## Type I (Sequential Sums of Squares)
Type I sums of squares calculate the variance explained by each factor in the order they are entered into the statistical model. Each subsequent factor's contribution is assessed after accounting for the variance explained by the preceding factors.

- Best Use Cases: Ideal for balanced datasets, nested models, and polynomial regression (where simpler terms are prioritized before more complex ones).
- Insight: Comparing Type I with other sum of squares types can reveal the extent of data imbalance.
- Key Feature: Sequential adjustment, meaning the order of terms in the model matters.

## Type II (Partial Sums of Squares, Hierarchical)
Type II partial sums of squares adjust each effect for all other effects that do not contain the effect in question. This means that main effects are adjusted for other main effects, but typically not for interactions that include them.

- Mechanism: For an effect 'U', it's adjusted for effect 'V' only if 'V' does not entirely encompass 'U'. For instance, in a two-factor model with interaction (A, B, A*B):
    - Main effect A is adjusted for B.
    - Main effect B is adjusted for A.
    - The A*B interaction is adjusted for both A and B.
- Interpretation: Type II essentially allows lower-order terms to explain as much variance as possible, adjusting for one another, before considering higher-order terms. It prioritizes main effects in the absence of significant interactions.
- Distinction: Unlike Type III, Type II generally does not adjust main effects for interactions that include them.

## Type III (Partial Sums of Squares, Fully Adjusted)
Type III partial sums of squares adjust every effect for all other effects in the model, regardless of whether they are contained within higher-order terms. This approach effectively tests the effect of a factor **as if it were added last to the model**, after all other effects have been considered.
- Mechanism: Every effect is adjusted for all other effects present in the model. If a model only contains main effects, Type II and Type III will yield identical results.
- Use Case & Controversy: Useful for comparing main effects even when interactions are present, though many statisticians advise caution or against this practice if interactions are significant, as main effects might not be interpretable in such cases.
- Key Feature: Tests specific hypotheses about population marginal means, assuming all other effects are also in the model.

## Effect of Codings on categorical factors
When you have a categorical variable (like "Genotype" with "APOE3" and "APOE4", or "Time_Condition" with "6M-Reg", "9M-Reg", etc.), statistical models convert these categories into numerical dummy variables. Contrasts define how these dummy variables are coded.

Different contrast types lead to different numerical representations, which in turn define the specific comparisons being made between group means.

- "contr.sum" (Sum Contrasts / Deviation Contrasts):
Compares the mean of each group to the grand mean (the mean of all group means).
For a factor with k levels, the sum of the dummy variable coefficients for that factor is 0. The effect of the last level is inferred from the others.
- "contr.poly" (Polynomial Contrasts):
Used specifically for ordered categorical variables (like your "Time_Condition" if it implies a temporal progression).
It tests for linear, quadratic, cubic, etc., trends across the levels of the factor. These contrasts are inherently orthogonal (uncorrelated) for evenly spaced levels.
- "contr.treat" (Treatment Contrasts / Dummy Coding - R's default for unordered factors):
Compares the mean of each group to the mean of a reference group (usually the first level alphabetically or specified). These are not orthogonal contrasts.


{{important}}
Type III Sum of Squares is highly sensitive to the choice of contrasts, especially in unbalanced designs.For Type III SS to provide meaningful and unambiguous results (where the sum of squares for each effect uniquely reflects its contribution regardless of the order of other factors in the model), the contrasts used to code your categorical variables must be orthogonal (uncorrelated).<br/>
When you have unordered categorical factors (like "Genotype"), using contr.sum in R ensures that the main effects are coded in a way that, when combined with an appropriate model specification, allows their contribution to be assessed independently of (orthogonal to) the interaction terms. If you used contr.treat (R's default) with an unbalanced design and Type III SS, the results for main effects could change depending on which level is chosen as the reference category, making the interpretation problematic.
{{end}}

Below is the R code to calculate Type III SS ANOVA table
```r
options(contrasts = c("contr.sum", "contr.poly"))
library(car)  # for Anova()

df <- data.frame(
  A = factor(c("a1", "a1", "a1", "a2", "a2", "a2", "a2", "a2")),
  B = factor(c("b1", "b2", "b3", "b1", "b2", "b2", "b3", "b3")),
  Y = c(10, 15, 12, 20, 25, 23, 19, 21)
)

mod_full <- lm(Y ~ A * B, data = df)
Anova(mod_full, type = 3)

# Below is step by step on how to calculate type3 SS(A), SS(B), and SS(A:B). Which should be identical to Anova(mod_full,type=3)
rss_full <- sum(resid(mod_full)^2)  # Residual sum of squares
rss_full  # Should match Residuals row in ANOVA

mod_noA <- update(mod_full, . ~ . - A)
rss_noA <- sum(resid(mod_noA)^2)
ss_A <- rss_noA - rss_full

mod_noB <- update(mod_full, . ~ . - B)
rss_noB <- sum(resid(mod_noB)^2)
ss_B <- rss_noB - rss_full

mod_noAB <- update(mod_full, . ~ . - A:B)
rss_noAB <- sum(resid(mod_noAB)^2)
ss_AB <- rss_noAB - rss_full

options(contrasts = c("contr.treatment", "contr.poly"))
```

We could also got the same results using Python
```python
from statsmodels.formula.api import ols
import statsmodels.api as sm

df['A'] = pd.Categorical(df['A'])
df['B'] = pd.Categorical(df['B'])

# Type II ANOVA
model_type2 = ols('Body_Weight ~ C(A) + C(B) + C(A):C(B)', data=df).fit()
anova_table_type2 = sm.stats.anova_lm(model, typ=2)#

# Type III ANOVA
model_type3 = smf.ols('Y ~ C(A, Sum) * C(B, Sum)', data=df).fit()
# Perform Type III ANOVA
anova_table_type3 = sm.stats.anova_lm(model, typ=3)
```

# Effect size calculation
After we got the correct ANOVA table, we could extract ss_effect values for each term (A,B, A:B), and ss_error for the residue term. Then, we could calculate partial eta-squared and convert to Cohen's f, shown below in python

```python
# Function to calculate partial eta-squared and convert to Cohen's f
def calculate_eta_f(ss_effect, ss_error):
    """Calculates partial eta-squared and Cohen's f."""
    if (ss_effect + ss_error) == 0:
        eta_sq_p = 0.0
    else:
        eta_sq_p = ss_effect / (ss_effect + ss_error)
    cohen_f = sqrt(eta_sq_p / (1 - eta_sq_p)) if eta_sq_p < 1 else np.inf
    return eta_sq_p, cohen_f
```

# Power calculation

```python
from statsmodels.stats.power import FTestPower

# Power analysis using FTestPower
# f_genotype would be cohen_f from calculate_eta_f for genotype factor
power_calculator = FTestPower()
alpha = 0.05
power_genotype = power_calculator.solve_power(effect_size=f_genotype, alpha=alpha, df_num=df_genotype, df_denom=df_error)
```

---
