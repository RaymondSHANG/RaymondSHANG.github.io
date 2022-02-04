---
layout: post
title: "Multiple Comparisons After ANOVA"
subtitle: "with p adjusted"
date: 2022-02-03 16:18:49
header-style: text
catalog: true
author: "Yuan"
tags: [R,multcomp,lsmeans]
---
>stay hungry

## ANOVA and post hoc ANOVA
In an ANOVA experiment, if we got a pvalue indicating significant(reject null), we need go to pairwise comparisons to find out the extact pairs that showing the significant differences. In this senario, we need to adjust p-values because of multiple comparisons.

If you want to compare all pairs, "Tukey" adjustment could give you the best adjust p-values.

If you have one control group, and want to compare all other groups to this one, using "Dunnett"

Of course, you could always use "Bonferoni"

To apply multiple comparisons you could either use multicomp package or lsmeans package in R.

A good online resource for [multiple comparisons](https://stat.ethz.ch/~meier/teaching/anova/contrasts-and-multiple-testing.html)
## Data preparation

```r
library(ggpubr)
#customer levels and define comparisons
cuslevels <- c("A1","B1","A2","B2")
color_select <- mycolors[c(1,2,5,6)]
currentname = "test"
df = data.frame(y=c(sample(c(1:10), 4, replace = TRUE),
                    sample(c(2:12), 4, replace = TRUE),
                    sample(c(4,11), 4, replace = TRUE),
                    sample(c(5,16), 4, replace = TRUE)),
                group=c(rep("A1",4),rep("A2",4),rep("B1",4),rep("B2",4))) #
df$group = factor(df$group,levels = cuslevels)
#ANOVA
model <- aov(y ~ group, data = df)

```

## Using multcomp package
The following shows how to do various multiple comparisons based on anova model, 

inlcuding:
1. pairwise comparisons(Tukey for all pairwise comparisons, Dunnett for comparing to a control groups)
2. customized contrasts
   
```r
library(multcomp)

#Using Tukey to adjust all pairwise comparisons
summary(glht(model, mcp(group="Tukey"))) #Dunnett

#Contrast prepare:
c1 <- c(-1, 1, 0, 0) # B1 vs. A1
c2 <- c(0, 0, -1, 1) # B2 vs. A2
c3 <- c(-1, 0, 1, 0) # A2 vs. A1
c4 <- c(0, -1, 0, 1) # B2 vs. B1
comparisons_adjust <- summary(glht(model,mcp(group=rbind(c1,c2,c3,c4))), test = adjusted("free"))#“single-step”, “Shaffer”, “Westfall”, “free”, “holm”, “hochberg”, “hommel”, “bonferroni”, “BH”, “BY”, “fdr”, “none”

# Notice that it is 'rbind' above
# If the degrees-of-freedom of all contrasts are identical,
# using multcomp’s method free is more powerful than simply using the Bonferroni-Holm method. 
# 'free' is a generalization of the Bonferroni-Holm method that takes the correlations 
# among the model parameters into account and uniformly more powerful.
#summary(glht(model, mcp(group=rbind(c1,c2,c3,c4))),test = adjusted("free"))

#Extract adjusted p-values
comparisons_adjust$test$pvalues
```


## Using lsmeans package
```r

contr2 <- list("B1 vs. A1"=c(-1, 1, 0, 0),
               "B2 vs. A2"=c(0, 0, -1, 1),
               "A2 vs. A1"=c(-1, 0, 1, 0),
               "B2 vs. B1" = c(0, -1, 0, 1),
               "B vs A" = c(-0.5,0.5,-0.5,0.5),
               "2 vs 1" = c(-0.5,-0.5,0.5,0.5))
lsm <- lsmeans(model, ~group)
#Pairwise comparisons
##Tukey,
contrast(lsm, alpha=0.05, method="pairwise", adjust='Tukey')
##trt.vs.ctrl
contrast(lsm, alpha=0.05, method="trt.vs.ctrl", adjust='Dunnett') #sidak

#Contrasts and p-adjustments
comparisons_noadjust <- lsmeans::contrast(lsm, contr2)
comparisons_adjust <- lsmeans::contrast(lsm, contr2,adjust = "fdr")#sidak,Bonferroni,holm,BH,...

#Output/save the results
comparisons_lsmeans_df <- as.data.frame(comparisons_noadjust)
# If we want the 'free' adjustment in multcomp package
comparisons_adjust <- summary(as.glht(lsmeans::contrast(lsm, contr2)), 
                              test = adjusted("free"))

comparisons_df$padj_free <- comparisons_adjust$test$pvalues

#Save results and plot
write.csv(comparisons_df,file=paste0(currentname,"_comparisons.txt"))

plot_current <- ggbarplot(df, x = "group", y = "y",fill="group",
                            add = "mean_se", error.plot = "upper_errorbar",
                            palette = color_select)+
                            ggtitle(currentname)

```
---
