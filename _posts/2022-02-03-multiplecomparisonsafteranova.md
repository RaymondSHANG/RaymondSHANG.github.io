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

If you want a confidence interval band, use "scheffe", which is available in "DescTools" package and "emmeans" package

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

## Combination of lsmeans, afex, and multcomp
Analysis of Factorial Experiments (afex): Convenience functions for analyzing factorial experiments using ANOVA or mixed models. aov_ez(),aov_car(), and aov_4() allow specification of between, within (i.e., repeated-measures), or mixed (i.e., split-plot) ANOVAs for data in long format (i.e., one observation per row), automatically aggregating multiple observations per individual and cell of the design

Combination of afex and lsmeans could used for more flexible custermized comparisons, such as contrasts in 2-way anova.
An example code from [Henrik](https://stats.stackexchange.com/questions/157022/testing-contrast-in-two-way-anova-using-multcomp) in stackexchange

```r
library(afex)
require(lsmeans)
require(multcomp)
data(obk.long)
# Step 1: set up the model using afex
# but use return = "aov" to obtain an object lsmeans can handle.
fit <- aov.car(value~treatment*gender + Error(id),data=obk.long, return = "aov")


# Step 2: set up reference grid on which we 
# we can test any type of tests using lsmeans functionality:
(ref1 <- lsmeans(fit, c("treatment", "gender")))
##  treatment gender   lsmean        SE df lower.CL upper.CL
##  control   F      4.333333 0.8718860 10 2.390650 6.276016
##  A         F      4.500000 0.8718860 10 2.557317 6.442683
##  B         F      5.833333 0.6165165 10 4.459649 7.207018
##  control   M      4.111111 0.7118919 10 2.524917 5.697305
##  A         M      8.000000 0.8718860 10 6.057317 9.942683
##  B         M      6.222222 0.7118919 10 4.636028 7.808416
## 
## Confidence level used: 0.95 

# we simply define the contrasts as a list on the reference grid:
c_list <- list(c1 = c(0, -1, 1, 0, 0, 0),
               c2 = c(0, -0.5, 0.5, 0, -0.5, 0.5))

# because we want to control for Type I errors we test this using
# the Bonferroni-Holm correction
summary(contrast(ref1, c_list), adjust = "holm")
##  contrast   estimate        SE df t.ratio p.value
##  c1        1.3333333 1.0678379 10   1.249  0.4805
##  c2       -0.2222222 0.7757662 10  -0.286  0.7804
## 
## P value adjustment: holm method for 2 tests 

# alternatively, we can pass it to multcomp for even cooler corrections:
summary(as.glht(contrast(ref1, c_list)), test = adjusted("free"))
## Note: df set to 10
## 
##   Simultaneous Tests for General Linear Hypotheses
## 
## Linear Hypotheses:
##         Estimate Std. Error t value Pr(>|t|)
## c1 == 0   1.3333     1.0678   1.249    0.359
## c2 == 0  -0.2222     0.7758  -0.286    0.780
## (Adjusted p values reported -- free method)

```

## DIY Contrasts
I found a very good online resourse by [Rose Maier](http://rstudio-pubs-static.s3.amazonaws.com/65059_586f394d8eb84f84b1baaf56ffb6b47f.html) published in 2015, demostrating how R contrasts work in lm() models.

This article also showed how to DIY your own contrasts. 
>"contrasts() command isn’t actually expecting contrast weights, like you might think. It’s built for coding schemes (like traditional dummy coding), so it’s actually expecting the inverse of the matrix of desired contrast weights, which can be thought of as the contrast coding scheme rather than the contrast weights themselves (eye roll). This is very frustratingly not at all clear in the help documentation for the function, or in the many webpages providing examples and tutorials on how to use it."

Thus, to apply contrast weights, you’ll need to give it the inverse of your matrix of weights.

>For example, let’s say we wanted to compare Control to A, B, and C (contrast 1), and then compare A to B (contrast 2), and then A to C (contrast 3). Note that these are not orthogonal.
>1. Specfiy the weights for your contrasts (and be sure to check the order of the levels of the factor, so your weights will line up properly)
>2. Create a temporary matrix with each contrast as one row. The top row (for the constant) should be 1/j for j groups.
>3. Get the inverse of that temporary matrix.
>4. The first column of the inverse will be all 1’s. Drop that first column. The remaining columns are your contrast matrix.

After geting your contrast matrix, you can apply that matrix to the factor itself in the dataframe using contrasts(data$factor) <- mat, or you can just specify it as part of the model using the contrasts argument in lm().


### Prepare data 2way anova
```r
# first, generate some random data
n <- 128
cond <- gl(4, n/4, labels=c("A", "B", "C","Control"))
dose <- gl(4, 4, length=n, labels=c("None", "Low", "Med", "High"), ordered=TRUE)
data <- data.frame(cond=cond, dose=dose)
data$y <- ifelse(data$cond=="Control", rnorm(n, mean=100, sd=10), 
                 ifelse(data$cond=="A", rnorm(n, mean=110, sd=10),
                        rnorm(n, mean=120, sd=10)))
data$y <- ifelse(data$dose=="Low", data$y + 10, 
                 ifelse(data$dose=="Med", data$y + 15,
                        data$y))

str(data)
```
### DIY your contrasts: nonorthogonal contrasts
```r
# specify contrast weights
levels(data$cond)
## [1] "Control" "A"       "B"       "C"
# create temporary matrix
c1 <- c(-1 , 1/3, 1/3,  1/3)
c2 <- c( 0,   1,  -1,    0 )
c3 <- c( 0,   1,   0,   -1 )
mat.temp <- rbind(constant=1/4, c1, c2, c3)
mat.temp
##           [,1]      [,2]       [,3]       [,4]
## constant  0.25 0.2500000  0.2500000  0.2500000
## c1       -1.00 0.3333333  0.3333333  0.3333333
## c2        0.00 1.0000000 -1.0000000  0.0000000
## c3        0.00 1.0000000  0.0000000 -1.0000000
# get the inverse of that matrix
mat <- solve(mat.temp)
mat
##      constant    c1            c2            c3
## [1,]        1 -0.75 -5.551115e-17  2.775558e-17
## [2,]        1  0.25  3.333333e-01  3.333333e-01
## [3,]        1  0.25 -6.666667e-01  3.333333e-01
## [4,]        1  0.25  3.333333e-01 -6.666667e-01
# drop the first column
mat <- mat[ , -1]

model6 <- lm(y ~ cond, data=data, contrasts=list(cond = mat))
summary(model6)
## 
## Call:
## lm(formula = y ~ cond, data = data, contrasts = list(cond = mat))
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -27.3811  -8.6832  -0.2297   7.5401  24.0410 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  118.578      1.038 114.195  < 2e-16 ***
## condc1        12.912      2.398   5.384 3.51e-07 ***
## condc2        -8.609      2.937  -2.931  0.00402 ** 
## condc3        -8.758      2.937  -2.982  0.00345 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 11.75 on 124 degrees of freedom
## Multiple R-squared:  0.2469, Adjusted R-squared:  0.2287 
## F-statistic: 13.55 on 3 and 124 DF,  p-value: 1.056e-07

# double check your contrasts
attributes(model6$qr$qr)$contrasts
## $cond
##            c1            c2            c3
## Control -0.75 -5.551115e-17  2.775558e-17
## A        0.25  3.333333e-01  3.333333e-01
## B        0.25 -6.666667e-01  3.333333e-01
## C        0.25  3.333333e-01 -6.666667e-01
```
<mark> <b>Note</b> that the contrasts above is not as expected for c2 and c3, which should be A-B, and A-C in the design, rather than (A+C)-(2B) and (A+B)-(2C). For (A+C) - (2B), it is the difference between A and B controlling for the difference between B and C, <b>NOT</b> the direct difference of A and B. 
That's why Rose Maier suggested always check your contrasts in your final model</mark>

>Remember c2 and c3 are nonorthogonal contrasts (which is why the weights look so messed up), and what we see in the regression coefficients is always the unique effect of each predictor. So we’re seeing the difference between A and B controlling for the difference between A and C, and vice versa. Both c2 and c3 are orthogonal to c1, so it doesn’t affect our interpretation of the others.



### DIY your contrasts: orthogonal contrasts

```r
# specify contrast weights
levels(data$dose)
## [1] "None" "Low"  "Med"  "High"
NLvMH <- c(-1/2, -1/2, 1/2, 1/2)
NvL <- c( 1,  -1,   0,    0 )
MvH <- c( 0,   0,   1,   -1 )

# create temporary matrix
mat.temp <- rbind(constant=1/4, NLvMH, NvL, MvH)
mat.temp
##           [,1]  [,2] [,3]  [,4]
## constant  0.25  0.25 0.25  0.25
## NLvMH    -0.50 -0.50 0.50  0.50
## NvL       1.00 -1.00 0.00  0.00
## MvH       0.00  0.00 1.00 -1.00

# get the inverse of that matrix
mat <- solve(mat.temp)
mat
##      constant NLvMH  NvL  MvH
## [1,]        1  -0.5  0.5  0.0
## [2,]        1  -0.5 -0.5  0.0
## [3,]        1   0.5  0.0  0.5
## [4,]        1   0.5  0.0 -0.5

# drop the first column
mat <- mat[ , -1]

model7 <- lm(y ~ dose, data=data, contrasts=list(dose=mat) )
summary(model7)
## 
## Call:
## lm(formula = y ~ dose, data = data, contrasts = list(dose = mat))
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -33.751  -8.190   0.040   7.813  25.270 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  118.578      1.076 110.187  < 2e-16 ***
## doseNLvMH      3.179      2.152   1.477  0.14215    
## doseNvL       -8.723      3.044  -2.866  0.00489 ** 
## doseMvH       13.232      3.044   4.347 2.84e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 12.18 on 124 degrees of freedom
## Multiple R-squared:  0.1911, Adjusted R-squared:  0.1715 
## F-statistic: 9.765 on 3 and 124 DF,  p-value: 7.859e-06

# double check your contrasts
attributes(model7$qr$qr)$contrasts
## $dose
##      NLvMH  NvL  MvH
## None  -0.5  0.5  0.0
## Low   -0.5 -0.5  0.0
## Med    0.5  0.0  0.5
## High   0.5  0.0 -0.5
```


### Running Fewer than J-1 Contrasts for J Groups

You can get j-1 orthogonal contrasts out of a factor with j levels. Is it okay to run fewer than j-1 contrasts?

>Yep. If you want to save time and only specify the contrast(s) you care about, you can do that, and R will come up with some orthogonal contrasts to fill in the rest. What you won’t be able to do is take the inverse of your contrast weights; you can only take the inverse of a square matrix, and if you have fewer than j-1 contrasts, your temporary matrix won’t be square. But remember: as long as your contrasts are orthogonal, your t-tests will all be fine even if you don’t do the inverse thing. So go ahead and just use the contrasts you want directly with the contrasts() function, and be aware that your contrast estimates may not accurately reflect the differences between group means.

>Note that if you add fewer than j-1 contrasts to the contrasts argument in lm(), it will NOT fill out the remaining contrasts for you. Rather, any group differences other than those represented in your contrast will get lumped into the error term!

```r
# A v B is the only contrast I care about
AvB <- c(0, 1, -1, 0)
mat <- cbind(AvB)
mat
contrasts(data$cond) <- mat
## model8 is OK
model8 <- lm( y ~ cond, data=data)
summary(model8)
## 
## Call:
## lm(formula = y ~ cond, data = data)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -27.3811  -8.6832  -0.2297   7.5401  24.0410 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  118.578      1.038 114.195  < 2e-16 ***
## condAvB       -4.304      1.469  -2.931 0.004022 ** 
## cond           7.030      2.077   3.385 0.000953 ***
## cond           9.425      2.077   4.538 1.32e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 11.75 on 124 degrees of freedom
## Multiple R-squared:  0.2469, Adjusted R-squared:  0.2287 
## F-statistic: 13.55 on 3 and 124 DF,  p-value: 1.056e-07
attributes(model8$qr$qr)$contrasts
## $cond
##         AvB                      
## Control   0 -0.7071068 -0.5000000
## A         1  0.4714045 -0.1666667
## B        -1  0.4714045 -0.1666667
## C         0 -0.2357023  0.8333333

#Model9 is not!
model9 <- lm( y ~ cond, data=data, contrasts=list(cond=mat))
summary(model9)
## 
## Call:
## lm(formula = y ~ cond, data = data, contrasts = list(cond = mat))
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -33.101  -9.173  -0.838   8.166  30.238 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  118.578      1.156 102.610   <2e-16 ***
## condAvB       -4.304      1.634  -2.634   0.0095 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 13.07 on 126 degrees of freedom
## Multiple R-squared:  0.05218,    Adjusted R-squared:  0.04466 
## F-statistic: 6.937 on 1 and 126 DF,  p-value: 0.009501
```
---
