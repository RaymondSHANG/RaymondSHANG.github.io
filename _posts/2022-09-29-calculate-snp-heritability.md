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

<span style="text-align:center"><b>Model Comparisons</b></span>

| Model            | $\alpha$ | $w_i$                                                    | Comments                                                                                                                                               | implementation in LDAK software                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ---------------- | -------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <b>GCTA</b>      | -1       | 1                                                        | Constatnt expected varianace of $h_i$ accross genome                                                                                                   | In LDAK, this model is achieved by adding the options --ignore-weights YES and --power -1 when Calculating Kinships or Calculating Taggings.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| <b>LDAK</b>      | -0.25    | $w_i$                                                    | Adding LD, for high LD regions, $w_i$ should be smaller                                                                                                | In LDAK, this model is achieved by adding the options --weights <weightsfile> and --power -0.25 when Calculating Kinships or Calculating Taggings, where <weightsfile> provides the LDAK Weightings.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| <b>LDAK-Thin</b> | -0.25    | $I_iw_i$                                                 | $I_i$ indicates whether SNP remains after thinning for duplicate SNPs                                                                                  | In LDAK, the LDAK-Thin Model is achieved by adding the options --weights <weightsfile> and --power -0.25 when Calculating Kinships or Calculating Taggings, where <weightsfile> gives weight one to the SNPs that remain after Thinning Predictors with options --window-kb 100 and --window-prune 0.98.                                                                                                                                                                                                                                                                                                                                                                            |
| <b>LDSC</b>      | -1       | $\sum_{j=1}^{74}{\tau_ja_{ji}} + \tau_{75}$              | $a$ values could be derived from <a href="https://alkesgroup.broadinstitute.org/LDSCORE/">LDSC website</a>                                             | To implement this model in LDAK, you should first download the Baseline LD annot.gz files from the LDSC website, then use these to make files called baselineLD1, baselineLD2, ..., baselineLD74 (where baselineLDk has two columns, providing the SNP names then values of Annotation k). You would then use Calculate Taggings adding --annotation-number 74, --annotation-prefix baselineLD, --ignore-weights YES and --power -1. <br/> Equivalently, you could make an extra file called baselineLD75 that contains the names of all SNPs, then replace --annotation-number 74 and --annotation-prefix baselineLD with --partition-number 75 and --partition-prefix baselineLD. |
| <b>BLD-LDAK</b>  | -0.25    | $\sum_{j=1}^{64}{\tau_jb_{ji}} + \tau_{65}w_i+\tau_{66}$ | where b1, b2, ..., b64 are the non-MAF annotations from the Baseline LD Model and $w_i$ is the LDAK weighting (computed using only high-quality SNPs). | 1. Download the files bld1, bld2, ..., bld64 from the <a href="https://storage.googleapis.com/broad-alkesgroup-public/LDSCORE/1000G_Phase3_baselineLD_v2.1_ldscores.tgz">BLD-LDAK Annotations</a>.<br/>2. Next calculate the LDAK Weightings and rename them bld65.<br/>3. Calculate Taggings adding --annotation-number 65, --annotation-prefix bld, --ignore-weights YES and --power -0.25.                                                                                                                                                                                                                                                                                       |

{{note}}<br/>
To get BLD-LDAK weightsThis part is directly from Dougspeed's website.<br/>
1. Get the 64 annotations: downloaded the folder 1000G_Phase3_baselineLD_v2.1_ldscores.tgz from https://data.broadinstitute.org/alkesgroup/LDSCORE. Within this folder, the .annot.gz files contain the 74 annotations of the Baseline LD Model. The BLD-LDAK Model uses Annotations 1-58 and 59-64.<br/>

2. Extracted all 74 annotations (plus Annotation 0, the base category) using the following commands:<br/>


3. Exclude the 10 MAF bins using these two commands:<br/>
   
{{end}}

```bash

rm bld0 base{1..74}
for j in {1..22}; 
do 
   gunzip -c baselineLD_v1.1/baselineLD.$j.annot.gz | awk '(NR>1){for(j=1;j<=74;j++){if($(5+j)!=0){print $1":"$2, $(5+j) >> "base"j}}print $1":"$2 >> "bld0"}'; 
done

for j in {1..58}; do cp base$j > bld$j; done
for j in {59..64}; do cp base$((10+j)) bld$j; done
```

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
