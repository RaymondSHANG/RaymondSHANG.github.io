---
layout: post
title: "Eigengene: explaination and reproduce"
subtitle: "WGCNA"
date: 2022-02-03 00:35:51
header-style: text
catalog: true
author: "Yuan"
tags: [R,WGCNA,SVD,PCA,Eigengene]
---
>wake up!

The concept of eigengene was first introduced by [Orly et al](https://www.pnas.org/content/97/18/10101). By definition, eigengenes are the right singular vectors of the SVD of the expression matrix. This concept become popular when WGCNA used it to represent the overal expression of a module. Actually, eigengene and WGCNA modules are perfect matchings! In a specific WGCNA module, all genes were in the same cluster by a distance which is closely related to the correlation matrix. In this senario, all genes in one module are highly correlated, and thus their PC1 could best represent their overal expression.

In [WGCNA](https://github.com/cran/WGCNA/blob/85d34a5cd9945f44425f5490ad0cecda1aa5ecf7/R/Functions.R), by default and in most of times, the first principle component is used to represent the expression values of a group of genes, under the name of eigen gene. But sometimes, we just want to calculate eigengene genes, how to do that in a simple way?

After going through the codes in WGCNA functions, I have two solutions for this:
data_PCA is row as genes,and columns as samples
1. Using WGCNA
    ```r
    library(WGCNA)
    ##calculate eigen gene value of all MT genes
    complex_colors_genes <- rep("grey",dim(data_PCA)[1])
    MEList_genes <- moduleEigengenes(as.matrix(t(data_PCA)), colors = complex_colors_genes)
    gene_eigen <- MEList_genes$eigengenes[,"MEgrey"]
    ```
2. Using SVD
    ```r
    eigengenes <- svd(t(scale(t(data_PCA))),nu=1,nv=1)$v
    scaledExpr <- scale(t(data_PCA))
    averExpr <- rowMeans(scaledExpr, na.rm = TRUE)
    if(cor(averExpr,eigengenes) < 0){
        eigengenes <- -eigengenes
    }
    ```
---
