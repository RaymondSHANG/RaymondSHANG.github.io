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
#### Lasso identified Beta-blockers' influence on AD treatment
To quickly explore factors influencing AD progression, I categorized patients with at least two visits as fast or slow progressors. Logistic regression models were then built to evaluate the effect of various factors distinguishing these groups. Variables with over 20% missing values were excluded.

Four modeling approaches were employed: full logistic regression, Lasso regression (hyperparameter tuning via 10-fold cross-validation and the 1SE rule), Ridge regression (hyperparameter tuning via 10-fold cross-validation and minimum error), and elastic net regression (hyperparameter tuning via RMSE).

Lasso regression, a technique particularly effective for feature selection in high-dimensional datasets, achieves variable selection by shrinking the coefficients of less influential variables to zero. The variables selected by the Lasso model, along with their associated coefficients, are shown in Figure B. These results indicate that while certain factors are directly linked to AD severity, others, such as beta-blocker medications, appear to correlate with treatment outcomes. This observation implied that specific medical treatments might possess the capacity to slow the rate of AD progression. Therefore, we conducted further research to explore and identify medical treatments that could potentially modify the course of AD progression.

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

#### Medication for Diabetes, Hypertension, Hyperlipidemia, and Inflammation
Beta-blockers are commonly used in the treatment of hypertension. Our lab's literature-based method, targeted-risk AD prevention (TRAP), identified diabetes, dyslipidemia, hypertension, and inflammation as key drivers of AD risk.Building upon these insights, I proceeded to investigate the rate of cognitive decline using generalized linear models (GLMs), transitioning from the initial Lasso logistic regression modeling. This methodological change was implemented to leverage the capacity of GLMs to model the continuous range of cognitive function measures, enabling a more detailed examination of the relationships between medication use and the degree of cognitive decline..



---
