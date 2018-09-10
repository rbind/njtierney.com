---
title: 'some CRAN Gotchas '
author: ''
date: '2017-08-09'
slug: some-cran-gotchas
draft: true
categories:
  - rstats
  - Blag
tags: []
---

Recently I submitted `visdat` to CRAN, and have just submitted `naniar` to CRAN as well (fingers crossed!).

There were a couple of small things that I changed in order to get everything on CRAN OK, and I thought it might be helfpul to list them here, both for me, and for others, and also for others to comment on.

Error message:

- This refers to a blank url in a vignette - which was picked up by Jim Hester, and looked like this [text]() - see here, no link in the URL, hence blank URL. I started thinking really deep about this, about how pkgdown works, and everything. Nope. Much simpler!



Package Version too large.

I use pkgdown to make my website docs for visdat and naniar. It's really nifty, and looks super pro with minimal work from me. The files are a bit weird, so I thought that this error was to do with that

https://stackoverflow.com/questions/36701433/r-package-building-with-devtoolsbuild-win-version-contains-large-components


Nope, this is **literally** to do with the package version - once I changed it to 0.1.0 everything worked fine. 

Spelling mistakes.

README figures

I got this strange error in win-builder,

I decided to move the rmd figures inside the `man/` folder - which I copied off of what I had seen i the [ggplot2 README]. That's a nice thing about github - want to know how 

surprisingly naniar got onto CRAN quite quickly (I'll write more about it soon), so what do you do next?

- Add a cran status badge with `devtools::use_cran_badge()`, and copy and paste the text provided.
- Add a cran downloads badge by going to https://cranlogs.r-pkg.org/ and entering in your pkg name.


In general, I have held off from submitting to CRAN because I was scared of getting something wrong. I wrote my first R package in 2013, thanks largely to Hilary Parker's amazing blog post and then following through with Hadley's R Packages book. But I have held off on putting things on CRAN for a long time.

In some ways, this has made things good, for example, `visdat` changed name 3 times (footprintr -> vizdat -> visdat), and `naniar` actually changed name 4 times (ggmissing -> naniar -> narnia -> naniar). Don't ask. I've also spent a large amount of time making sure that the naming of my functions has been good, and made sense. This meant changing names from

`summary_missing_variables` and `...

to `miss_var_summary`, and then establishing a naming scheme where the missing summaries and friends start with `miss_` - this makes it easier for them to be tabbed through. Similarly, there is `gg_miss_var`, which was initially `gg_missing_variables`. Iteration here took time.

But I think that I'll back myself in the future more now. Having github there as a way to test out ideas, and rapidly change things has been really nice, and in some ways has saved me time.

But I wonder if maybe had I put these packages out onto CRAN sooner then maybe I would have gotten more feedback from a wider audience?

http://r-pkgs.had.co.nz/release.html
