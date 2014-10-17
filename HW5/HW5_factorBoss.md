# HW5: Be the boss of my factors
Santina  
Thursday, October 16, 2014  

# Introduction 
In this assignment, we are going to see how to be in [control of factors](http://stat545-ubc.github.io/block014_factors.html) in a data set, using the provided [Gapminder excerpt](http://www.stat.ubc.ca/~jenny/notOcto/STAT545A/examples/gapminder/data/gapminderDataFiveYear.txt) as an example, and experiment with [reading and writing files](https://github.com/STAT545-UBC/STAT545-UBC.github.io/blob/master/cm011_files-out-in-script.r). 

It will follow roughly (with perhaps some experimentations on the side) the assignment guideline. This is the last assignment of STAT545! I can't wait for the second half of the course, which will be even more exciting! 

As always, let's get the gapminder data. Since it's from an URL, I am going to try to download it by reading it into R and saving it as a text file in my current directory. 

## Notes to myself 
Just some notes to remind me what I can use 
- writing to and reading from files
  * use `write.table()` and `read.delim()` or `read.table()`, make sure to experiment with different arguments 
- writing and reading R objects
  * RDS format: `saveRDS()`, `readRDS()`
  * plain text format (often preferable):`dput()` and `dget()` 

## same old stuff: packages and the gapminder data
load some packages 

```r
library(ggplot2) # for making plots
library(ggthemes)# for customizaing ggplot graphs 
library(scales)  # for graphs scale
library(plyr)    # for easy computation with data frames
library(dplyr)   # do this after loading plyr
library(knitr)   # for rendering pretty tables
```

Loading our gapminder data 

```r
gpURL <- "http://www.stat.ubc.ca/~jenny/notOcto/STAT545A/examples/gapminder/data/gapminderDataFiveYear.txt"
dataExcerpt <- read.delim(file = gpURL) 
str(dataExcerpt)
```

```
## 'data.frame':	1704 obs. of  6 variables:
##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
##  $ pop      : num  8425333 9240934 10267083 11537966 13079460 ...
##  $ continent: Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
##  $ gdpPercap: num  779 821 853 836 740 ...
```

```r
#saving it as "gapminder_excerpt.txt" in my current homework directory
write.table(dataExcerpt, "gapminder_excerpt.txt") 
```

Let's try to see how well we can read from the text file we just created. 

```r
gapExcerpt <-  read.delim(file = "gapminder_excerpt.txt")
str(gapExcerpt)
```

```
## 'data.frame':	1704 obs. of  1 variable:
##  $ country.year.pop.continent.lifeExp.gdpPercap: Factor w/ 1704 levels "1 Afghanistan 1952 8425333 Asia 28.801 779.4453145",..: 1 817 928 1039 1150 1261 1372 1483 1594 2 ...
```
Hum.... a little massier than the result of `str(dataExcerpt)`. Let's come back to this later. 

# Drop Oceania 
Since there are only two countries in this "continent" category, I will remove it and use `droplevels()` to ensure the level is completely clean of Oceania.


```r
# take everything except those whose continent is Oceania 
dataExcerpt2 <- dataExcerpt %>%
  filter(continent != "Oceania") %>%
  droplevels
# check if Oceania is dropped 
dataExcerpt2$continent %>%
  table()
```

```
## dataExcerpt2$continent
##   Africa Americas     Asia   Europe 
##      624      300      396      360
```

```r
# versus 
dataExcerpt$continent %>%
  table()
```

```
## dataExcerpt$continent
##   Africa Americas     Asia   Europe  Oceania 
##      624      300      396      360       24
```

Yes, so Oceania is no longer included in our data set `dataExcerpt2`. The number of rows in dataExcerpt is 1704 versus in the Oceania-dropped dataExcerpt is 1680. 


# life expectancy 

Let's look at the slopes of the life expectancy over years for each country. Using `~country + continent` we basically do this for every single country while retaining their continent identity (let me know if I can describe this better). 


```r
j_coefs <- ddply(dataExcerpt, ~ country + continent, function(dat, offset = 1952) {
  the_fit <- lm(lifeExp ~ I(year - offset), dat)
  setNames(coef(the_fit), c("intercept", "slope"))
}) #this chunk was copied from the homework outline  

head(j_coefs) %>% kable()
```



|country     |continent | intercept|  slope|
|:-----------|:---------|---------:|------:|
|Afghanistan |Asia      |     29.91| 0.2753|
|Albania     |Europe    |     59.23| 0.3347|
|Algeria     |Africa    |     43.38| 0.5693|
|Angola      |Africa    |     32.13| 0.2093|
|Argentina   |Americas  |     62.69| 0.2317|
|Australia   |Oceania   |     68.40| 0.2277|

Upon closer examination (with inline R code which you can't see unless you go to view raw), there are 4 columns, 142 rows. There are 142 unique countries and 5 unique continents.  

# Order of data vs order of factor levels 

