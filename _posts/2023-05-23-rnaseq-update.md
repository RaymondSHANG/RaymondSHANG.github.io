---
layout: post
title: "RNAseq analysis pipline update"
subtitle: "Building an RNA Seq Pipeline for Reproducible Research and Reporting"
date: 2023-05-23 19:31:19
header-style: text
catalog: true
author: "Yuan"
tags: [RNA Sequencing, R, DESeq2, GSEA, Over-representation test,ClusterProfile, Pathview, PCA analysis, Salmon, Pathway analysis,knit, html]
---
{% include linksref.html %}
> From Reads to Reports: RNA-Seq Unveiled

The github repository for this blogpost (including codes and potential documents) could be found at:
{{note}} [RNA Seq Analysis report generation pipeline](<a href="https://github.com/RaymondSHANG/MyRNAPipe" target="_blank" rel="noopener noreferrer">
  https://github.com/RaymondSHANG/MyRNAPipe
</a>) {{end}}

### How to use:
Most of time, we just need to use ```RNASeq_pip.Rmd```. Additionally, there are several required files:
- 1. Sample demographic excel file (such as ```samples_SexTreatment.xlsx``` ) with at least 2 columns 'Sample' and 'Group'. Other columns will be treated as co-variable in the GLM model (~ + Co-factor1 + Co-factor2 + ... + Group).

| Sample       | APOE  | Group     |
|--------------|-------|-----------|
| GTH11051_28  | APOE3 | M.Control |
| GTH11051_29  | APOE3 | M.Control |
| GTH11051_30  | APOE3 | M.Control |

- 2. Put Your Salmon RNASeq counts in a datadir folder
- 3. Generate a separate AnyMeaningfullName.R in your project directory like this:

```R
#
#'''
#Author: Yuan Shang
#Script to render RNA analytic reports
#Input:sampleInfo.xlsx has 'Sample','Group','Color'(optional) as columns
#Specify:dataDir,species,sampleInfo, and projectName in proparams
#Output: RNASeq Analytic report
#'''
rm(list=ls())
if (rstudioapi::isAvailable()) {
  if (require('rstudioapi') != TRUE) {
    install.packages('rstudioapi')
  }else{
    library(rstudioapi) # load it
  }
  wdir <- dirname(getActiveDocumentContext()$path)
}else{
  wdir <- getwd()
}

setwd(wdir)

proparams <- list(projectName = "YourProjectName",
                  date = Sys.Date(),
                  sampleInfo="samples.xlsx",#"sampleInfo_SexAPOE18m.xlsx"
                  dataDir="salmonResult",
                  species="Mouse",
                  outputDir="treatment",
                  wdir=wdir,
                  pairedGroup=FALSE,
                  preloadcount=FALSE,
                  pathwayAnalysis=TRUE
)


rmarkdown::render("/Directory to/RNASeq_pip.Rmd", 
                  params = proparams,
                  output_dir=paste0(wdir,"/",proparams$outputDir),
                  #knit_root_dir=proparams$wdir,
                  output_file=paste0(proparams$projectName,".html"))


proparams2 <- list(projectName = "GTH11051",
                   date = Sys.Date(),
                   sampleInfo="samples.xlsx",#"sampleInfo_SexAPOE18m.xlsx"
                   dataDir="salmonResult",
                   species="Mouse",
                   outputDir="treatment",
                   wdir=wdir,
                   pairedGroup=FALSE,
                   #groupsSelect="Apoe33,Apoe34,Apoe44",
                   preloadcount=TRUE,
                   pathwayAnalysis=TRUE,
                   geneset="../data/TargetedRNASeqPathways.xlsx",
                   subdir="selectedPathway"
)


rmarkdown::render("~/Dropbox/github/MyRNAPipe/RNASeq_pip_TargetPathway.Rmd", 
                  params = proparams2,
                  output_dir=paste0(wdir,"/",proparams2$outputDir,"/",proparams2$subdir),
                  #knit_root_dir=proparams$wdir,
                  output_file=paste0(proparams2$projectName,"_sub",".html"))

```
- 4. Run your ```AnyMeaningfullName.R```, and you will get full reports in html file format.

### Other notes
You may need to update your own ```tx2gene_release95_mouse.txt```, which was used in ```RNASeq_pip.Rmd```.
A sample R code to generate this file was shown below:

```R
library(AnnotationHub)
library(stringr)
library(org.Mm.eg.db)
#BiocManager::install("GenomicFeatures")
library(GenomicFeatures)
gtf <- "~/Dropbox/RaymondTools/cDNA/salmon/Mus_musculus.GRCm38.95.gtf.gz"
txdb.filename <- "~/Dropbox/RaymondTools/cDNA/salmon/Mus_musculus.GRCm38.95.sqlite"
txdb <- makeTxDbFromGFF(gtf)
saveDb(txdb, txdb.filename)

keytypes(txdb)
#[1] "CDSID"    "CDSNAME"  "EXONID"   "EXONNAME" "GENEID"   "TXID"     "TXNAME" 
columns(txdb)
#1] "CDSCHROM"   "CDSEND"     "CDSID"      "CDSNAME"    "CDSPHASE"   "CDSSTART"   "CDSSTRAND"  "EXONCHROM"  "EXONEND"    "EXONID"     "EXONNAME"   "EXONRANK"   "EXONSTART"  "EXONSTRAND"
#[15] "GENEID"     "TXCHROM"    "TXEND"      "TXID"       "TXNAME"     "TXSTART"    "TXSTRAND"   "TXTYPE"

k <- keys(txdb, keytype = "TXNAME")
#AnnotationDbi::select(txdb, keys = keys, columns="TXNAME", keytype="GENEID")
t2g <- AnnotationDbi::select(txdb, k,columns=c("GENEID","TXCHROM"),keytype = "TXNAME")
head(t2g)
t2g$TXNAME <- gsub("\\.\\d{+}$","",t2g$TXNAME)   ##Remove version information
t2g$GENEID <- gsub("\\.\\d{+}$","",t2g$GENEID)   ##Remove version information

head(t2g)

# Further map GENEID to 

keytypes(org.Mm.eg.db)
# [1] "ACCNUM"       "ALIAS"        "ENSEMBL"      "ENSEMBLPROT"  "ENSEMBLTRANS" "ENTREZID"     "ENZYME"       "EVIDENCE"     "EVIDENCEALL"  "GENENAME"     "GO"           "GOALL"       
#[13] "IPI"          "MAP"          "OMIM"         "ONTOLOGY"     "ONTOLOGYALL"  "PATH"         "PFAM"         "PMID"         "PROSITE"      "REFSEQ"       "SYMBOL"       "UCSCKG"      
#[25] "UNIGENE"      "UNIPROT" 
kt <- unique(t2g$GENEID)

ensembl2symbol <-AnnotationDbi::select(org.Mm.eg.db, 
                                       keys = unique(t2g$GENEID),
                                       columns = c("ENTREZID", "SYMBOL","ENSEMBL"),
                                       keytype = "ENSEMBL")
ensembl2symbol <- ensembl2symbol[!duplicated(ensembl2symbol$ENSEMBL),]
for(i in 1:length(ensembl2symbol$ENSEMBL)){
  if(is.na(ensembl2symbol$ENTREZID[i])){
    ensembl2symbol$ENTREZID[i] <- paste0("UNKNOWN_",ensembl2symbol$ENSEMBL[i])
  }
  if(is.na(ensembl2symbol$SYMBOL[i])){
    ensembl2symbol$SYMBOL[i] <- paste0("UNKNOWN_",ensembl2symbol$ENSEMBL[i])
  }
}
#length(unique(ensembl2symbol[duplicated(ensembl2symbol$ENSEMBL),]$ENSEMBL))
#ensembl2symbol[duplicated(ensembl2symbol$ENSEMBL) | duplicated(ensembl2symbol$ENSEMBL,fromLast=TRUE) ,]
t2g_all <- merge(t2g,ensembl2symbol,by.x="GENEID",by.y="ENSEMBL")
colnames(t2g_all)
t2g_all <-t2g_all[,c(2,1,4,5,3)]
#colnames(t2g_all) <- c("TXNAME","ENSEMBL","ENTREZID","SYMBOL")
tmp = t2g_all[t2g_all$TXCHROM=="MT",]
tmp$SYMBOL = sub("UNKNOWN_","mt-",tmp$SYMBOL)
tmp$ENTREZID = sub("UNKNOWN_","mt-",tmp$ENTREZID)
for(i in 1: length(tmp$SYMBOL)){
  if(!startsWith(tmp$SYMBOL[i],"mt-EN")){
    tmp$SYMBOL[i] <- paste0("mt-",str_to_title(tmp$SYMBOL[i]))
  }
}
tmp$SYMBOL = sub("mt-Cox","mt-Co",tmp$SYMBOL)
tmp$SYMBOL = sub("mt_","mt-",tmp$SYMBOL)
tmp$ENTREZID = sub("mt_","mt-",tmp$ENTREZID)

t2g_all[rownames(tmp),] <- tmp
write.csv(t2g_all,file="~/Dropbox/RaymondTools/DEG/tx2gene_release95_mouse.txt")

```
---
