---
title: '`naniar` Version 1.1.0 "Prince Caspian"'
author: Nicholas Tierney
date: '2024-03-04'
slug: naniar-version-1-1-0-prince-caspian
categories:
  - rstats
  - Missing Data
  - rbloggers
  - research software engineer
  - data visualisation
tags:
  - data science
  - missing-data
  - rbloggers
  - research software engineer
draft: yes
output: hugodown::md_document
rmd_hash: 8469c091cafbfbcb

---

I'm happy to announce that naniar version 1.1.0 "Prince Caspian" is released. This post will briefly touch on some of the changes that were made in this release.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'><a href='https://rdrr.io/r/base/options.html'>options</a></span><span class='o'>(</span>tidyverse.quiet <span class='o'>=</span> <span class='kc'>TRUE</span><span class='o'>)</span></span>
<span><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://tidyverse.tidyverse.org'>tidyverse</a></span><span class='o'>)</span></span>
<span><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://github.com/njtierney/naniar'>naniar</a></span><span class='o'>)</span></span></code></pre>

</div>

# Imputation helpers: `impute_fixed`, `impute_zero`, `impute_factor`, and `impute_mode`

I've been hesitant to include imputation helper functions like `impute_mean`, and friends in `naniar`, since I believe that the `simputation` R package does a great job at doing good imputation. However, there are some cases where imputation makes sense for `impute_mean`. Where previously I wanted to avoid users shooting themselves in the foot, I do believe that there are valid uses of these functions.

This release included improvements to `impute_fixed`, `impute_zero`, `impute_factor`, and `impute_mode`. notably these do not implement "scoped variants" which were previously implemented - for example, `impute_fixed_if` etc. This is in favour of using the new `across` workflow within `dplyr`, and it is easier to maintain. This resolves issues [#261](https://github.com//njtierney/naniar/issues/261) and [#213](https://github.com//njtierney/naniar/issues/213).

Here's a demonstration of these functions. Let's create a vector of missing values:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>vec_num</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/stats/Normal.html'>rnorm</a></span><span class='o'>(</span><span class='m'>10</span><span class='o'>)</span></span>
<span><span class='nv'>vec_int</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/stats/Poisson.html'>rpois</a></span><span class='o'>(</span><span class='m'>10</span>, <span class='m'>5</span><span class='o'>)</span></span>
<span><span class='nv'>vec_fct</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/factor.html'>factor</a></span><span class='o'>(</span><span class='nv'>LETTERS</span><span class='o'>[</span><span class='m'>1</span><span class='o'>:</span><span class='m'>10</span><span class='o'>]</span><span class='o'>)</span></span>
<span></span>
<span><span class='nv'>vec_num</span> <span class='o'>&lt;-</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/set-prop-n-miss.html'>set_prop_miss</a></span><span class='o'>(</span><span class='nv'>vec_num</span>, <span class='m'>0.4</span><span class='o'>)</span></span>
<span><span class='nv'>vec_int</span> <span class='o'>&lt;-</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/set-prop-n-miss.html'>set_prop_miss</a></span><span class='o'>(</span><span class='nv'>vec_int</span>, <span class='m'>0.4</span><span class='o'>)</span></span>
<span><span class='nv'>vec_fct</span> <span class='o'>&lt;-</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/set-prop-n-miss.html'>set_prop_miss</a></span><span class='o'>(</span><span class='nv'>vec_fct</span>, <span class='m'>0.4</span><span class='o'>)</span></span>
<span></span>
<span><span class='nv'>vec_num</span></span>
<span><span class='c'>#&gt;  [1]         NA -0.8741683  0.1950870  0.4785823         NA  0.7194988</span></span>
<span><span class='c'>#&gt;  [7] -0.3733622         NA         NA -2.1250407</span></span>
<span></span><span><span class='nv'>vec_int</span></span>
<span><span class='c'>#&gt;  [1] NA NA  5 NA  4  3 NA  9  7  3</span></span>
<span></span><span><span class='nv'>vec_fct</span></span>
<span><span class='c'>#&gt;  [1] A    B    C    D    &lt;NA&gt; &lt;NA&gt; G    H    &lt;NA&gt; &lt;NA&gt;</span></span>
<span><span class='c'>#&gt; Levels: A B C D E F G H I J</span></span>
<span></span></code></pre>

</div>

We can impute fixed values into the numeric of these:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_fixed.html'>impute_fixed</a></span><span class='o'>(</span><span class='nv'>vec_num</span>, <span class='o'>-</span><span class='m'>999</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;  [1] -999.0000000   -0.8741683    0.1950870    0.4785823 -999.0000000</span></span>
<span><span class='c'>#&gt;  [6]    0.7194988   -0.3733622 -999.0000000 -999.0000000   -2.1250407</span></span>
<span></span><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_fixed.html'>impute_fixed</a></span><span class='o'>(</span><span class='nv'>vec_int</span>, <span class='o'>-</span><span class='m'>999</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;  [1] -999 -999    5 -999    4    3 -999    9    7    3</span></span>
<span></span></code></pre>

</div>

And `impute_zero` is just a special case of `impute_fixed`, where the fixed value is 0:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_zero.html'>impute_zero</a></span><span class='o'>(</span><span class='nv'>vec_num</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;  [1]  0.0000000 -0.8741683  0.1950870  0.4785823  0.0000000  0.7194988</span></span>
<span><span class='c'>#&gt;  [7] -0.3733622  0.0000000  0.0000000 -2.1250407</span></span>
<span></span><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_zero.html'>impute_zero</a></span><span class='o'>(</span><span class='nv'>vec_int</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;  [1] 0 0 5 0 4 3 0 9 7 3</span></span>
<span></span></code></pre>

</div>

Similar to `impute_mean`, `impute_mode` imputes the mode, the most common number.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_mode.html'>impute_mode</a></span><span class='o'>(</span><span class='nv'>vec_num</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;  [1]  0.2704336 -0.8741683  0.1950870  0.4785823  0.2704336  0.7194988</span></span>
<span><span class='c'>#&gt;  [7] -0.3733622  0.2704336  0.2704336 -2.1250407</span></span>
<span></span><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_mode.html'>impute_mode</a></span><span class='o'>(</span><span class='nv'>vec_int</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;  [1] 4 4 5 4 4 3 4 9 7 3</span></span>
<span></span></code></pre>

</div>

You can't however use [`impute_fixed()`](http://naniar.njtierney.com/reference/impute_fixed.html) or [`impute_zero()`](http://naniar.njtierney.com/reference/impute_zero.html) on factors, and this doesn't work for factors, even if it's a new character.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_fixed.html'>impute_fixed</a></span><span class='o'>(</span><span class='nv'>vec_fct</span>, <span class='o'>-</span><span class='m'>999</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; Warning in `[&lt;-.factor`(`*tmp*`, is.na(x), value = -999): invalid factor level, NA generated</span></span>
<span></span><span><span class='c'>#&gt;  [1] A    B    C    D    &lt;NA&gt; &lt;NA&gt; G    H    &lt;NA&gt; &lt;NA&gt;</span></span>
<span><span class='c'>#&gt; Levels: A B C D E F G H I J</span></span>
<span></span><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_zero.html'>impute_zero</a></span><span class='o'>(</span><span class='nv'>vec_fct</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; Warning in `[&lt;-.factor`(`*tmp*`, is.na(x), value = 0): invalid factor level, NA generated</span></span>
<span></span><span><span class='c'>#&gt;  [1] A    B    C    D    &lt;NA&gt; &lt;NA&gt; G    H    &lt;NA&gt; &lt;NA&gt;</span></span>
<span><span class='c'>#&gt; Levels: A B C D E F G H I J</span></span>
<span></span><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_fixed.html'>impute_fixed</a></span><span class='o'>(</span><span class='nv'>vec_fct</span>, <span class='s'>"ZZ"</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; Warning in `[&lt;-.factor`(`*tmp*`, is.na(x), value = "ZZ"): invalid factor level, NA generated</span></span>
<span></span><span><span class='c'>#&gt;  [1] A    B    C    D    &lt;NA&gt; &lt;NA&gt; G    H    &lt;NA&gt; &lt;NA&gt;</span></span>
<span><span class='c'>#&gt; Levels: A B C D E F G H I J</span></span>
<span></span></code></pre>

</div>

However, you can use `impute_mode`:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_mode.html'>impute_mode</a></span><span class='o'>(</span><span class='nv'>vec_fct</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;  [1] A B C D B B G H B B</span></span>
<span><span class='c'>#&gt; Levels: A B C D E F G H I J</span></span>
<span></span></code></pre>

</div>

For factors, you can impute a specific value:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'><a href='http://naniar.njtierney.com/reference/impute_factor.html'>impute_factor</a></span><span class='o'>(</span><span class='nv'>vec_fct</span>, value <span class='o'>=</span> <span class='s'>"ZZ"</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;  [1] A  B  C  D  ZZ ZZ G  H  ZZ ZZ</span></span>
<span><span class='c'>#&gt; Levels: A B C D E F G H I J ZZ</span></span>
<span></span></code></pre>

</div>

Think of it like `impute_fixed`, but a special case for factors.

Now let's demonstrate how to do this in a data frame. First we create the data

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://dplyr.tidyverse.org'>dplyr</a></span><span class='o'>)</span></span>
<span></span>
<span><span class='nv'>dat</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://tibble.tidyverse.org/reference/tibble.html'>tibble</a></span><span class='o'>(</span></span>
<span>  num <span class='o'>=</span> <span class='nv'>vec_num</span>,</span>
<span>  int <span class='o'>=</span> <span class='nv'>vec_int</span>,</span>
<span>  fct <span class='o'>=</span> <span class='nv'>vec_fct</span></span>
<span><span class='o'>)</span></span>
<span></span>
<span><span class='nv'>dat</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 3</span></span></span>
<span><span class='c'>#&gt;       num   int fct  </span></span>
<span><span class='c'>#&gt;     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;fct&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span> <span style='color: #BB0000;'>NA</span>        <span style='color: #BB0000;'>NA</span> A    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span> -<span style='color: #BB0000;'>0.874</span>    <span style='color: #BB0000;'>NA</span> B    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>  0.195     5 C    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>  0.479    <span style='color: #BB0000;'>NA</span> D    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span> <span style='color: #BB0000;'>NA</span>         4 <span style='color: #BB0000;'>NA</span>   </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>  0.719     3 <span style='color: #BB0000;'>NA</span>   </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span> -<span style='color: #BB0000;'>0.373</span>    <span style='color: #BB0000;'>NA</span> G    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span> <span style='color: #BB0000;'>NA</span>         9 H    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span> <span style='color: #BB0000;'>NA</span>         7 <span style='color: #BB0000;'>NA</span>   </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span> -<span style='color: #BB0000;'>2.13</span>      3 <span style='color: #BB0000;'>NA</span></span></span>
<span></span></code></pre>

</div>

You can use it inside mutate like so:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span></span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    num <span class='o'>=</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/impute_fixed.html'>impute_fixed</a></span><span class='o'>(</span><span class='nv'>num</span>, <span class='o'>-</span><span class='m'>9999</span><span class='o'>)</span>,</span>
<span>    int <span class='o'>=</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/impute_zero.html'>impute_zero</a></span><span class='o'>(</span><span class='nv'>int</span><span class='o'>)</span>,</span>
<span>    fct <span class='o'>=</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/impute_factor.html'>impute_factor</a></span><span class='o'>(</span><span class='nv'>fct</span>, <span class='s'>"out"</span><span class='o'>)</span></span>
<span>  <span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 3</span></span></span>
<span><span class='c'>#&gt;          num   int fct  </span></span>
<span><span class='c'>#&gt;        <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;fct&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span> -<span style='color: #BB0000; text-decoration: underline;'>9</span><span style='color: #BB0000;'>999</span>         0 A    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span>    -<span style='color: #BB0000;'>0.874</span>     0 B    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>     0.195     5 C    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>     0.479     0 D    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span> -<span style='color: #BB0000; text-decoration: underline;'>9</span><span style='color: #BB0000;'>999</span>         4 out  </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>     0.719     3 out  </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span>    -<span style='color: #BB0000;'>0.373</span>     0 G    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span> -<span style='color: #BB0000; text-decoration: underline;'>9</span><span style='color: #BB0000;'>999</span>         9 H    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span> -<span style='color: #BB0000; text-decoration: underline;'>9</span><span style='color: #BB0000;'>999</span>         7 out  </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span>    -<span style='color: #BB0000;'>2.13</span>      3 out</span></span>
<span></span></code></pre>

</div>

Or if you want to impute across all applicable variables with a single function, you could use `where` like so:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span></span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    <span class='nf'><a href='https://dplyr.tidyverse.org/reference/across.html'>across</a></span><span class='o'>(</span></span>
<span>      .cols <span class='o'>=</span> <span class='nf'><a href='https://tidyselect.r-lib.org/reference/where.html'>where</a></span><span class='o'>(</span><span class='nv'>is.numeric</span><span class='o'>)</span>,</span>
<span>      .fn <span class='o'>=</span> <span class='nv'>impute_zero</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 3</span></span></span>
<span><span class='c'>#&gt;       num   int fct  </span></span>
<span><span class='c'>#&gt;     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;fct&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span>  0         0 A    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span> -<span style='color: #BB0000;'>0.874</span>     0 B    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>  0.195     5 C    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>  0.479     0 D    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span>  0         4 <span style='color: #BB0000;'>NA</span>   </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>  0.719     3 <span style='color: #BB0000;'>NA</span>   </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span> -<span style='color: #BB0000;'>0.373</span>     0 G    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span>  0         9 H    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span>  0         7 <span style='color: #BB0000;'>NA</span>   </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span> -<span style='color: #BB0000;'>2.13</span>      3 <span style='color: #BB0000;'>NA</span></span></span>
<span></span><span></span>
<span><span class='nv'>dat</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span></span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    <span class='nf'><a href='https://dplyr.tidyverse.org/reference/across.html'>across</a></span><span class='o'>(</span></span>
<span>      .cols <span class='o'>=</span> <span class='nf'><a href='https://tidyselect.r-lib.org/reference/where.html'>where</a></span><span class='o'>(</span><span class='nv'>is.numeric</span><span class='o'>)</span>,</span>
<span>      .fn <span class='o'>=</span> \<span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/impute_fixed.html'>impute_fixed</a></span><span class='o'>(</span><span class='nv'>x</span>, <span class='o'>-</span><span class='m'>99</span><span class='o'>)</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 3</span></span></span>
<span><span class='c'>#&gt;        num   int fct  </span></span>
<span><span class='c'>#&gt;      <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;fct&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span> -<span style='color: #BB0000;'>99</span>       -<span style='color: #BB0000;'>99</span> A    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span>  -<span style='color: #BB0000;'>0.874</span>   -<span style='color: #BB0000;'>99</span> B    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>   0.195     5 C    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>   0.479   -<span style='color: #BB0000;'>99</span> D    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span> -<span style='color: #BB0000;'>99</span>         4 <span style='color: #BB0000;'>NA</span>   </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>   0.719     3 <span style='color: #BB0000;'>NA</span>   </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span>  -<span style='color: #BB0000;'>0.373</span>   -<span style='color: #BB0000;'>99</span> G    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span> -<span style='color: #BB0000;'>99</span>         9 H    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span> -<span style='color: #BB0000;'>99</span>         7 <span style='color: #BB0000;'>NA</span>   </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span>  -<span style='color: #BB0000;'>2.13</span>      3 <span style='color: #BB0000;'>NA</span></span></span>
<span></span></code></pre>

</div>

# Improvements

-   Add `digit` argument to `miss_var_summary` to help display %missing data correctly when there is a very small fraction of missingness. [#284](https://github.com//njtierney/naniar/issues/284)

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>N</span> <span class='o'>&lt;-</span> <span class='m'>30000000</span></span>
<span></span>
<span><span class='nv'>df</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://tibble.tidyverse.org/reference/tibble.html'>tibble</a></span><span class='o'>(</span>x <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/rep.html'>rep</a></span><span class='o'>(</span><span class='kc'>NA_real_</span>, <span class='nv'>N</span><span class='o'>)</span><span class='o'>)</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span></span>
<span>  <span class='nf'><a href='https://tibble.tidyverse.org/reference/add_row.html'>add_row</a></span><span class='o'>(</span>x <span class='o'>=</span> <span class='m'>0</span><span class='o'>)</span></span>
<span></span>
<span><span class='nv'>df</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> <span class='nf'><a href='http://naniar.njtierney.com/reference/miss_var_summary.html'>miss_var_summary</a></span><span class='o'>(</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 × 3</span></span></span>
<span><span class='c'>#&gt;   variable   n_miss pct_miss</span></span>
<span><span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>       <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;num&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>1</span> x        30<span style='text-decoration: underline;'>000</span>000     100.</span></span>
<span></span><span><span class='nv'>df</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> <span class='nf'><a href='http://naniar.njtierney.com/reference/miss_var_summary.html'>miss_var_summary</a></span><span class='o'>(</span>digits <span class='o'>=</span> <span class='m'>6</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 × 3</span></span></span>
<span><span class='c'>#&gt;   variable   n_miss  pct_miss</span></span>
<span><span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>       <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;num:.6!&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>1</span> x        30<span style='text-decoration: underline;'>000</span>000 99.999<span style='text-decoration: underline;'>997</span></span></span>
<span></span></code></pre>

</div>

-   [`geom_miss_point()`](http://naniar.njtierney.com/reference/geom_miss_point.html) works with `shape` argument [#290](https://github.com//njtierney/naniar/issues/290)

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'><a href='https://ggplot2.tidyverse.org/reference/ggplot.html'>ggplot</a></span><span class='o'>(</span></span>
<span>  <span class='nv'>airquality</span>,</span>
<span>  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span></span>
<span>    x <span class='o'>=</span> <span class='nv'>Ozone</span>,</span>
<span>    y <span class='o'>=</span> <span class='nv'>Solar.R</span>,</span>
<span>    shape <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/factor.html'>factor</a></span><span class='o'>(</span><span class='nv'>Month</span><span class='o'>)</span></span>
<span>  <span class='o'>)</span></span>
<span><span class='o'>)</span> <span class='o'>+</span></span>
<span>  <span class='nf'><a href='http://naniar.njtierney.com/reference/geom_miss_point.html'>geom_miss_point</a></span><span class='o'>(</span>size <span class='o'>=</span> <span class='m'>4</span><span class='o'>)</span></span>
</code></pre>
<img src="figs/unnamed-chunk-13-1.png" width="700px" style="display: block; margin: auto;" />

</div>

-   Implement `Date`, `POSIXct` and `POSIXlt` methods for [`impute_below()`](http://naniar.njtierney.com/reference/impute_below.html) - [#158](https://github.com//njtierney/naniar/issues/158)

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat_date</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://tibble.tidyverse.org/reference/tibble.html'>tibble</a></span><span class='o'>(</span></span>
<span>  values <span class='o'>=</span> <span class='m'>1</span><span class='o'>:</span><span class='m'>7</span>,</span>
<span>  number <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='m'>111</span>, <span class='m'>112</span>, <span class='kc'>NA</span>, <span class='kc'>NA</span>, <span class='m'>108</span>, <span class='m'>150</span>, <span class='m'>160</span><span class='o'>)</span>,</span>
<span>  posixct <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/as.POSIXlt.html'>as.POSIXct</a></span><span class='o'>(</span><span class='nv'>number</span>, origin <span class='o'>=</span> <span class='s'>"1970-01-01"</span><span class='o'>)</span>,</span>
<span>  posixlt <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/as.POSIXlt.html'>as.POSIXlt</a></span><span class='o'>(</span><span class='nv'>number</span>, origin <span class='o'>=</span> <span class='s'>"1970-01-01"</span><span class='o'>)</span>,</span>
<span>  date <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/as.Date.html'>as.Date</a></span><span class='o'>(</span><span class='nv'>number</span><span class='o'>)</span></span>
<span>  <span class='o'>)</span></span>
<span><span class='nv'>dat_date</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 7 × 5</span></span></span>
<span><span class='c'>#&gt;   values number posixct             posixlt             date      </span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dttm&gt;</span>              <span style='color: #555555; font-style: italic;'>&lt;dttm&gt;</span>              <span style='color: #555555; font-style: italic;'>&lt;date&gt;</span>    </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>1</span>      1    111 1970-01-01 <span style='color: #555555;'>10:01:51</span> 1970-01-01 <span style='color: #555555;'>10:01:51</span> 1970-04-22</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>2</span>      2    112 1970-01-01 <span style='color: #555555;'>10:01:52</span> 1970-01-01 <span style='color: #555555;'>10:01:52</span> 1970-04-23</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>3</span>      3     <span style='color: #BB0000;'>NA</span> <span style='color: #BB0000;'>NA</span>                  <span style='color: #BB0000;'>NA</span>                  <span style='color: #BB0000;'>NA</span>        </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>4</span>      4     <span style='color: #BB0000;'>NA</span> <span style='color: #BB0000;'>NA</span>                  <span style='color: #BB0000;'>NA</span>                  <span style='color: #BB0000;'>NA</span>        </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>5</span>      5    108 1970-01-01 <span style='color: #555555;'>10:01:48</span> 1970-01-01 <span style='color: #555555;'>10:01:48</span> 1970-04-19</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>6</span>      6    150 1970-01-01 <span style='color: #555555;'>10:02:30</span> 1970-01-01 <span style='color: #555555;'>10:02:30</span> 1970-05-31</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>7</span>      7    160 1970-01-01 <span style='color: #555555;'>10:02:40</span> 1970-01-01 <span style='color: #555555;'>10:02:40</span> 1970-06-10</span></span>
<span></span><span></span>
<span><span class='nv'>date_date_imp</span> <span class='o'>&lt;-</span> <span class='nv'>dat_date</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    number_imp <span class='o'>=</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/impute_below.html'>impute_below</a></span><span class='o'>(</span><span class='nv'>number</span><span class='o'>)</span>,</span>
<span>    .after <span class='o'>=</span> <span class='nv'>number</span></span>
<span>  <span class='o'>)</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    posixct_imp <span class='o'>=</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/impute_below.html'>impute_below</a></span><span class='o'>(</span><span class='nv'>posixct</span><span class='o'>)</span>,</span>
<span>    .after <span class='o'>=</span> <span class='nv'>posixct</span></span>
<span>    <span class='o'>)</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    posixlt_imp <span class='o'>=</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/impute_below.html'>impute_below</a></span><span class='o'>(</span><span class='nv'>posixlt</span><span class='o'>)</span>,</span>
<span>    .after <span class='o'>=</span> <span class='nv'>posixlt</span></span>
<span>  <span class='o'>)</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    date_imp <span class='o'>=</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/impute_below.html'>impute_below</a></span><span class='o'>(</span><span class='nv'>date</span><span class='o'>)</span>,</span>
<span>    .after <span class='o'>=</span> <span class='nv'>date</span></span>
<span>  <span class='o'>)</span></span>
<span></span>
<span><span class='nv'>date_date_imp</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 7 × 9</span></span></span>
<span><span class='c'>#&gt;   values number number_imp posixct             posixct_imp        </span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>      <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dttm&gt;</span>              <span style='color: #555555; font-style: italic;'>&lt;dttm&gt;</span>             </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>1</span>      1    111       111  1970-01-01 <span style='color: #555555;'>10:01:51</span> 1970-01-01 <span style='color: #555555;'>10:01:51</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>2</span>      2    112       112  1970-01-01 <span style='color: #555555;'>10:01:52</span> 1970-01-01 <span style='color: #555555;'>10:01:52</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>3</span>      3     <span style='color: #BB0000;'>NA</span>       103. <span style='color: #BB0000;'>NA</span>                  1970-01-01 <span style='color: #555555;'>10:01:43</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>4</span>      4     <span style='color: #BB0000;'>NA</span>       102. <span style='color: #BB0000;'>NA</span>                  1970-01-01 <span style='color: #555555;'>10:01:41</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>5</span>      5    108       108  1970-01-01 <span style='color: #555555;'>10:01:48</span> 1970-01-01 <span style='color: #555555;'>10:01:48</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>6</span>      6    150       150  1970-01-01 <span style='color: #555555;'>10:02:30</span> 1970-01-01 <span style='color: #555555;'>10:02:30</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>7</span>      7    160       160  1970-01-01 <span style='color: #555555;'>10:02:40</span> 1970-01-01 <span style='color: #555555;'>10:02:40</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># ℹ 4 more variables: posixlt &lt;dttm&gt;, posixlt_imp &lt;dttm&gt;, date &lt;date&gt;,</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>#   date_imp &lt;date&gt;</span></span></span>
<span></span></code></pre>

</div>

# Bug fixes

-   Fix bug with `all_complete`, which was implemented as `!anyNA(x)` but should be `all(complete.cases(x))`.
-   Correctly implement [`any_na()`](http://naniar.njtierney.com/reference/any-all-na-complete.html) (and [`any_miss()`](http://naniar.njtierney.com/reference/any-all-na-complete.html)) and [`any_complete()`](http://naniar.njtierney.com/reference/any-all-na-complete.html). Rework examples to demonstrate workflow for finding complete variables.
-   Fix bug with `shadow_long` not working when gathering variables of mixed type. Fix involved specifying a value transform, which defaults to character. [#314](https://github.com//njtierney/naniar/issues/314)
-   Provide `replace_na_with`, a complement to `replace_with_na` - [#129](https://github.com//njtierney/naniar/issues/129)

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>x</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='m'>1</span><span class='o'>:</span><span class='m'>5</span>, <span class='kc'>NA</span>, <span class='kc'>NA</span>, <span class='kc'>NA</span><span class='o'>)</span></span>
<span><span class='nv'>x</span></span>
<span><span class='c'>#&gt; [1]  1  2  3  4  5 NA NA NA</span></span>
<span></span><span><span class='nf'><a href='http://naniar.njtierney.com/reference/replace_na_with.html'>replace_na_with</a></span><span class='o'>(</span><span class='nv'>x</span>, <span class='m'>0</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; [1] 1 2 3 4 5 0 0 0</span></span>
<span></span><span></span>
<span><span class='nv'>dat</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://tibble.tidyverse.org/reference/tibble.html'>tibble</a></span><span class='o'>(</span></span>
<span>  ones <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='m'>1</span>,<span class='m'>1</span>,<span class='kc'>NA</span><span class='o'>)</span>,</span>
<span>  twos <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='kc'>NA</span>,<span class='kc'>NA</span>, <span class='m'>2</span><span class='o'>)</span>,</span>
<span>  threes <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='kc'>NA</span>, <span class='kc'>NA</span>, <span class='kc'>NA</span><span class='o'>)</span></span>
<span><span class='o'>)</span></span>
<span></span>
<span><span class='nv'>dat</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    ones <span class='o'>=</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/replace_na_with.html'>replace_na_with</a></span><span class='o'>(</span><span class='nv'>ones</span>, <span class='m'>0</span><span class='o'>)</span>,</span>
<span>    twos <span class='o'>=</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/replace_na_with.html'>replace_na_with</a></span><span class='o'>(</span><span class='nv'>twos</span>, <span class='o'>-</span><span class='m'>2</span><span class='o'>)</span>,</span>
<span>    threes <span class='o'>=</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/replace_na_with.html'>replace_na_with</a></span><span class='o'>(</span><span class='nv'>threes</span>, <span class='o'>-</span><span class='m'>3</span><span class='o'>)</span></span>
<span>  <span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 3 × 3</span></span></span>
<span><span class='c'>#&gt;    ones  twos threes</span></span>
<span><span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>1</span>     1    -<span style='color: #BB0000;'>2</span>     -<span style='color: #BB0000;'>3</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>2</span>     1    -<span style='color: #BB0000;'>2</span>     -<span style='color: #BB0000;'>3</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>3</span>     0     2     -<span style='color: #BB0000;'>3</span></span></span>
<span></span><span></span>
<span><span class='nv'>dat</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    <span class='nf'><a href='https://dplyr.tidyverse.org/reference/across.html'>across</a></span><span class='o'>(</span></span>
<span>      <span class='nf'><a href='https://tidyselect.r-lib.org/reference/everything.html'>everything</a></span><span class='o'>(</span><span class='o'>)</span>,</span>
<span>      \<span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span> <span class='nf'><a href='http://naniar.njtierney.com/reference/replace_na_with.html'>replace_na_with</a></span><span class='o'>(</span><span class='nv'>x</span>, <span class='o'>-</span><span class='m'>99</span><span class='o'>)</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 3 × 3</span></span></span>
<span><span class='c'>#&gt;    ones  twos threes</span></span>
<span><span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>1</span>     1   -<span style='color: #BB0000;'>99</span>    -<span style='color: #BB0000;'>99</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>2</span>     1   -<span style='color: #BB0000;'>99</span>    -<span style='color: #BB0000;'>99</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>3</span>   -<span style='color: #BB0000;'>99</span>     2    -<span style='color: #BB0000;'>99</span></span></span>
<span></span></code></pre>

</div>

-   Fix bug with `gg_miss_fct` where it used a deprecated function from forcats - [#342](https://github.com//njtierney/naniar/issues/342)

# Internal changes

Naniar now uses [`cli::cli_abort`](https://cli.r-lib.org/reference/cli_abort.html) and [`cli::cli_warn`](https://cli.r-lib.org/reference/cli_abort.html) instead of `stop` and `warning` ([#326](https://github.com//njtierney/naniar/issues/326)). Internally in tests we changed to use `expect_snapshot` instead of `expect_error`.

# Deprecations

The following functions have been soft deprecated, and will eventually be made defunct in future versions of naniar. [`shadow_shift()`](http://naniar.njtierney.com/reference/shadow_shift.html) - [#193](https://github.com//njtierney/naniar/issues/193) in favour of [`impute_below()`](http://naniar.njtierney.com/reference/impute_below.html), and [`miss_case_cumsum()`](http://naniar.njtierney.com/reference/miss_case_cumsum.html) and [`miss_var_cumsum()`](http://naniar.njtierney.com/reference/miss_var_cumsum.html) - [#257](https://github.com//njtierney/naniar/issues/257) - in favour of `miss_case_summary(data, cumsum = TRUE)` and `miss_var_summary(data, cumsum = TRUE)`.

# Thanks!

Thanks to everyone for using naniar, I'm happy that I've got another release out the door and am looking forward to more changes in the future. Especially thanks to everyone who contributed to issues and or pull request for this release, including: [@szimmer](https://github.com/szimmer), [@HughParsonage](https://github.com/HughParsonage), , [@siavash-babaei](https://github.com/siavash-babaei), [@maksymiuks](https://github.com/maksymiuks), [@jonocarroll](https://github.com/jonocarroll), [@jzadra](https://github.com/jzadra), [@krlmlr](https://github.com/krlmlr)

