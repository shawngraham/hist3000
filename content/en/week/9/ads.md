---
title: "Archaeology Data Service"
description: "Data Services for Archaeologists"
date: 2020-01-28T00:10:51+09:00
draft: false
weight: -8
author: "Shawn Graham"
---

### Introduction
In this exercise, we're working with data deposited in ADS.  The code at the bottom is an adaptation of Rachel Optiz's notebook, ['Exploring published data from the ADS'](https://github.com/ropitz/spatialarchaeology/blob/master/ADSDataExercise.ipynb). Dr. Opitz's work is written in Python, if you want to take a look at how she does it.

Any data published as CSV files with the ADS can be pulled into R for exploration, asking your own questions with it, and generally doing research.
For example, play with the data from Ewan Campbell's 2007 deposit "Imported Material in Atlantic Britain and Ireland c.AD400-800", found at http://archaeologydataservice.ac.uk/archives/view/campbell_cba_2007/downloads.cfm

Open our R Studio in the cloud (right-click [this link](http://mybinder.org/v2/gh/o-date/r-conda/master?urlpath=rstudio)). Create a new script, and then paste the below into it. Work through it one line at a time, taking note of the comments as you go. When you get to the end, explore the ADS, find another dataset, and try modifiying the code to explore the new data. What kind of work do you imagine must be necessary to get data into shape for the ADS?

```R
# set up required libraries
library(curl)
library(stringr)
library(dplyr)

#start by reading in Campbell's table of glass artefacts and printing it out

# we load it up from the web, and make the ID number the row numbrer
glass <- read.csv(curl("http://ads.ahds.ac.uk/catalogue/adsdata/arch-788-1/dissemination/csv/imports_database/Glass.csv"), header = TRUE, row.names="ID")

# view the data
View(glass)

# Get all the finds from the table where the "Form" is "Cone Beaker"
# We create a new object 'ConeBeakers', which gets 'glass' filtered on the 'Form' column for the phrase 'Cone beaker'
# '<-' passes the results from the operations on its right to the object on its left
# '%>% pipes the object on its left to the commands on its right
ConeBeakers <- glass %>%
  filter(str_detect(Form, "Cone beaker"))

# Want a different vessel form? Just copy those two lines above,
# paste them after this comment block, and change "Cone beaker" to something
# else you see in the table 'glass'.

#----
# Now you can start to explore. Which sites have most of the Cone beakers?
# Make a bar plot of how often each site appears in the Cone Beaker table.

# first we use the 'table' command to count up the number of times each site appears
siteCounts <- table(ConeBeakers$Site)

# then we sort the list
siteCounts <- sort(siteCounts, decreasing=TRUE)

# barplot(data, title, label, show labels, make 'em really small)
barplot(siteCounts, main="Sites", xlab="Site", las=2, cex.names=.5 )

# Now you do it for a different vessel form.

#---
# Whithorn has a lot of Cone Beakers. I wonder what else is there?
# We can follow the same pattern as when we searched for cone beakers,
# but searching the 'site' column instead from our original 'glass' data.

Whithorn <- glass %>%
  filter(str_detect(Site, "WHITHORN"))

View(Whithorn)

# and now we make a barplot for the other kinds of forms at that site

whithornCounts <- table(Whithorn$Form)
whithornCounts <- sort(whithornCounts, decreasing=TRUE)
barplot(whithornCounts, main="Fomrs", xlab="Form", las=2, cex.names=.5)

# you get the idea. In week 5, I showed you some basic stats.
# Try doing that on this data.

crosstab <- xtabs(~Form+Group, Whithorn)
crosstab
barplot(crosstab, las=2)

#----
# Explore ADS: can you find an interesting dataset and get it loaded into R?
#----
```
