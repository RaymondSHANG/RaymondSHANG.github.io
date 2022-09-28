---
layout: post
title: "Construct a GWAS reference panel"
subtitle: "for almost everywhere usage in GWAS related studies"
date: 2022-09-27 14:19:59
header-style: text
catalog: true
author: "Yuan"
tags: [GWAS, plink, 1000 Genomes Project, imputations, summary statistics]
---
{% include linksref.html %}
> I did not mean to learn GWAS, but it happened.

# Heritability 
I wanted to calculate PRS for Alzheimer's, following the tutorials I wrote in May,2022. However, the WGAS summary data I have lack heritability estimation info. Thus, I need to estimate this value ($h^2_{snp}$) using SumHer, as suggested by [Tutorial: a guide to performing polygenic risk score analyses](https://www.nature.com/articles/s41596-020-0353-1).

It turned out that $h^2_{snp}$ is just the surface part of an iceberg:
- First of all, the definition of heritability is complicated: Heritability measures, in a particular population, the proportion of variance of the phenotype that is due to genetic differences between individuals(Defined from [Matti Pirinen, University of Helsinki
](https://www.mv.helsinki.fi/home/mjxpirin/GWAS_course/material/GWAS8.html)).
- Traditionaly, heritability estimation relies on pedigree-based or family-based designs. However, genome-wide significant SNPs cumulatively explain only a small fraction of the heritability estimated from pedigree-based or family-based studies, leading to the so-called ’missing heritability’ problem
- Broad-sense heritability ($⁠H^2$⁠) represents the proportion of phenotypic variance explained by all genetic factors, including additive effects($\sigma_g^2$), dominant effects($\sigma_D^2$) and gene-gene interactions (epistatic effects: $\sigma_I^2$).
- Narrow-sense heritability only refer to additive effects: $h^2=\sigma_g^2/\sigma_P^2$. Thus, in most heritability analysis, predictors that explain more than 1% of phenotypic variance are removed and modelled separately.
- The additive model for phenotype Y vs genetics, a good review by [Noah Zaitlen and Peter Kraft](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3432754/):
  
  $y_j=m+\sum{h_i*z_{ij}} + \epsilon_j$

  Here: $z_{ij}=\dfrac{g_{ij}-2*p_j}{\sqrt{2p_j(1-p_j)}}$, $p_j$ is MAF for $SNP_j$, $z_{ij}$ is normalized genotypes, so $var(z_{ij})==1$.\
  Y is normalized to have variance 1. $\epsilon_j$ is from environmental
- If SNPs were independent to each other, $COV(Z_{i},Z_{k})==0$, then:

  $1 = Var(Y) = \sum{Var(h_i*z_{ij})} + Var(E) = \sum{h_i^2Var(z_{ij})} + Var(E) = \sum{h_i^2} + Var(E)$

  Thus, we could simplely wrote: $h^2=\sum{h_i^2}$
- If $SNP_i$ has MAF $p_i$ and allelic causal effect $\lambda_i$, then the phenotypic variance explained (causally) by the SNP is:
  
  $h_i^2=\frac{2p_i(1-p_i)\lambda_i^2}{\sigma^2(Y)}$

- For a set of SNPs, assuming their LD matrix(Pearson Correlations) is R, and marginal effect sizes for each SNPs is $\beta_i$, then we have:
  
  $h^2=Var(Z^Th) = h^TVar(Z^T)h = h^TRh=(R^{-1}\beta)^TR(R^{-1}\beta)=(\beta)^{T}R^{-1}\beta$



{{note}}

For pedigree studies involving monozygotic (MZ) twins and dizygotic (DZ) twins, heritability can be estimated as twice the difference between phenotypic correlations for MZ and DZ twin pairs:<br/>
$H^2=2(r_{MZ}−r_{DZ})$⁠ <br/>
This is easy to be understood:<br/>
$r$ is defined as correlation coefficient, which represents the percentage of the covariance of (X,Y) explained by variance of X, and variance of Y. <br/>
Here, r=genetics+enviroment

{{end}}



---
