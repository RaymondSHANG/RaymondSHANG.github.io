---
layout: post
title: "Combination therapy targeting Alzheimer's disease risk factors is associated with a significant delay in Alzheimer's disease–related cognitive decline"
subtitle: "analysis from real world evidence"
date: 2025-04-25 14:20:14
header-style: text
catalog: true
author: "Yuan"
tags: [Alzheimer's Disease, RWE, Real World Evidence, Combined Therapy, Diabetes, Hypertension, Hyperlipidemia, Inflammation, Regression, GLM, NACC, ADNI,1SE rule, Lasso,hyperparameters tunings, elastic net]
---
{% include linksref.html %}
>Old remedies, new songs for memories redeemed

{{note}} The online version of our recent publication is open accessible:
<br>
<a href="https://alz-journals.onlinelibrary.wiley.com/doi/10.1002/trc2.70074" target="_blank" rel="noopener noreferrer">
  https://alz-journals.onlinelibrary.wiley.com/doi/10.1002/trc2.70074
</a> {{end}}

In this blog, I will show ideas accompany with this paper.

### A Data Scientist's View: The Variability of AD
As a data scientist deeply involved in analyzing the NACC dataset, I observed a striking heterogeneity in the progression of Alzheimer's disease (AD). While some patients progressed rapidly to severe AD within just 2 years, others maintained relatively stable cognitive function for over 7-10 years. This profound variability prompted my investigation into the factors that may influence AD progression, with the ultimate goal of identifying potential therapeutic interventions.

<table>
  <tr>
    <td style="vertical-align: top; width: 70%; padding-right: 10px;">
      <img src="/img/in-post/AD-progression.png" alt="Image Description" style="width: 100%; height: auto;">
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <p>Figure A: Individual trajectories of cognitive status in Alzheimer's disease (AD) patients, measured by the Clinical Dementia Rating - Sum of Boxes (CDR-SB), reveal substantial heterogeneity in progression. Lines represent individual patients, illustrating a spectrum from rapid decline to relative stability over 7+ years, and suggesting a potential therapeutic opportunity to convert rapid progressors to slow progressors.
      </p>
    </td>
  </tr>
</table>

### Co-factors that affect AD risk and progression
#### Lasso identified Beta blockers in the effect of AD treatment
To take a quick look, I labeled all AD patients with at least 2 visits as fast/slow progressors, and then built logistic regression models to evaluate the effect of factors distincguish these two groups. Variables with 20% missing values were dropped.

For logistic regression methods, we built 4 models: full logistic regression model based on all variables; the Lasso regression model, hyperparameters tunings are based on 10-fold cross-validation and 1SE rule; the Ridge regression model, hyperparameters tunings are based on 10-fold cross validation and minimum errors; the elastic net regression model, hyperparameters tunings are based on RMSE.

When the number of variables in a prediction problem is very large, lasso regression tries to avoid overfitting and as a result, some of the coefficients go to zero, thus the corresponding variables are eliminated from the prediction. Figure B showed the variable selected by the Lasso and their coefficients.

<table>
  <tr>
    <td style="vertical-align: top; width: 70%; padding-right: 10px;">
      <img src="/img/in-post/lasso_nacc.png" alt="Image Description" style="width: 100%; height: auto;">
    </td>
    <td style="vertical-align: top; padding-left: 20px;">
      <p>Figure B Variable selected by the Lasso and their coefficients: These parameters could be further divided into 4 categories: 1. Overall status: MMSE, AGE, Live or die; 2. Symptoms: language, psychosis, aphasia; 3. Medical treatments: Referred by professional contact, medications for AD symptoms, beta-adrenergic blocking agent; 4. Others.
      </p>
    </td>
  </tr>
</table>


---
