---
title: 'Tidyverse Case Study: Anscombe’s quartet'
author: ''
date: '2017-12-08'
slug: tidy-anscombe
draft: true
categories:
  - rstats
  - rbloggers
tags: []
---

Anscombe's quartet is a really cool dataset that is used to illustrate
the importance of data visualisation. It comes built into R, and reading the helpfile it states that it is:

> Four x-y datasets which have the same traditional statistical properties (mean, variance, correlation, regression line, etc.), yet are quite different.

Different, how? A visualisation of each group reveals this best:

These groups are each very similar.

As I mentioned earlier, R actually has the anscombe dataset built in, which is super cool. Even cooler, 
the helpfile event provides code exploreing and visualising anscombe's quartet:


```r
require(stats); require(graphics)
summary(anscombe)
```

```
##        x1             x2             x3             x4    
##  Min.   : 4.0   Min.   : 4.0   Min.   : 4.0   Min.   : 8  
##  1st Qu.: 6.5   1st Qu.: 6.5   1st Qu.: 6.5   1st Qu.: 8  
##  Median : 9.0   Median : 9.0   Median : 9.0   Median : 8  
##  Mean   : 9.0   Mean   : 9.0   Mean   : 9.0   Mean   : 9  
##  3rd Qu.:11.5   3rd Qu.:11.5   3rd Qu.:11.5   3rd Qu.: 8  
##  Max.   :14.0   Max.   :14.0   Max.   :14.0   Max.   :19  
##        y1               y2              y3              y4        
##  Min.   : 4.260   Min.   :3.100   Min.   : 5.39   Min.   : 5.250  
##  1st Qu.: 6.315   1st Qu.:6.695   1st Qu.: 6.25   1st Qu.: 6.170  
##  Median : 7.580   Median :8.140   Median : 7.11   Median : 7.040  
##  Mean   : 7.501   Mean   :7.501   Mean   : 7.50   Mean   : 7.501  
##  3rd Qu.: 8.570   3rd Qu.:8.950   3rd Qu.: 7.98   3rd Qu.: 8.190  
##  Max.   :10.840   Max.   :9.260   Max.   :12.74   Max.   :12.500
```

```r
##-- now some "magic" to do the 4 regressions in a loop:
ff <- y ~ x
mods <- setNames(as.list(1:4), paste0("lm", 1:4))
for(i in 1:4) {
  ff[2:3] <- lapply(paste0(c("y","x"), i), as.name)
  mods[[i]] <- lmi <- lm(ff, data = anscombe)
  print(anova(lmi))
}
```

```
## Analysis of Variance Table
## 
## Response: y1
##           Df Sum Sq Mean Sq F value  Pr(>F)   
## x1         1 27.510 27.5100   17.99 0.00217 **
## Residuals  9 13.763  1.5292                   
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## Analysis of Variance Table
## 
## Response: y2
##           Df Sum Sq Mean Sq F value   Pr(>F)   
## x2         1 27.500 27.5000  17.966 0.002179 **
## Residuals  9 13.776  1.5307                    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## Analysis of Variance Table
## 
## Response: y3
##           Df Sum Sq Mean Sq F value   Pr(>F)   
## x3         1 27.470 27.4700  17.972 0.002176 **
## Residuals  9 13.756  1.5285                    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## Analysis of Variance Table
## 
## Response: y4
##           Df Sum Sq Mean Sq F value   Pr(>F)   
## x4         1 27.490 27.4900  18.003 0.002165 **
## Residuals  9 13.742  1.5269                    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

```r
## See how close they are (numerically!)
sapply(mods, coef)
```

```
##                   lm1      lm2       lm3       lm4
## (Intercept) 3.0000909 3.000909 3.0024545 3.0017273
## x1          0.5000909 0.500000 0.4997273 0.4999091
```

```r
lapply(mods, function(fm) coef(summary(fm)))
```

```
## $lm1
##              Estimate Std. Error  t value    Pr(>|t|)
## (Intercept) 3.0000909  1.1247468 2.667348 0.025734051
## x1          0.5000909  0.1179055 4.241455 0.002169629
## 
## $lm2
##             Estimate Std. Error  t value    Pr(>|t|)
## (Intercept) 3.000909  1.1253024 2.666758 0.025758941
## x2          0.500000  0.1179637 4.238590 0.002178816
## 
## $lm3
##              Estimate Std. Error  t value    Pr(>|t|)
## (Intercept) 3.0024545  1.1244812 2.670080 0.025619109
## x3          0.4997273  0.1178777 4.239372 0.002176305
## 
## $lm4
##              Estimate Std. Error  t value    Pr(>|t|)
## (Intercept) 3.0017273  1.1239211 2.670763 0.025590425
## x4          0.4999091  0.1178189 4.243028 0.002164602
```

```r
## Now, do what you should have done in the first place: PLOTS
op <- par(mfrow = c(2, 2), mar = 0.1+c(4,4,1,1), oma =  c(0, 0, 2, 0))
for(i in 1:4) {
  ff[2:3] <- lapply(paste0(c("y","x"), i), as.name)
  plot(ff, data = anscombe, col = "red", pch = 21, bg = "orange", cex = 1.2,
       xlim = c(3, 19), ylim = c(3, 13))
  abline(mods[[i]], col = "blue")
}
mtext("Anscombe's 4 Regression data sets", outer = TRUE, cex = 1.5)
```

<img src="/post/2017-12-08-tidy-anscombe_files/figure-html/anscombe-base-1.png" width="672" />

```r
par(op)
```

This is great, and certainly succinct, but I think it would be interesting to
compare this process to how you would explore this using the tidyverse tools.

There are a few key parts to this analysis:

1. Tidy up the data
1. Explore the summary statistics of each group
1. Fit a model to each group
1. Make the plots in ggplot2.

## Tidy up the data

Before we tidy up the data, it is good to think about what format we want the data in.

Currently, the data is in this format:


```r
head(anscombe)
```

```
##   x1 x2 x3 x4   y1   y2    y3   y4
## 1 10 10 10  8 8.04 9.14  7.46 6.58
## 2  8  8  8  8 6.95 8.14  6.77 5.76
## 3 13 13 13  8 7.58 8.74 12.74 7.71
## 4  9  9  9  8 8.81 8.77  7.11 8.84
## 5 11 11 11  8 8.33 9.26  7.81 8.47
## 6 14 14 14  8 9.96 8.10  8.84 7.04
```

What we want is a format where we have:

|| variable | group | value ||
||  x       | 1     | 10    ||

We can get the data into two columns "variable" and "value" using the `tidyr`
package.

Here, we use the `gather` function, and we tell it that we want two


```r
library(tidyr)
library(tibble)

tidy_anscombe <- anscombe %>% 
  gather(key = "variable",
         value = "value") %>%
  as_tibble() 

tidy_anscombe
```

```
## # A tibble: 88 x 2
##    variable value
##    <chr>    <dbl>
##  1 x1          10
##  2 x1           8
##  3 x1          13
##  4 x1           9
##  5 x1          11
##  6 x1          14
##  7 x1           6
##  8 x1           4
##  9 x1          12
## 10 x1           7
## # ... with 78 more rows
```

## Explore the summary statistics of each group


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
tidy_anscombe %>%
  group_by(variable) %>%
  summarise_all(funs(min,max,median,mean,sd,var))
```

```
## # A tibble: 8 x 7
##   variable   min   max median  mean    sd   var
##   <chr>    <dbl> <dbl>  <dbl> <dbl> <dbl> <dbl>
## 1 x1        4    14      9     9     3.32 11   
## 2 x2        4    14      9     9     3.32 11   
## 3 x3        4    14      9     9     3.32 11   
## 4 x4        8    19      8     9     3.32 11   
## 5 y1        4.26 10.8    7.58  7.50  2.03  4.13
## 6 y2        3.1   9.26   8.14  7.50  2.03  4.13
## 7 y3        5.39 12.7    7.11  7.5   2.03  4.12
## 8 y4        5.25 12.5    7.04  7.50  2.03  4.12
```

## Fit a model to each group



## Make the plots in ggplot (which you should have done in the first place).


```r
library(ggplot2)
ggplot(tidy_anscombe,
       aes(x = variable,
           y = value)) +
  geom_point() + 
  facet_wrap(~variable)
```

<img src="/post/2017-12-08-tidy-anscombe_files/figure-html/anscombe-plot-1.png" width="672" />

# End++

So there you have it, some ways to tidy up

The base code is relatively straightforward, but, especially if you are just
starting out coding, I think that the tidyverse provides some easier footholds
that will also do the user justice later on.

# Further reading

Check out [Rasmus Bååth's post on fitting a Bayesian spike-slab model to 
accurately fit to anscombe's quartet](). Cool stuff!

Note: Can I create "Tierney's Trio", or something? Some sort of canonical data
set that contains missing values?
