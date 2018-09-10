---
title: 'Tidyverse Case Study: Exploring XKCD comics'
author: ''
draft: true
date: '2017-11-09'
slug: tidyverse-xkcd
categories:
  - rstats
  - rbloggers
tags: []
---

I'm a huuuge fan of XCKD.

Randall Munroe has done some awesome stuff, including:

- Writing about a submarine, landing him a job in NASA
- asking people about their colour preferences.

XKCD is also a cool name - you need to say each of the letters, there's no real shortcut word for it.

Once again, reading Joe's post about Data packages, I came across the XKCD package - which is really really awesome.

It provides a whole bunch of info about the XKCD comics.

Randall now writes the comics full time, along with his other really awesome books - 

After going through the Billboard data, I am wondering if I can explore the rise of XKCD - perhaps we can track when the comics become regular?


```r
library(XKCDdata)
xkcd_numbers <- 1:400
# xkcd_numbers <- 500:1000
# xkcd_data <- purrr::map_df(xkcd_numbers, get_comic)

# View(xkcd_data)
# 
# xkcd_data %>%
#   select(year,month,day,num) %>%
#   mutate(date = ISOdate(year,month,day)) %>%
#   padr::pad() %>% 
#   mutate(tidyr::fill(num))
#   ggplot(aes(x = date,
#              y = num)) + 
#   geom_point()
```

# Future Work

I'm also a big fan of the webcomics, Sunday Morning Breakfast Cereal, Perry Bible Fellowship, and Questionable Content. Perhaps these can make some nice toy R packages in the future.
