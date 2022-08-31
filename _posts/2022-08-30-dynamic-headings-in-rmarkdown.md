---
layout: post
title: "Dynamic Headings in Rmarkdown"
subtitle: "Output text as raw Markdown content"
date: 2022-08-30 17:22:55
header-style: text
catalog: true
author: "Yuan"
tags: [R,markdown,dynamic heading,knitr]
---
{% include linksref.html %}
> I have nothing to say. Fingers crossed

I'm writing a R markdown file to process bulky RNAseq analysis and generate reports for general lab experiments. To better presents the results in the final output reports, I need to generate dynamic subheadings based on the experimental designs.\
Gladly, I found one post online by [ R | Science Loft](https://www.r-bloggers.com/2020/07/programmatically-create-new-headings-and-outputs-in-rmarkdown/).

Basically, the most critical steps is to set <b>"Output text as raw Markdown content"</b> by Set chunk option ```results="asis"```.\
Meanwhile, with results="asis", you could also do much more flexible decorations on your outputs using either html or markdown settings.

Some key points:
1. Set chunk option ```results="asis"```
2. Create headings using ```cat()``` function in loop
3. Call ```print()``` on the code to generate desired output (tables, plots, code output, etc)
4. Wrap the output within appropriate HTML tags
    1. For ```DT::datatable``` outputs initiate dependencies previously and use ```htmltools::tagList()```
    2. For neatly rendered console outputs, wrap the code with ```htmltools::pre()``` and ```htmltools::code()```

## 1. Example for creating tables in loop:

The tables printed in loop don’t render in the final document. A solution is to create an initial datatable output is needed to load some magic dependencies. To hide this chunk from the final output though, you can set chunk option ```include=FALSE```:

````md
```{r, include=FALSE}

library(DT)

datatable(iris)

```
````

An example given by R | Science Loft:

````md
```{r, results="asis"}

library(htmltools)

list_to_output <- list(data1 = iris,
                       data2 = mtcars,
                       data3 = airquality)

for(i in names(list_to_output)){
  
  cat("\n") 
  cat("##", i, "\n") # Create second level headings with the names.
  
  print(
   tagList(
    datatable(list_to_output[[i]])
   )
  )
  
  cat("\n")
  
}

```
````


## 2. Example for creating console output:

We need to wrap the code output in <code></code> and  tags for rendering properly. Without these the console output will render like normal text. This is especially helpful if you are trying to print named vectors where the name and the value will be aligned neatly
````
```{r, results="asis"}

for(i in paste("Output", 1:3)){
  
  cat('\n') 
  cat("###", i, "\n")
  
  
  vec <- 1:10
  
  pre(# can also use tags manually: <pre><code>BlaBlaBla</code></pre>
    code(
      print(vec)   
    )
  )
  
  
}
```
````

## 3.Example for creating plot output

````
```{r,fig.width=3, fig.height=3, results="asis"}

for(i in paste("Plot", 1:3)){
  
  cat('\n') 
  cat("###", i, "\n")

print(
  ggplot(iris) +
    geom_point(aes(x=Sepal.Length, y=Petal.Length))
)

}

````
---
