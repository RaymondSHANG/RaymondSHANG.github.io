---
layout: post
title: "Calculate SNP Heritability"
subtitle: "Using SumHer"
date: 2022-09-29 14:59:07
header-style: text
catalog: true
author: "Yuan"
tags: [SumHer,GWAS,Heritability Model,tagging]
---
{% include linksref.html %}
> 夫英雄者，胸怀大志，腹有良谋，有包藏宇宙之机，吞吐天地之志者也。

# Calculating taggings
All applications of SumHer require a tagging file, which records the (relative) expected heritability tagged by each predictor.
Recommendations from [Dougspeed](http://dougspeed.com/calculate-taggings/):
1.  To create this tagging file requires a (well-matched) Reference Panel. Using an extensive panel (e.g., imputed or sequencing data, retaining SNPs with MAF above 0.005 and information score above 0.8).
2. When calculating the tagging file, you must specify a Heritability Model. 
3. When analysing human data:
   using the BLD-LDAK Model to estimate SNP Heritability or Heritability Enrichments (for pre-defined categories) from summary statistics;
   using the LDAK-Thin Model to estimate Genetic Correlations; 
   using the LDAK-Thin Model to analyse individual-level data;
   using the BLD-LDAK+Alpha Model to estimate the selection-related parameter alpha. 
4. When analysing non-human data, always using the LDAK-Thin Model.


---
