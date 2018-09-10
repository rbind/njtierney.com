---
title: analysing my spotify with R
author: ''
draft: yes
date: '2017-07-13'
slug: spotify
categories:
  - rstats
tags: []
---

- maybe also look into this R package: https://github.com/ewenme/geniusr

Inspired by Kara Woo's awesome blog post on analysing her own spotify data, I linked my spotify account with lastfm - as I'm pretty sure that this is the only way to track my spotify listening habits (right? Let me know in the comments if there is another thing I should know about)


```r
library(tidyverse)
```

```
## ── Attaching packages ────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ ggplot2 3.0.0     ✔ purrr   0.2.5
## ✔ tibble  1.4.2     ✔ dplyr   0.7.6
## ✔ tidyr   0.8.1     ✔ stringr 1.3.1
## ✔ readr   1.1.1     ✔ forcats 0.3.0
```

```
## ── Conflicts ───────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

```r
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

```r
library(knitr)

## Load data
plays <- read_csv("~/Downloads/njtierney (1).csv",
                  col_names = c("artist", "album", "track", "date"),
                  col_types = cols(date = col_date(format = "%d %b %Y %H:%M")))

## Set ggplot2 theme
theme_set(theme_minimal() +
          theme(axis.text = element_text(size = 14),
                axis.title = element_text(size = 16),
                legend.text = element_text(size = 14)))

## Color palette
msievse <- c("#547a9e" ,"#d5bfa8", "#aec2cb", "#93775f", "#a0a17f", "#73715c",
             "#dfddd1", "#7598ac", "#4f3e36", "#6f7269")
```



```r
all_months <- plays %>% expand(year(date), month(date)) %>% 
  filter(`year(date)` >= 2009)

plays %>% 
  group_by(year(date), month(date)) %>% 
  right_join(all_months) %>%
  tally() %>% 
  mutate(month = as.Date(paste(`year(date)`, `month(date)`, "01",
                               sep = "-"))) %>% 
  ggplot(aes(x = month, y = n)) +
  geom_line() +
  labs(x = "Month", y = "Scrobbles")
```

```
## Joining, by = c("year(date)", "month(date)")
```

<img src="/post/2017-07-13-analysing-my-spotify-with-r_files/figure-html/unnamed-chunk-2-1.png" width="672" />


```r
top_10_artists <- plays %>%
  count(artist, sort = TRUE) %>%
  top_n(n = 10, wt = n)

ggplot(top_10_artists, aes(x = reorder(artist, n), y = n)) +
  geom_bar(stat = "identity") +
  geom_text(aes(label = n), position = position_dodge(0.9), hjust = 1.2, 
            color = "white", size = 4) +
  labs(y = "Scrobbles", x = "") +
  coord_flip()
```

<img src="/post/2017-07-13-analysing-my-spotify-with-r_files/figure-html/unnamed-chunk-3-1.png" width="672" />


```r
## Remove data with messed up year
plays_clean <- filter(plays, year(date) >= 2009)

plays_clean %>% 
  group_by(year(date), artist) %>% 
  tally() %>% 
  top_n(5, wt = n) %>% 
  rename(year = `year(date)`) %>%
  arrange(-year,-n)
```

```
## # A tibble: 10 x 3
## # Groups:   year [2]
##     year artist                  n
##    <dbl> <chr>               <int>
##  1  2017 Daft Punk             227
##  2  2017 Christophe Beck       221
##  3  2017 The 1975              220
##  4  2017 Bombay Bicycle Club   185
##  5  2017 The xx                165
##  6  2016 Daft Punk             263
##  7  2016 Powderfinger          169
##  8  2016 Gordi                 150
##  9  2016 Bon Iver              139
## 10  2016 The Avalanches        124
```


```r
plays %>%
  add_count(track) %>%
  arrange(-n) %>%
  distinct()
```

```
## # A tibble: 11,468 x 5
##    artist        album                        track     date           n
##    <chr>         <chr>                        <chr>     <date>     <int>
##  1 Maggie Rogers Dog Years                    Dog Years 2017-04-26    41
##  2 Maggie Rogers Now That The Light Is Fading Dog Years 2017-04-06    41
##  3 Maggie Rogers Now That The Light Is Fading Dog Years 2017-03-23    41
##  4 Maggie Rogers Now That The Light Is Fading Dog Years 2017-03-05    41
##  5 Maggie Rogers Now That The Light Is Fading Dog Years 2017-02-27    41
##  6 Maggie Rogers Dog Years                    Dog Years 2017-02-13    41
##  7 Maggie Rogers Dog Years                    Dog Years 2017-02-12    41
##  8 Maggie Rogers Dog Years                    Dog Years 2017-01-31    41
##  9 Maggie Rogers Dog Years                    Dog Years 2017-01-27    41
## 10 Maggie Rogers Dog Years                    Dog Years 2017-01-19    41
## # ... with 11,458 more rows
```

