---
layout: post
title: "Rmarkdown chunk code options"
subtitle: "Hide code, text output, messages, or plots"
date: 2022-08-31 10:05:30
header-style: text
catalog: true
author: "Yuan"
tags: [R, Markdown, chunk code, options]
---
{% include linksref.html %}

This blog is mainly from [rmarkdown-cookbook/](https://bookdown.org/yihui/rmarkdown-cookbook/hide-one.html) and [Yihui Xie's Book](https://yihui.org/knitr/options/#code-chunk)

## Settings in chunk code
````
## Global Settings:

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = FALSE)
knitr::opts_chunk$set(message = FALSE)

#Settings
options(stringsAsFactors=FALSE)
options(warn=-1)
```

## Hide all output:
This will run the code, but not the chunk output in the output document
```{r, include=FALSE}
1 + 1
```

## Hide source code:
This will hide source code in the output
```{r, echo=FALSE}
1 + 1
```

## Hide text output (you can also use `results = FALSE`):

```{r, results='hide'}
print("You will not see the text output.")
```

## Hide messages:

```{r, message=FALSE}
message("You will not see the message.")
```

## Hide warning messages:

```{r, warning=FALSE}
# this will generate a warning but it will be suppressed
1:2 + 1:3
```

## Hide plots:

```{r, fig.show='hide'}
plot(cars)
```

Note that the plot will be generated in the above chunk. It is
just not displayed in the output.

## More specific settings:

```{r, echo=c(4, 5), message=c(1, 4), warning=2:3}
# one way to generate random N(0, 1) numbers
x <- qnorm(runif(10))
# but we can just use rnorm() in practice
x <- rnorm(10)
x

for (i in 1:5) message('Here is the message ', i)

for (i in 1:5) warning('Here is the warning ', i)
```

Only the 4th and 5th expressions are shown/echoed (note that a comment also counts as one expression). For the messages, only the 1st and 4th were shown, etc.

## chunk options could also written at the begining 
```{r}
#| my-chunk, echo = FALSE, fig.width = 10,
#| fig.cap = "This is a long long
#|   long long caption."

plot(cars)
```

````

## Using variables in chunk code
````
Use variables in chunk options
```{r}
my_width <- 7
```

```{r, fig.width=my_width}
plot(cars)
```

#If-else statement
```{r}
fig_small <- FALSE  # change to TRUE for larger figures
width_small <- 4
width_large <- 8
```

```{r, fig.width=if (fig_small) width_small else width_large}
plot(cars)
```
#Run a code chunk only if a required package is available
```{r, eval=require('leaflet')}
library(leaflet)
leaflet() %>% addTiles()
```
# Set root.dir
```{r}
#Set working directory
#The working directory starts to change in the following chunks after the chuck that contains knitr::opts_knit$set(root.dir = ...).
knitr::opts_knit$set(root.dir = params$wdir)

```

````
---
