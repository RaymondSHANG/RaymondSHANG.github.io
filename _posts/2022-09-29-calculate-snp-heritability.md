---
layout: post
title: "Calculate SNP Heritability"
subtitle: "Using SumHer"
date: 2022-09-29 14:59:07
header-style: text
catalog: true
author: "Yuan"
tags: [SumHer,GWAS,Heritability Model,tagging,awk]
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

## Different heritability models
In the additive model<br/>
  $y_j=m+\sum{h_i*z_{ij}} + \epsilon_j$         (1)

We have:<br/>
$1 = Var(Y) = \sum{Var(h_i*z_{ij})} + Var(E) = \sum{h_i^2Var(z_{ij})} + Var(E) = \sum{h_i^2} + Var(E)$ <br/>

If $h_i$ is also treated as a variable, which is most of current GWAS method doing, as there are much much more number of SNPs than number of samples, thus they always assume some prior of $\beta_i$ to fit equation (1) using Bayesian approach. In this approach, heritability(variance contributed by genetics) could be expressed as:<br/>
$E(\sum{h_i^2})$ = $\sum{E(h_i^2)}$ <br/>
For each $h_i$, we have:<br/>
$E(h_i^2) = (E(h_i))^2 + Var(h_i)$         (2) <br/>

In a GWAS study, most all the time, $h_i$ is small, and thus it is assumed:<br/>
$h  \overset{\mathrm{iid}}{\sim}  N(0,\sigma^2)$ <br/>

Thus,$E(h_i)=0$, and eq(2) could be written as:<br/>
$E(h_i^2) =  \sigma^2$         (3) <br/>

In Many softwares, such as GCTA, LDSC, Lassosum, LDpred, they are using uniform $\sigma^2$ model, where $\sigma^2$ is constant for all SNPs.<br/>
Some other software,such as LDAK, assume $\sigma_i^2$ is affected by MAF, LD, and information score, and have a general form of:<br/>
$\sigma_i^2 \propto (p_i(1-p_i))^{1+\alpha}*w_i*r_i$         (4)<br/>

Here: 
1. $p_i$ is MAF of $SNP_i$;<br/>
2. $w_i$ is a SNP specific weight that is a function of the inverse of the LD score of $SNP_i$, so SNPs in regions of low LD contribute more than those in high LD regions;<br/>
3. $r_i \in [0,1]$ is an information score measuring genotype certainty so theat high-quality SNPs contribute more than low-quality ones.<br/>
# Bash script
The below one is from [dougspeed](https://dougspeed.com/wp-content/uploads/refpanel_format_snpher_confounding.txt)

```bash

#Tidy alzheimers summary statistics - the paper tells us there were 17008 cases and 37154 controls
#If possible, compute a chi-squared test statistic (otherwise, could use P)
#For linear regression, can use (beta/SD)^2, for logistic regression, (log.odds/SD)^2
awk < IGAP_stage_1.txt '(NR>1){snp=$3;a1=$4;a2=$5;dir=$6;stat=($6/$7)*($6/$7);n=17008+37154}(NR==1)\
{print "Predictor A1 A2 Direction Stat n"}(NR>1 && (a1=="A"||a1=="C"||a1=="G"||a1=="T") \
&& (a2=="A"||a2=="C"||a2=="G"||a2=="T")){print snp, a1, a2, dir, stat, n}' > alz.txt

#Check for duplicates using the unix functions sort and uniq
#If this command results in no output, it means all predictors are unique
awk < alz.txt '{print $1}' | sort | uniq -d | head
#If there are duplicates, we can remove then using
mv alz.txt alz2.txt; awk '!seen[$1]++' alz2.txt > alz.txt

#Get list of MHC SNPs (from reference panel)
awk < ref.bim '($1==6 && $4>25000000 && $4<34000000){print $2}' > mhc.snps

#Identify large-effect SNPs (those explaining more than 1% of phenotypic variance)
#This command uses the fact that the variance explained by each SNP is stat/(stat+n)
awk < alz.txt '(NR>1 && $5>$6/99){print $1}' > alz.big
#Find SNPs tagging the alzheimers large-effect loci (within 1cM and correlation squared >.1)
./ldak5.linux --remove-tags alz --bfile ref --top-preds alz.big --window-cm 1 --min-cor .1

#Create exclusion files, containing mhc and (for alzheimers) SNPs tagging large-effect SNPs
cat mhc.snps alz.out > alz.excl

#Estimate SNP heritability. To estimate $\htsnp$ there are two steps: first we compute a (1-part) tagfile which contains $q_j + \sum_{l \in N_j} q_l r^2_{jl}$ for each SNP; then we perform the regression to estimate $\htsnp$. To compute an LDAK tagfile, we must first compute LDAK weights; this can take a few hours, but can be efficiently parallelized.


#Calculate LDAK SNP weights; here, we calculate weights separately for each chromosome then merge
#The on-screen instructions explain how to further parallelize (use --calc-weights not --calc-weights-all) 
for j in {1..22}; do
./ldak5.linux --cut-weights alz_chr$j --bfile ref --extract alz.txt --chr $j
./ldak5.linux --calc-weights-all alz_chr$j --bfile ref --extract alz.txt --chr $j
done

#Merge weights across chromosomes
cat alz_chr{1..22}/weights.short > alz.weights

#Calculate the (1-part) LDAK tagfile
./ldak5.linux --calc-tagging alz_ldak --bfile ref --extract alz.txt --weights alz.weights --power -.25 --window-cm 1

#Perform the regression (remember to exclude the MHC / large-effect SNPs)
#Pay attention to the screen output (for this example it says I should add --check-sums NO)
./ldak5.linux --sum-hers alz_ldak --tagfile alz_ldak.tagging --summary alz.txt --exclude alz.excl


#Estimates of SNP heritability will be in alz_ldak.hers

#To instead use the GCTA Model, use --ignore-weights YES and --power -1 when computing the tagfile
#This takes longer (as it uses all SNPs), but we can calculate separately for each chromosome then merge
for j in {1..22}; do
./ldak5.linux --calc-tagging alz_gcta$j --bfile ref --extract alz.txt --ignore-weights YES \
--power -1 --window-cm 1 --chr $j
done
#Join the tagfiles across chromosomes
rm list.txt; for j in {1..22}; do echo "alz_gcta$j.tagging" >> list.txt; done
./ldak5.linux --join-tagging alz_gcta --taglist list.txt

#Then perform the regression (again will have to add --check-sums NO)
./ldak5.linux --sum-hers alz_gcta --tagfile alz_gcta.tagging --summary alz.txt --exclude alz.excl

#Estimate confounding bias. We recommend estimating the scaling factor $C$, but SumHer can also estimate the LDSC intercept $1+A$. There is no need to compute a new tagfile, just add \verb|--genomic-control YES| or \verb|--intercept YES| when performing the regression.


#Estimate the scaling factor C
./ldak5.linux --sum-hers alz_ldak.gcon --tagfile alz_ldak.tagging --summary alz.txt --exclude alz.excl \
--genomic-control YES
#The scaling factor will be in alz_ldak.gcon.extra

#Estimate the intercept 1+A
./ldak5.linux --sum-hers alz_gcta.cept --tagfile alz_gcta.cept.tagging --summary alz.txt --exclude alz.excl \
--intercept YES
#The intercept will be in alz_gcta.cept.extra

```



---
