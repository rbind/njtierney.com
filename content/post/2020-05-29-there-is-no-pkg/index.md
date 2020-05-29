---
title: 'Error in loadNamespace(Name) : There is No Package Called ''here'''
author: Nicholas Tierney
date: '2020-05-29'
slug: there-is-no-pkg
categories:
- rstats
- error
- loadNamespace
tags:
- rstats
- error
- loadNamespace
output: hugodown::hugo_document
rmd_hash: 9ce056cc92de0753

---




This error (or a variant of it) is quite common when using R:

```
Error in loadNamespace(name) : there is no package called 'here'
```

Another variant is:

```
Error in library(here) : there is no package called 'here'
```

Let's list out some ways that you can address this issue.

# Install the package

Install the package that is claimed not to be there. That is, for this error:

```
Error in loadNamespace(name) : there is no package called 'PKG'
```

You install the `PKG` package (use your package name intead of `PKG`):

```r
install.packages("PKG")
```

## Why does that work?

The error:

```
... there is no package called 'PKG'
```

is given because R is looking for a package to use, and it cannot find that package. 
Installing your package means that R can find it, and load it so you can use it!


## Some common other approaches

If the previous step doesn't work, check the following:

**Is the package perhaps misspelt?**

I have, more often than I care to admit, had a spelling mistake that caused me to go on a rabbit hole.

**Does the package exist on CRAN? Type "PKG CRAN rstats" into a search engine**

Perhaps you might find the right spelling, in which case, install the package with the right spelling using `install.packages`.

Perhaps you might find that it is on github (or bitbucket or gitlab), not on CRAN.

Let's take a github example. You need to install an R package from github with a different command.

Let's say we want to install the ["treezy" package from github](https://github.com/njtierney/treezy#installation). You scroll down and find the instructions here:

<img src="https://imgs.njtierney.com/treezy-install.png" width="60%" style="display: block; margin: auto;" />

So you will need to do two things:

1. Install `remotes` from CRAN (`install.packages("remotes"`)
2. Run `remotes::install_github()`

Similarly there are packages for R packages that you might find on other repositories such as `gitlab` (`install_gitlab`) or `bitbucket` (`install_bitbucket`).

# Why write this blog post?

I teach an [introduction to data analysis](https://ida.numbat.space/) class, and many students encounter this error:

```
Error in loadNamespace(name) : there is no package called 'PKG'
```

but they do not have the skills and experience to identify how to solve this problem. 
In class, I decided to showcase how I would try to solve this problem, live, on zoom, to my class. So I googled the error, and then I discovered that the top hit is[this stack overflow page](https://stackoverflow.com/questions/16758129/error-in-loadnamespacename-there-is-no-package-called-rcpp), which was decidedly not helpful for the problem that my students had.

So I wrote this blogpost.

Hopefully it's helpful!

Next up in this series is tackling this problem:

```
Error: package or namespace load failed for 'tidyverse' in loadNamespace(j <- i[[1L]], c(lib.loc, .libPaths()), versionCheck = vI[[j]]): namespace 'tibble' 2.1.3 is already loaded, but >= 3.0.0 is required
```
