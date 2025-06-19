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


---
