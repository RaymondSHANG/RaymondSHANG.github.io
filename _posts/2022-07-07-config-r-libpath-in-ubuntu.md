---
layout: post
title: "config R libPath in ubuntu"
subtitle: "by specify lib"
date: 2022-07-07 10:17:29
header-style: text
catalog: true
author: "Yuan"
tags: [R,.libPath(),ubuntu,lib]
---
{% include linksref.html %}
After install R,and Rstudio-desktop in ubuntu, I'm trying to work under ubuntu to analyze previous pending jobs.
When, installing the required packages in Rstudio, I got the "installationpath not writable,unable to update packages" info.

Based on [the solution provided in itecnote](https://itecnote.com/tecnote/r-installation-path-not-writable-r-unable-to-update-packages/), I could install by specify the place I want the packages to be installed. If I had the admin right, I could also remove the old packages.

## Install packages by specify installation lib
```R
.libPaths()
#[1] "/home/xxx/R/x86_64xxxx-library/4.2"
#[2] "/usr/local/lib/R/site-library"
#[3] "/usr/lib/R/site-library"
#[4] "/usr/lib/R/library"
install.packages(c("PKG1", "PKG2", "PKG3"),lib = "/home/USER/R/x86_64-pc-linux-gnu-library/X.X") 
BiocManager::install(c("PKG1", "PKG2", "PKG3"),lib = "/home/USER/R/x86_64-pc-linux-gnu-library/X.X")

```

## Remove old lib
We could either remove the older package folders with "sudo rm -rf PKG1". Or we could run R with administrator right, and run remove.package by specifying the place hold the old packages.

```R
#Run R with Administation right
remove.packages(c("PKG1", "PKG2", "PKG3"), lib = "/usr/lib/R/library")
```

---
