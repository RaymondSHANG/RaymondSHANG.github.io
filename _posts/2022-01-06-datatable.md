---
layout: post
title: "DataTable"
subtitle: "withExamples"
date: 2022-01-06 22:41:32
header-style: text
catalog: true
author: "Yuan"
tags: [data.table,R,SQL,examples]
---
>sweet!

# Data.table
I've worked on large dataset for a long time. Previously, I mainly used dataframes to deal with various data manipulations and wranglings. It is OK and I got used to that. However, I never felt so sweet before using data.table. It's much much faster. If you are familiar with SQL queries, data.table would be your first choice when dealing with large dataset in data science. You will like it!

Acutually, just with a few practice with data.table, I have much better understanding why the community developed tibble after dataframe, and then with data.table. Also, I used 'rownames' a lot. Maybe I should find some alternative to in my later codes.

Below are some example codes I summarized based on [Deepanshu Bhalla]()'s blog,[Colby RUG,Manny Gimond](https://mgimond.github.io/rug_2019_12/Index.html) and my own experience:

```R
rm(list=ls())
library(data.table)

#Read/Write
mydata = fread("https://github.com/arunsrinivasan/satrdays-workshop/raw/master/flights_2014.csv")
fwrite(mydata,file = "mydata_test.csv")

#describe
dim(mydata)
head(mydata)
colnames(mydata)
names(mydata)

#select/update some columns
##return a vector
mydata[,year] #Return a vector
mydata$year 
mydata$'year'
mydata[['year']]
mydata[[1]]
a = 1
mydata[[a]]

##return a DT
mydata[,1,with=F]
mydata[,.(year),]
mydata[,'year'] #Return a DT
mydata[,c('year','flight')]
mydata[,.(year,flight)]
#mydata[,.("month","day")] #Wrong way!

colselect=c('year','flight')
mydata[,c('year','flight'),with=F]
mydata[,colselect,with=F]#with=F is required unless you have a col with name:'colselect'
###mydata[,flight]
mydata[,c(1,2)]
colselect2=c(1,2)
mydata[,colselect2,with=F]
mydata[,which(unlist(lapply(mydata, function(onecol)!all(is.na(onecol))))),with=F]#Remove columns with all NAs
mydata[apply(mydata, 1, function(r) !all(is.na(r))),] #Remove rows with all NAs

selectedvalues = c(1,4,12,2014)
mydata[apply(mydata, 1, function(r) any(r %in% selectedvalues)),] #select rows contain any of the selectedvalues
mydata[,which(unlist(lapply(mydata, function(onecol) any(onecol %in% selectedvalues)))),with=F] #select columns contain any of the selectedvalues

#general select, SQL style
## DT[i, j, by]
##   R:                 i                 j        by
## SQL:  where | order by   select | update  group by
mydata[(year ==2014)|(year==2015) &(month==6),.N,by=.(origin,dest)]
mydata[(year ==2014)|(year==2015) &(month==6),.(day)]

#Add/update columns to the original DT
mydata[,newcol:=year*month,]
mydata[,'newcol':=year*month,]
mydata[, c("newcol1","newcol2"):=list(year*month, year/month)]

#add a row/multiple rows to the original DT
newDT <- mydata[1,,]
mydata <- rbindlist(list(mydata,  newDT))


#DT[][][]...SQL style query
mydata[,flag:=ifelse(month>6,1,0),][,.N,by=.(flag)]

#unique DT
unique(mydata)

#.SD and .SDcols
#If you need to calculate summary statistics for 
#a larger list of variables, you can use .SD and .SDcols operators. 
#The .SD operator implies 'Subset of Data'. 


#merge
#merge(dt1, dt2, by="A") #inter join
#merge(dt1, dt2, by="A", all.x = TRUE) #left join
#merge(dt1, dt2, by="A", all.y = TRUE) #right join
#merge(dt1, dt2, all=TRUE) #full join

# %like%,%between%, %in%
DT = data.table(Name=c("dep_time","dep_delay","arrival"), ID=c(2,3,4))
DT[Name %like% "dep"] 
DT[Name %in% c("dep_time","dep_delay")] 
DT[ID %between% c(3,9)]

#Cumulative SUM by GROUP
mydata[, cum:=cumsum(distance), by=carrier][,.(cum)]

#Convert a data.table to data.frame
setDF(mydata)

#Other interesting operations
mydata[, .N, by = month] [order(-N)]
mydata[, .(mean_arr_delay = mean(arr_delay, na.rm = TRUE)), by = month][order(-mean_arr_delay)][1:3]
mydata[, lapply(.SD, mean, na.rm = TRUE), .SDcols = c("arr_delay", "dep_delay"), by = origin][(arr_delay + dep_delay) > 20]
mydata[carrier == "DL",
       lapply(.SD, mean, na.rm = TRUE),
       by = .(origin, dest),
       .SDcols = c("arr_delay", "dep_delay")]
mydata[, .SD[1], .SDcols="air_time", by=origin][air_time > 300, sum(air_time)]

#Reference
#Deepanshu Bhalla
#https://www.listendata.com/2016/10/r-data-table.html

```


---
