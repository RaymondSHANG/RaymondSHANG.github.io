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
  
  $y_j=m+\sum{h_i*z_{ij}} + \epsilon_j$         (1)

  Here: $z_{ij}=\dfrac{g_{ij}-2*p_j}{\sqrt{2p_j(1-p_j)}}$, $p_j$ is MAF for $SNP_j$, $z_{ij}$ is normalized genotypes, so $var(z_{ij})==1$.\
  Y is normalized to have variance 1. $\epsilon_j$ is from environmental
- If SNPs were independent to each other, $COV(Z_{i},Z_{k})==0$, then:

  $1 = Var(Y) = \sum{Var(h_i*z_{ij})} + Var(E) = \sum{h_i^2Var(z_{ij})} + Var(E) = \sum{h_i^2} + Var(E)$

  Thus, we could simplely wrote: $h^2=\sum{h_i^2}$
- If $SNP_i$ has MAF $p_i$ and allelic causal effect $\lambda_i$, then the phenotypic variance explained (causally) by the SNP is:
  
  $h_i^2=\frac{2p_i(1-p_i)\lambda_i^2}{\sigma^2(Y)}$

- For a set of SNPs, assuming their LD matrix(Pearson Correlations) is R, and marginal effect sizes for each SNPs is $\beta_i$, then we have:
  
  $h^2=Var(Z^Th) = h^TVar(Z^T)h = h^TRh=(R^{-1}\beta)^TR(R^{-1}\beta)=(\beta)^{T}R^{-1}\beta$

- In Equation (1), most of the time, there are much much more $h_i$ to estimate than the sample size. Thus, people in the field apply a bayesian approach, assuming various prior distributions for $h_i$ in their calculation models. 

- In the task of heritability estimation, $VAR(h_i)$ is also an important variable to estimate. Based on the assumptions for $VAR(h_i)$, there are different heritability models:
  - GCTA model: uniform $VAR(h_i)$ over all SNPs.
  - LDAK model: $VAR(h_i)$ is related to MAF ($p_i$), and LD.
  - The field is still envoling, more complicated models were developped. But after all, they are all models. Some models are useful under certain conditions.

{{note}}

For pedigree studies involving monozygotic (MZ) twins and dizygotic (DZ) twins, heritability can be estimated by Falconer's formula as twice the difference between phenotypic correlations for MZ and DZ twin pairs:<br/>
$H^2=2(r_{MZ}−r_{DZ})$⁠ <br/>
This is easy to be understood:<br/>
$r$ is defined as correlation coefficient, which represents the percentage of the covariance of (X,Y) explained by variance of X, and variance of Y. <br/>
Here, r=genetics+enviroment

{{end}}


# Calculating heritability from GWAS summary statistics using SumHer
The whole process to calculate individual SNP heritability is very complicated, with a series to pre-calculated steps one after another:

1. Get a ref panel based on similar population structures of your GWAS populations.
2. Calculate Taggings based on ref panel. When calculating the tagging file, you must specify a Heritability Model. This step could be parallelized by adding <code>--chr integer</code> to calculate taggings for each chromosome separately, then combine these with the argument <code>--join-tagging output</code>, using the option <code>--taglist tagstems</code>to specify the per-chromosome tagging files.
   
   To save the heritability matrix, add <code>--save-matrix YES</code>. This is required if you want to calculate heritability for each SNP.
3. Calculate per-predictor(per SNP) heritabilities, adding <code>--matrix matrixfile</code>, where <i>matrixfile</i> is is the heritability matrix created by adding <code>--save-matrix YES</code> when Calculating Taggings.
   Usually, we should follow its advice and add <code>--cutoff 0.01</code>, in order to exclude SNPs that individually explain at least 1% of phenotypic variance.

{{note}}<br/>
1. It would be very useful if we could automatically assign an individual's race based on his/her genetics information, and then choose the ref panel automatically, and calculate race specific heritability model parameters.<br/>
2. All applications of SumHer require a tagging file, which records the (relative) expected heritability tagged by each predictor. To create this tagging file requires a (well-matched) Reference Panel(imputed or sequencing data, retaining SNPs with MAF above 0.005 and information score above 0.8)<br/>
3. At the time of Sept 28, 2022. Based on [Dougspeed's suggestions](http://dougspeed.com/calculate-taggings/): <br/>
   When analysing human data, we recommend using the BLD-LDAK Model to estimate SNP Heritability or Heritability Enrichments (for pre-defined categories), using the LDAK-Thin Model to estimate Genetic Correlations, and using the BLD-LDAK+Alpha Model to estimate the selection-related parameter alpha. When analysing non-human data, we recommend always using the LDAK-Thin Model.
{{end}}

# Calculate a GWAS reference panel for Alzheimer's Disease GWAS data analysis
In this poster, I only add GWAS ref panel part. A reference panel is very useful in almost very where in gwas related studies including:
1. genotype imputations: <br/>
   predicting or imputing genotypes that are not directly genotyped in a sample of individuals.<br/>
2. Phasing: <br/>
   Since long haplotype segments are shared between individuals, the reference panels of haplotypes in different human populations can give information for phasing or for choosing tag-SNPs to include on the genotyping arrays. 
3. High-resolution fine-mapping: <br/>
   "Structural variants and short tandem repeats are not always accurately captured by current WGS technologies. Further, there are several regions where WGS-based imputation estimates genotypes inaccurately and custom imputation approaches may be needed to fine-map such regions. 
   For example, the genomic region corresponding to the HLA complex (also known as the major histocompatibility complex (MHC)) is highly pleiotropic for various human traits related to the immune system and infectious disease. The complicated linkage disequilibrium structure in this region prevents WGS-based SNP imputation from unambiguously determining their genotypes. The construction of HLA reference panels and custom imputation methods targeting HLA polymorphisms, such as the software packages SNP2HLA, HIBAG and HLA*IMP, have provided a catalogue of HLA variant–phenotype association maps" [source: Genome-wide association studies](https://www.nature.com/articles/s43586-021-00056-9)
4. Analysing summary statistics:<br/>
   It is used to estimate the correlations between nearby predictors (the linkage disequilibrium). In most cases, the summary statistics will correspond to SNPs (i.e., contain the results from an association study that regressed the phenotype on each SNP individually). In this case, the reference panel should also contain SNP data, from samples ancestrally similar to those used in the association study from which the summary statistics come. 
   If your aim is to perform heritability analysis (i.e., to use SumHer to estimate SNP heritability, heritability enrichments, genetic correlations or the selection-related parameter alpha), then we recommend you use an extensive reference panel.
   If instead your aim is to construct a prediction model (i.e., to use MegaPRS), then the reference panel needs only contain predictors for which you have summary statistics. 

## Choose of GWAS summary dataset
So far, there are 3 big AD related GWAS results to be used.
1. Kunkle 2019, 94437 individuals, $35274$ clinical AD, $59163$ controls.
2. Jensen 2019, $455 258$ individuals including clinical diagnosed AD and AD-by-proxy, and controls(71880 cases including 46613 proxy, 383378 controls including 318246 proxy)
3. Douglas P.Wightman 2021, $1 126 563$ individuals, 90338 AD cases (46613 AD by proxy),1036225 controls (318246 proxy). 
{{note}}
Note that study 3 is an extension of study 2. Study 2 includes Study 1. Study 1 only include clinically diagnosed LOADs.
{{end}}

I checked the BETA(APOE4) , and BETA(APOE2) values of these 3 different summary statistics, they are very different:

|       Study | BETA(APOE4) | BETA(APOE2) |
| ----------: | :---------: | :---------: |
|  Kunkle2019 |    1.20     |    -0.47    |
|  Jensen2019 |    0.16     |   -0.088    |
| Douglas2021 |     NA      |   -0.084    |

The detailed GWAS summary results listed below:

In Kunkle2019:

| Chromosome | Position | MarkerName | Effect_allele | Non_Effect_allele | Beta    | SE     | Pvalue    |
| ---------- | -------- | ---------- | ------------- | ----------------- | ------- | ------ | --------- |
| 19         | 45411941 | rs429358   | T             | C                 | -1.2017 | 0.0189 | 1.17e-881 |
| 19         | 45412079 | rs7412     | T             | C                 | -0.4673 | 0.0305 | 6.401e-53 |

In Jenson 2019:

| uniqID.a1a2     | CHR | BP       | A1  | A2  | SNP      | Z              | P                  | Nsum   | Neff      | dir  | EAF       | BETA                | SE                  |
| --------------- | --- | -------- | --- | --- | -------- | -------------- | ------------------ | ------ | --------- | ---- | --------- | ------------------- | ------------------- |
| 19:45411941_C_T | 19  | 45411941 | C   | T   | rs429358 | 52.4629370156  | 0                  | 436498 | 429961.08 | ?+++ | 0.140838  | 0.162121254326543   | 0.0030902054583474  |
| 19:45412079_T_C | 19  | 45412079 | T   | C   | rs7412   | -21.2330461217 | 4.72954731205e-100 | 436498 | 429961.08 | ?--- | 0.0721435 | -0.0884537292534518 | 0.00416585207541431 |

In Douglas2021:

| chr | PosGRCh37 | testedAllele | otherAllele | z                   | p                     | N      |
| --- | --------- | ------------ | ----------- | ------------------- | --------------------- | ------ |
| 19  | 45411941  | C            | T           | inf                 | 0.0                   | 762544 |
| 19  | 45412079  | T            | C           | -26.694836005944186 | 5.40359796851728e-157 | 762701 |

{{note}}
Based on [mobius and atlas.akhan](https://www.biostars.org/p/319584/) and [Nature genetics: Integration of summary data from GWAS and eQTL studies predicts complex trait gene targets](https://www.nature.com/articles/ng.3538) <br/>

Var(Y|X) is the variance of the residual under linear regression and N is the sample size <br/>
assuming both Y and X are transformed to have unit variance and mean zero<br/>
<br/>
$beta(standardized) = Zscore*\sqrt{\frac{Var(Y|X)}{N}}$ <br/>
<br/>
$Var(Y|X) = \frac{1}{(1 + (Zscore*Zscore)/N)}$ <br/>
<br/>
$Var(beta) = \frac{Var(Y|X)}{N}$ <br/>

Thus <br/>
<br/>
$BETA =  \frac{z}{\sqrt{2*MAF*(1-MAF)*(n+z*z)}}$ <br/>
<br/>
$MAF(APOE2) = 0.072$ as shown in Jenson 2019, so we could calculate BETA(APOE2) based on Douglas 2021.

{{end}}

Since the differences are large, and I decide to choose Kunkle2019 which include only the clinical diagosed data.



---
