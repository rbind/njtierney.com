---
title: Explore longitudinal data with brolgar
author: ''
date: '2019-07-25'
draft: true
slug: introducing-brolgar
categories:
  - ggplot2
  - rstats
  - Statistics
  - data visualisation
  - longitudinal data
  - panel data
tags:
  - rstats
---

Looking at longitudinal data often yields something like the following:

<img src="/post/2019-07-25-introducing-brolgar_files/figure-html/show-spaghetti-1.png" width="672" />


These can be frustrating to deal with, as it is not clear how to see the right features in the data.

I've spent a fair bit of time this year with [Di Cook](http://dicook.org/) thinking about some ways to improve how we look at and explore longitudinal data. It is a hard problem, and I'm certainly not done yet, but we created the `brolgar` package to help make it easier to explore and visualise longitudinal data. 

Why the name `brolgar`? It is an acronym, standing for:

* **br**owse 
* **o**ver
* **l**ongitudinal 
* **d**ata 
* **g**raphically and 
* **a**nalytically in 
* **R**

It is so named after the "brolga", [a beautiful, gregarious native Australian crane](https://en.wikipedia.org/wiki/Brolga):

<p><a href="https://commons.wikimedia.org/wiki/File:Brolgas_Healesville.jpg#/media/File:Brolgas_Healesville.jpg"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/32/Brolgas_Healesville.jpg/1200px-Brolgas_Healesville.jpg" alt="Brolgas Healesville.jpg"></a><br>By Felix Andrews (<a href="//commons.wikimedia.org/wiki/User:Floybix" title="User:Floybix">Floybix</a>) - <span class="int-own-work" lang="en">Own work</span>, <a href="http://creativecommons.org/licenses/by-sa/3.0/" title="Creative Commons Attribution-Share Alike 3.0">CC BY-SA 3.0</a>, <a href="https://commons.wikimedia.org/w/index.php?curid=1116364">Link</a></p>


I'll direct you to the [official brolgar website](http://brolgar.njtierney.com/) for more details on the package, but here I will showcase three ideas that I think make workflow significantly more efficient:

* Idea 1: Longitudinal data is a time series
* Idea 2: Explore by taking samples
* Idea 3: Finding interesting features

# Idea 1: Longitudinal data is a time series

To efficiently look at your longitudinal data, we assume it **is a time series**, with irregular time periods between measurements. This might seem strange, (that's OK!), but there are **two important things** to remember:

1. The **key** variable in your data is the **identifier** of your individuals or series.
2. The **index** variable is the **time** component of your data.

Together, the **index** and **key** uniquely identify an observation.

The term `key` is used a lot in brolgar, so it is an important idea to internalise:

> **The key is the identifier of your individuals or series**


So in the `heights` data, we have the following setup:


```r
heights <- as_tsibble(x = heights,
                            key = countries,
                            index = year,
                            regular = FALSE)
```


```r
heights
```

```
## # A tsibble: 1,499 x 4 [!]
## # Key:       country [153]
##    country      year height_cm continent
##    <chr>       <dbl>     <dbl> <chr>    
##  1 Afghanistan  1870      168. Asia     
##  2 Afghanistan  1880      166. Asia     
##  3 Afghanistan  1930      167. Asia     
##  4 Afghanistan  1990      167. Asia     
##  5 Afghanistan  2000      161. Asia     
##  6 Albania      1880      170. Europe   
##  7 Albania      1890      170. Europe   
##  8 Albania      1900      169. Europe   
##  9 Albania      2000      168. Europe   
## 10 Algeria      1910      169. Africa   
## # … with 1,489 more rows
```


Once we consider our longitudinal data a time series, we gain access to a set of amazing tools from the [`tidyverts` team](https://tidyverts.org/). What this means is that now that we know what your **time index** is, and what represents **each series with a key**, we can use that information to make sure we respect the structure in the data. This means you can spend more time performing analysis, and using functions fluently, and less time remembering to tell R about the structure of your data. 

If you want to learn more about longitudinal data as a time series, you can [read more in the vignette, "Longitudinal Data Structures"](library/brolgar/html/longitudinal-data-structures.html)


# Idea 2: Explore by taking samples

To get a sense of what your data is, you sometimes just need to look at a small sample. We've got some really neat tools for that.

## Sample with `sample_n_keys()` 

In `dplyr`, you can use `sample_n()` to sample `n` observations. Similarly, with `brolgar`, you can take a random sample of `n` keys using `sample_n_keys()`:


```r
library(brolgar)
library(ggplot2)
heights %>%
  sample_n_keys(size = 10) %>%
  ggplot(aes(x = year,
             y = height_cm,
             group = country)) + 
  geom_line()
```

<img src="/post/2019-07-25-introducing-brolgar_files/figure-html/plot-sample-n-keys-1.png" width="672" />

## Filtering observations with `filter_n_obs()`

You can combine `sample_n_keys()` with `filter_n_obs` to filter keys with many observations:


```r
heights %>%
  filter_n_obs(n_obs > 5) %>%
  sample_n_keys(size = 10) %>%
  ggplot(aes(x = year,
             y = height_cm,
             group = country)) + 
  geom_line()
```

<img src="/post/2019-07-25-introducing-brolgar_files/figure-html/plot-filter-sample-n-keys-1.png" width="672" />

Similar to `dplyr::sample_frac()`, there is also `sample_frac_keys()`, which samples a fraction of available keys.

Now, how do you break these into many plots?

## Sample with `facet_sample()`

`facet_sample()` allows you to specify the number of keys per facet, and the number of facets with `n_per_facet` and `n_facets`. It splits the data into 12 facets with 5 per facet by default:


```r
ggplot(heights,
       aes(x = year,
           y = height_cm,
           group = country)) +
  geom_line() +
  facet_sample()
```

<img src="/post/2019-07-25-introducing-brolgar_files/figure-html/facet-sample-1.png" width="672" />

But you can specify your own number:


```r
ggplot(heights,
       aes(x = year,
           y = height_cm,
           group = country)) +
  geom_line() +
  facet_sample(n_per_facet = 3,
               n_facets = 20)
```

<img src="/post/2019-07-25-introducing-brolgar_files/figure-html/facet-sample-3-by-20-1.png" width="672" />


# Idea 3: Finding interesting features

In order to find interesting features, you need to reduce each series, or `key` down to a one-row summary table, and then _join_ this back to the regular data. This workflow is now demonstrated.

## Find features with `features()`

You can extract `features` of longitudinal data using the `features` function, from `fablelite`. You can, for example, calculate the minimum of a given variable for each key by providing a named list like so:


```r
heights %>%
  features(height_cm, 
           list(min = min))
```

```
## # A tibble: 153 x 2
##    country       min
##    <chr>       <dbl>
##  1 Afghanistan  161.
##  2 Albania      168.
##  3 Algeria      166.
##  4 Angola       159.
##  5 Argentina    167.
##  6 Armenia      164.
##  7 Australia    170 
##  8 Austria      162.
##  9 Azerbaijan   170.
## 10 Bahrain      161.
## # … with 143 more rows
```

`brolgar` provides some sets of features, which start with `feat_`.

For example, the five number summary is `feat_five_num`:


```r
heights %>%
  features(height_cm, feat_five_num)
```

```
## # A tibble: 153 x 6
##    country       min   q25   med   q75   max
##    <chr>       <dbl> <dbl> <dbl> <dbl> <dbl>
##  1 Afghanistan  161.  164.  167.  168.  168.
##  2 Albania      168.  168.  170.  170.  170.
##  3 Algeria      166.  168.  169   170.  171.
##  4 Angola       159.  160.  167.  168.  169.
##  5 Argentina    167.  168.  168.  170.  174.
##  6 Armenia      164.  166.  169.  172.  172.
##  7 Australia    170   171.  172.  173.  178.
##  8 Austria      162.  164.  167.  169.  179.
##  9 Azerbaijan   170.  171.  172.  172.  172.
## 10 Bahrain      161.  161.  164.  164.  164 
## # … with 143 more rows
```

Or finding those whose values only increase or decrease with `feat_monotonic`


```r
heights %>%
  features(height_cm, feat_monotonic)
```

```
## # A tibble: 153 x 5
##    country     increase decrease unvary monotonic
##    <chr>       <lgl>    <lgl>    <lgl>  <lgl>    
##  1 Afghanistan FALSE    FALSE    FALSE  FALSE    
##  2 Albania     FALSE    TRUE     FALSE  TRUE     
##  3 Algeria     FALSE    FALSE    FALSE  FALSE    
##  4 Angola      FALSE    FALSE    FALSE  FALSE    
##  5 Argentina   FALSE    FALSE    FALSE  FALSE    
##  6 Armenia     FALSE    FALSE    FALSE  FALSE    
##  7 Australia   FALSE    FALSE    FALSE  FALSE    
##  8 Austria     FALSE    FALSE    FALSE  FALSE    
##  9 Azerbaijan  FALSE    FALSE    FALSE  FALSE    
## 10 Bahrain     TRUE     FALSE    FALSE  TRUE     
## # … with 143 more rows
```

You can read more about creating your own features in the vignette [creating features](http://brolgar.njtierney.com/articles/creating-features.html).

## Fit a linear model with `key_slope()`

`key_slope()` returns the intercept and slope estimate for each key, given some linear model formula. We can get the number of observations, and slope information for each key to identify those changing over time. 


```r
height_slope <- key_slope(heights, height_cm ~ year)

height_slope
```

```
## # A tibble: 153 x 3
##    country     .intercept .slope_year
##    <chr>            <dbl>       <dbl>
##  1 Afghanistan      217.      -0.0263
##  2 Albania          202.      -0.0170
##  3 Algeria          111.       0.0297
##  4 Angola            43.9      0.0648
##  5 Argentina        147.       0.0117
##  6 Armenia           87.9      0.0419
##  7 Australia         46.1      0.0665
##  8 Austria           38.2      0.0695
##  9 Azerbaijan       150.       0.0111
## 10 Bahrain         -157.       0.165 
## # … with 143 more rows
```

## Linking individuals back to the data

You can join these features back to the data with `left_join`, like so:


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following object is masked from 'package:tsibble':
## 
##     id
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
heights_mono <- heights %>%
  features(height_cm, feat_monotonic) %>%
  left_join(heights, by = "country")

heights_mono
```

```
## # A tibble: 1,499 x 8
##    country     increase decrease unvary monotonic  year height_cm continent
##    <chr>       <lgl>    <lgl>    <lgl>  <lgl>     <dbl>     <dbl> <chr>    
##  1 Afghanistan FALSE    FALSE    FALSE  FALSE      1870      168. Asia     
##  2 Afghanistan FALSE    FALSE    FALSE  FALSE      1880      166. Asia     
##  3 Afghanistan FALSE    FALSE    FALSE  FALSE      1930      167. Asia     
##  4 Afghanistan FALSE    FALSE    FALSE  FALSE      1990      167. Asia     
##  5 Afghanistan FALSE    FALSE    FALSE  FALSE      2000      161. Asia     
##  6 Albania     FALSE    TRUE     FALSE  TRUE       1880      170. Europe   
##  7 Albania     FALSE    TRUE     FALSE  TRUE       1890      170. Europe   
##  8 Albania     FALSE    TRUE     FALSE  TRUE       1900      169. Europe   
##  9 Albania     FALSE    TRUE     FALSE  TRUE       2000      168. Europe   
## 10 Algeria     FALSE    FALSE    FALSE  FALSE      1910      169. Africa   
## # … with 1,489 more rows
```

Or for the linear model data:


```r
heights_slope <- heights %>%
  key_slope(height_cm ~ year) %>%
  left_join(heights, by = "country")

heights_slope
```

```
## # A tibble: 1,499 x 6
##    country     .intercept .slope_year  year height_cm continent
##    <chr>            <dbl>       <dbl> <dbl>     <dbl> <chr>    
##  1 Afghanistan       217.     -0.0263  1870      168. Asia     
##  2 Afghanistan       217.     -0.0263  1880      166. Asia     
##  3 Afghanistan       217.     -0.0263  1930      167. Asia     
##  4 Afghanistan       217.     -0.0263  1990      167. Asia     
##  5 Afghanistan       217.     -0.0263  2000      161. Asia     
##  6 Albania           202.     -0.0170  1880      170. Europe   
##  7 Albania           202.     -0.0170  1890      170. Europe   
##  8 Albania           202.     -0.0170  1900      169. Europe   
##  9 Albania           202.     -0.0170  2000      168. Europe   
## 10 Algeria           111.      0.0297  1910      169. Africa   
## # … with 1,489 more rows
```

You can then plot them with the amazing [`gghighlight` package by Hiroaki Yutani](https://github.com/yutannihilation/gghighlight) to highlight interesting parts of a plot.

For example, one that highlights only those keys that are increasing:


```r
library(gghighlight)
ggplot(heights_mono,
       aes(x = year,
           y = height_cm,
             group = country)) +
  geom_line() + 
  gghighlight(increase)
```

```
## Warning: You set use_group_by = TRUE, but grouped calculation failed.
## Falling back to ungrouped filter operation...
```

```
## label_key: country
```

<img src="/post/2019-07-25-introducing-brolgar_files/figure-html/features-mono-1.png" width="672" />

Or those with a negative slope:


```r
ggplot(heights_slope,
       aes(x = year,
           y = height_cm,
           group = country)) +
  geom_line() + 
  gghighlight(.slope_year < 0)
```

```
## Warning: You set use_group_by = TRUE, but grouped calculation failed.
## Falling back to ungrouped filter operation...
```

```
## label_key: country
```

<img src="/post/2019-07-25-introducing-brolgar_files/figure-html/gghighlight-slope-1.png" width="672" />


## Facet along some feature:

You can even facet _along_ some feature, using `facet_strata(along = var)`. For example, we could facet our data along the slope. `facet_strata` requires a `tsibble` object, so we need to be careful with how we do our join.


```r
heights %>%
  key_slope(height_cm ~ year) %>%
  left_join(x = heights, 
            y = ., 
            by = "country") %>% 
ggplot(aes(x = year,
           y = height_cm,
           group = country)) +
  geom_line() + 
  facet_strata(n_strata = 12,
               along = .slope_year)
```

<img src="/post/2019-07-25-introducing-brolgar_files/figure-html/facet-strata-along-1.png" width="672" />

This shows us the spread of countries when we break slop up into 12 groups arranged in order from increasing to decreasing slope.

Under the hood, `facet_strata()` and `facet_sample()` are powered by [`sample_n_keys()`](http://brolgar.njtierney.com/reference/sample-n-frac-keys.html) and [`stratify_keys()`](http://brolgar.njtierney.com/reference/stratify_keys.html), I've linked to their documentation online in the text.

## Find keys near some summary statistic

If you want to take the slope information and only return those individuals closest to the five number summary of slope - say those closest to these values:


```r
summary(heights_slope$.slope_year)
```

```
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max.     NA's 
## -0.10246  0.02187  0.03696  0.04175  0.06176  0.32150        9
```

You can find those keys near the minimum, 1st quantile, median, mean, 3rd quantile maximum, using `keys_near()`, specifying the `country` and `.slope_year`.


```r
heights_near <- heights %>%
  key_slope(height_cm ~ year) %>%
  keys_near(key = country,
            var = .slope_year)

heights_near
```

```
## # A tibble: 6 x 5
## # Groups:   stat [5]
##   country    .slope_year stat  stat_value stat_diff
##   <chr>            <dbl> <chr>      <dbl>     <dbl>
## 1 Eritrea        -0.102  min      -0.102   0       
## 2 Tajikistan      0.0199 q25       0.0205  0.000632
## 3 Mali            0.0401 med       0.0403  0.000120
## 4 Spain           0.0404 med       0.0403  0.000120
## 5 Austria         0.0695 q_75      0.0690  0.000515
## 6 Burundi         0.321  max       0.321   0
```

This returns those keys closest to some statistics.

You could include it in a plot like so:


```r
heights_near %>%
  left_join(heights, by = "country") %>%
ggplot(aes(x = year,
           y = height_cm,
           colour = stat)) + 
  geom_line() 
```

<img src="/post/2019-07-25-introducing-brolgar_files/figure-html/keys-near-plot-1.png" width="672" />

You can read more about `keys_near()` at the [finding summary keys vignette](http://brolgar.njtierney.com/articles/find-summary-keys.html).

# Fin

There is more to come for `brolgar` - the API will be likely to change as I get feedback from the community, and as I think and learn more about exploring longitudinal data. You can see my current thoughts on what to include in `brolgar` in [the brolgar issues](https://github.com/njtierney/brolgar).

If you have any thoughts, comments, problems, or concerns, post a comment below or even better [file an issue](https://github.com/njtierney/brolgar/issues/new).

Happy data exploring!
