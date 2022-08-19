---
layout: post
title: "Generate automatic reports using R markdown"
subtitle: "Parameters from R to yaml to markdown"
date: 2022-08-16 16:41:57
header-style: text
catalog: true
author: "Yuan"
tags: [R,Markdown,yaml,knit,render,Alzheimer's, NACC]
---
{% include linksref.html %}
> Time to summarise

# Intention
One colleague asked me about the potential effects of center drugs on AD, based on [NACC](https://naccdata.org/) dataset. \
The task is easy: 
1. get a contingency table of (drug_Normal,drug_AD,nonDrug_Normal,nonDrug_AD); 
2. run fisher.extact test to check whether taking this drug would affect the rate of developing Alzheimer's Disease; 
3. plot bargraph for visualization.

However, the task is also tedious! Firstly of all, similar contingency table should be generated in different subgroups including Sex, APOE genotypes, and even their interactions. In addition, I might need to do similar task again and again if some one else want to know the effects of other drugs/conditions. In this considerations, I decide to:\
1) Using R markdown to generate the analytic report, export pdf format.This .md file should be general, and stay unchanged.
2) The .md file accept specific parameters, and output pdf reports
3) A second R script(.R) file get the specific drug info and send to .md file to get pdf files.
   
# Challenges
The Challenges include:\
## Send parameters to R markdown file
While .R file could get args through "Rscript --vanila xxx.R args". R markdown does not have this option. However, the "yaml" head of a markdown file could set some specific parameters in param, which could be accessed in R markdown codes.\
Below is from my current .md yaml head.

````r
---
params: 
    feature_input: "NACCADEP"
    date: "08/18/2022"
title: "`NACC Test For `r params$feature_input``"
author: "Raymond Yuan SHANG"
date: "`r params$date`"
output: 
  pdf_document:
    df_print: kable
    highlight: tango
    latex_engine: xelatex
fontsize: 11pt
geometry: margin=1in
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## R Markdown and data process

```
#Get parameter from yaml head
input_para <- params$feature_input
input_para
```

````

{{tip}}
With the above yaml head, the final output file will have a dynamic title and date. <br>
Also, the parameters in params could be called in the code chunk, such as params$inputfeature
{{end}}

## Send parameters to yaml head in .md file
The yaml head above also provide  ways to get flexible document titles, subtitles, dates, etc. However, you centainly won't want to change this <i>params</i> by hand every time. You could update the <i>params</i> when you render the ".md" file, by specifying params = list(param1=xxx1,param2=xxx2).\
Shown below is from my "xxx.R" file, which specify feature_input, and generate pdf report for each feature by render the .md file.\

```r
featureNames <- c("NACCADEP", "NACCAPSY", "NACCAANX")#"NACCLIPL",
setwd("~/Dropbox/human/NACC/NACC_testParameter")
for (currentFeature in featureNames) {
  rmarkdown::render("NACC_testParameter.Rmd", 
                    params = list(feature_input = currentFeature,
                                  date = Sys.Date()),
                    output_file=paste0("output/",currentFeature))
}
```


## Rename Rmarkdown output files
By default, when you knit a XXX.md file, a XXX.pdf (or XXX.html, based on you output styles) will be generated. This is not flexible, as your file will be covered by a second runs with different ".md" call.\
To overcome this, you could either rename your output file, or specify <i>output_file</i> when you call "rmarkdown::render()" function as shown above.\
{{note}}
So far, I do not find a way to specify flexible to render inside a yaml header, and set flexible output_file there. <br>
A close solution is from <a href="https://stackoverflow.com/questions/28500096/r-markdown-variable-output-name">Floris Padt</a> in stackoverflow. However, the output filename is fixed in this solution(Shown below).
{{end}}
````
---
params: 
  sub_title:
    input: text
    label: Sub Title
    value: 'my_Sub_Title_and_File_Name'
title    : "Parameterized_Title_and_output_file"
subtitle : "`r params$sub_title`"
output:
  pdf_document:
    keep_tex: false
knit: (
  function(inputFile, encoding) { 
  
    pSubTitle <- 'This Works!'
  
    rmarkdown::render( 
      input       = inputFile, 
      encoding    = encoding, 
      params      = list(sub_title = pSubTitle),      
      output_file = pSubTitle) })
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## R Markdown

This is an R Markdown document. ....
````
---