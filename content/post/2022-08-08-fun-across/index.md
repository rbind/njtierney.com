---
title: A Fun Little Use of `dplyr::across()`
author: Nicholas Tierney
date: '2022-08-08'
slug: fun-across
categories:
  - rstats
  - functions
tags:
  - rstats
  - functions
output: hugodown::md_document
rmd_hash: 16f9a77ae8a31f58

---

<div class="highlight">

<img src="imgs/across-birds-small.jpg" width="700px" style="display: block; margin: auto;" />

</div>

*pigeons across the drain, Nick Tierney, film, olympus XA*

A colleague of mine the other day had a question along the lines of:

> How do I add multiple columns that give the ranks of values in corresponding columns

And I ended up cooking up a really fun example of using `across` from `dplyr`. I thought it would be fun to share!

Let's give a little more detail.

Load up the tidyverse.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://tidyverse.tidyverse.org'>tidyverse</a></span><span class='o'>)</span></span><span><span class='c'>#&gt; ── <span style='font-weight: bold;'>Attaching packages</span> ───────────────────────────── tidyverse 1.3.1 ──</span></span><span><span class='c'>#&gt; <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>ggplot2</span> 3.3.6     <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>purrr  </span> 0.3.4</span></span>
<span><span class='c'>#&gt; <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>tibble </span> 3.1.7     <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>dplyr  </span> 1.0.9</span></span>
<span><span class='c'>#&gt; <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>tidyr  </span> 1.2.0     <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>stringr</span> 1.4.0</span></span>
<span><span class='c'>#&gt; <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>readr  </span> 2.1.2     <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>forcats</span> 0.5.1</span></span><span><span class='c'>#&gt; ── <span style='font-weight: bold;'>Conflicts</span> ──────────────────────────────── tidyverse_conflicts() ──</span></span>
<span><span class='c'>#&gt; <span style='color: #BB0000;'>✖</span> <span style='color: #0000BB;'>dplyr</span>::<span style='color: #00BB00;'>filter()</span> masks <span style='color: #0000BB;'>stats</span>::filter()</span></span>
<span><span class='c'>#&gt; <span style='color: #BB0000;'>✖</span> <span style='color: #0000BB;'>dplyr</span>::<span style='color: #00BB00;'>lag()</span>    masks <span style='color: #0000BB;'>stats</span>::lag()</span></span></code></pre>

</div>

The data they had referred to concentrations of a quantity, that they wanted to rank. We can create these like so:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://tibble.tidyverse.org/reference/tibble.html'>tibble</a></span><span class='o'>(</span></span>
<span>  x_1 <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/stats/Uniform.html'>runif</a></span><span class='o'>(</span><span class='m'>10</span><span class='o'>)</span>,</span>
<span>  x_2 <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/stats/Uniform.html'>runif</a></span><span class='o'>(</span><span class='m'>10</span><span class='o'>)</span>,</span>
<span>  x_3 <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/stats/Uniform.html'>runif</a></span><span class='o'>(</span><span class='m'>10</span><span class='o'>)</span></span>
<span><span class='o'>)</span></span>
<span></span>
<span><span class='nv'>dat</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 3</span></span></span>
<span><span class='c'>#&gt;      x_1   x_2    x_3</span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span> 0.693 0.256 0.525 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span> 0.743 0.886 0.717 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span> 0.321 0.861 0.770 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span> 0.459 0.652 0.473 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span> 0.462 0.344 0.960 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span> 0.710 0.567 0.132 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span> 0.148 0.339 0.573 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span> 0.766 0.953 0.036<span style='text-decoration: underline;'>8</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span> 0.122 0.591 0.522 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span> 0.395 0.852 0.954</span></span></code></pre>

</div>

Or for fun, I did this:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>vals</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/stats/Uniform.html'>runif</a></span><span class='o'>(</span><span class='m'>30</span><span class='o'>)</span></span>
<span></span>
<span><span class='nv'>dat</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://tibble.tidyverse.org/reference/as_tibble.html'>as_tibble</a></span><span class='o'>(</span></span>
<span>  x <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/matrix.html'>matrix</a></span><span class='o'>(</span><span class='nv'>vals</span>, nrow <span class='o'>=</span> <span class='m'>10</span><span class='o'>)</span>,</span>
<span>  .name_repair <span class='o'>=</span> <span class='nf'>janitor</span><span class='nf'>::</span><span class='nv'><a href='https://rdrr.io/pkg/janitor/man/make_clean_names.html'>make_clean_names</a></span></span>
<span><span class='o'>)</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span></span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/rename.html'>rename</a></span><span class='o'>(</span></span>
<span>    x_1 <span class='o'>=</span> <span class='nv'>x</span></span>
<span>  <span class='o'>)</span></span>
<span></span>
<span><span class='nv'>dat</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 3</span></span></span>
<span><span class='c'>#&gt;       x_1    x_2    x_3</span></span>
<span><span class='c'>#&gt;     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span> 0.395  0.977  0.822 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span> 0.963  0.940  0.746 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span> 0.838  0.823  0.932 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span> 0.582  0.847  0.502 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span> 0.591  0.297  0.768 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span> 0.087<span style='text-decoration: underline;'>8</span> 0.329  0.540 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span> 0.060<span style='text-decoration: underline;'>5</span> 0.953  0.811 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span> 0.258  0.264  0.062<span style='text-decoration: underline;'>1</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span> 0.660  0.335  0.747 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span> 0.170  0.019<span style='text-decoration: underline;'>2</span> 0.154</span></span></code></pre>

</div>

> An aside: prefer `dat` over `df`. I've been burned recently by using `df` as my go-to name for data frames, but it turns out that `df` is actually a function in R, for calculating the density of an F distribution. Which is awesome, but sometimes leads to funny errors that you might not expect.

OK, so what my friend wanted was something like this:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span>  </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    x_1_rank <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/rank.html'>rank</a></span><span class='o'>(</span><span class='nv'>x_1</span><span class='o'>)</span>,</span>
<span>    x_2_rank <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/rank.html'>rank</a></span><span class='o'>(</span><span class='nv'>x_2</span><span class='o'>)</span>,</span>
<span>    x_3_rank <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/rank.html'>rank</a></span><span class='o'>(</span><span class='nv'>x_3</span><span class='o'>)</span></span>
<span>  <span class='o'>)</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 6</span></span></span>
<span><span class='c'>#&gt;       x_1    x_2    x_3 x_1_rank x_2_rank x_3_rank</span></span>
<span><span class='c'>#&gt;     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span> 0.395  0.977  0.822         5       10        9</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span> 0.963  0.940  0.746        10        8        5</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span> 0.838  0.823  0.932         9        6       10</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span> 0.582  0.847  0.502         6        7        3</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span> 0.591  0.297  0.768         7        3        7</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span> 0.087<span style='text-decoration: underline;'>8</span> 0.329  0.540         2        4        4</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span> 0.060<span style='text-decoration: underline;'>5</span> 0.953  0.811         1        9        8</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span> 0.258  0.264  0.062<span style='text-decoration: underline;'>1</span>        4        2        1</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span> 0.660  0.335  0.747         8        5        6</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span> 0.170  0.019<span style='text-decoration: underline;'>2</span> 0.154         3        1        2</span></span></code></pre>

</div>

Now, that's relatively easy. But they actually have a lot of these concentration columns - sometimes just 10, other times many times that number.

The reason you might want to avoid writing this out each time is because it will involve a lot of reptition, where you might accidentally copy something twice or not increment the numbers. E.g.,

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span>  </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    x_1_rank <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/rank.html'>rank</a></span><span class='o'>(</span><span class='nv'>x_1</span><span class='o'>)</span>,</span>
<span>    x_2_rank <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/rank.html'>rank</a></span><span class='o'>(</span><span class='nv'>x_2</span><span class='o'>)</span>,</span>
<span>    x_3_rank <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/rank.html'>rank</a></span><span class='o'>(</span><span class='nv'>x_2</span><span class='o'>)</span></span>
<span>  <span class='o'>)</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 6</span></span></span>
<span><span class='c'>#&gt;       x_1    x_2    x_3 x_1_rank x_2_rank x_3_rank</span></span>
<span><span class='c'>#&gt;     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span> 0.395  0.977  0.822         5       10       10</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span> 0.963  0.940  0.746        10        8        8</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span> 0.838  0.823  0.932         9        6        6</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span> 0.582  0.847  0.502         6        7        7</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span> 0.591  0.297  0.768         7        3        3</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span> 0.087<span style='text-decoration: underline;'>8</span> 0.329  0.540         2        4        4</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span> 0.060<span style='text-decoration: underline;'>5</span> 0.953  0.811         1        9        9</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span> 0.258  0.264  0.062<span style='text-decoration: underline;'>1</span>        4        2        2</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span> 0.660  0.335  0.747         8        5        5</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span> 0.170  0.019<span style='text-decoration: underline;'>2</span> 0.154         3        1        1</span></span></code></pre>

</div>

See the error? `x_3_rank` is actually from `rank(x_2)`

So the question is, can you generalise this type of column creation?

Yes! Yes we can. With `across`. It looks like this.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    <span class='nf'><a href='https://dplyr.tidyverse.org/reference/across.html'>across</a></span><span class='o'>(</span></span>
<span>      .cols <span class='o'>=</span> <span class='nf'><a href='https://tidyselect.r-lib.org/reference/everything.html'>everything</a></span><span class='o'>(</span><span class='o'>)</span>,</span>
<span>      .fns <span class='o'>=</span> <span class='nv'>rank</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>)</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 3</span></span></span>
<span><span class='c'>#&gt;      x_1   x_2   x_3</span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span>     5    10     9</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span>    10     8     5</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>     9     6    10</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>     6     7     3</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span>     7     3     7</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>     2     4     4</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span>     1     9     8</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span>     4     2     1</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span>     8     5     6</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span>     3     1     2</span></span></code></pre>

</div>

This says:

> mutate across all columns, and use the function, `rank`, on each one.

Uh, but we want the other columns preserved...how do we do that?

With `.names` - we specify a special pattern on how we want to name the columns with `.names = "{.col}_rank"`, which says: "call it the name of the column, and then add"\_rank" to it.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    <span class='nf'><a href='https://dplyr.tidyverse.org/reference/across.html'>across</a></span><span class='o'>(</span></span>
<span>      .cols <span class='o'>=</span> <span class='nf'><a href='https://tidyselect.r-lib.org/reference/everything.html'>everything</a></span><span class='o'>(</span><span class='o'>)</span>,</span>
<span>      .fns <span class='o'>=</span> <span class='nv'>rank</span>,</span>
<span>      .names <span class='o'>=</span> <span class='s'>"&#123;.col&#125;_rank"</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>)</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 6</span></span></span>
<span><span class='c'>#&gt;       x_1    x_2    x_3 x_1_rank x_2_rank x_3_rank</span></span>
<span><span class='c'>#&gt;     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span> 0.395  0.977  0.822         5       10        9</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span> 0.963  0.940  0.746        10        8        5</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span> 0.838  0.823  0.932         9        6       10</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span> 0.582  0.847  0.502         6        7        3</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span> 0.591  0.297  0.768         7        3        7</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span> 0.087<span style='text-decoration: underline;'>8</span> 0.329  0.540         2        4        4</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span> 0.060<span style='text-decoration: underline;'>5</span> 0.953  0.811         1        9        8</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span> 0.258  0.264  0.062<span style='text-decoration: underline;'>1</span>        4        2        1</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span> 0.660  0.335  0.747         8        5        6</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span> 0.170  0.019<span style='text-decoration: underline;'>2</span> 0.154         3        1        2</span></span></code></pre>

</div>

OK, but what if we've got data that contains other columns we don't want to apply `rank` to - say, some data that looks like this:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat_more</span> <span class='o'>&lt;-</span> <span class='nv'>dat</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://tibble.tidyverse.org/reference/rownames.html'>rowid_to_column</a></span><span class='o'>(</span>var <span class='o'>=</span> <span class='s'>"id"</span><span class='o'>)</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span>code <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/sample.html'>sample</a></span><span class='o'>(</span><span class='nv'>LETTERS</span>, <span class='nf'><a href='https://dplyr.tidyverse.org/reference/context.html'>n</a></span><span class='o'>(</span><span class='o'>)</span><span class='o'>)</span>,</span>
<span>         .after <span class='o'>=</span> <span class='nv'>id</span><span class='o'>)</span></span>
<span></span>
<span><span class='nv'>dat_more</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 5</span></span></span>
<span><span class='c'>#&gt;       id code     x_1    x_2    x_3</span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span>     1 X     0.395  0.977  0.822 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span>     2 C     0.963  0.940  0.746 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>     3 E     0.838  0.823  0.932 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>     4 S     0.582  0.847  0.502 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span>     5 D     0.591  0.297  0.768 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>     6 N     0.087<span style='text-decoration: underline;'>8</span> 0.329  0.540 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span>     7 K     0.060<span style='text-decoration: underline;'>5</span> 0.953  0.811 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span>     8 V     0.258  0.264  0.062<span style='text-decoration: underline;'>1</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span>     9 U     0.660  0.335  0.747 </span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span>    10 M     0.170  0.019<span style='text-decoration: underline;'>2</span> 0.154</span></span></code></pre>

</div>

We can tell it which columns to pay attention to, or even avoid, like so:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='c'># pay attention to "x_"</span></span>
<span><span class='nv'>dat_more_rank</span> <span class='o'>&lt;-</span> <span class='nv'>dat_more</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    <span class='nf'><a href='https://dplyr.tidyverse.org/reference/across.html'>across</a></span><span class='o'>(</span></span>
<span>      .cols <span class='o'>=</span> <span class='nf'><a href='https://tidyselect.r-lib.org/reference/starts_with.html'>starts_with</a></span><span class='o'>(</span><span class='s'>"x_"</span><span class='o'>)</span>,</span>
<span>      .fns <span class='o'>=</span> <span class='nv'>rank</span>,</span>
<span>      .names <span class='o'>=</span> <span class='s'>"&#123;.col&#125;_rank"</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>)</span></span>
<span></span>
<span><span class='nv'>dat_more_rank</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 8</span></span></span>
<span><span class='c'>#&gt;       id code     x_1    x_2    x_3 x_1_rank x_2_rank x_3_rank</span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span>     1 X     0.395  0.977  0.822         5       10        9</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span>     2 C     0.963  0.940  0.746        10        8        5</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>     3 E     0.838  0.823  0.932         9        6       10</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>     4 S     0.582  0.847  0.502         6        7        3</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span>     5 D     0.591  0.297  0.768         7        3        7</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>     6 N     0.087<span style='text-decoration: underline;'>8</span> 0.329  0.540         2        4        4</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span>     7 K     0.060<span style='text-decoration: underline;'>5</span> 0.953  0.811         1        9        8</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span>     8 V     0.258  0.264  0.062<span style='text-decoration: underline;'>1</span>        4        2        1</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span>     9 U     0.660  0.335  0.747         8        5        6</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span>    10 M     0.170  0.019<span style='text-decoration: underline;'>2</span> 0.154         3        1        2</span></span><span></span>
<span><span class='c'># do it to everything EXCEPT</span></span>
<span><span class='nv'>dat_more</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/mutate.html'>mutate</a></span><span class='o'>(</span></span>
<span>    <span class='nf'><a href='https://dplyr.tidyverse.org/reference/across.html'>across</a></span><span class='o'>(</span></span>
<span>      .cols <span class='o'>=</span> <span class='o'>-</span><span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='nv'>id</span>, <span class='nv'>code</span><span class='o'>)</span>,</span>
<span>      .fns <span class='o'>=</span> <span class='nv'>rank</span>,</span>
<span>      .names <span class='o'>=</span> <span class='s'>"&#123;.col&#125;_rank"</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>)</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 8</span></span></span>
<span><span class='c'>#&gt;       id code     x_1    x_2    x_3 x_1_rank x_2_rank x_3_rank</span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span>     1 X     0.395  0.977  0.822         5       10        9</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span>     2 C     0.963  0.940  0.746        10        8        5</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>     3 E     0.838  0.823  0.932         9        6       10</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>     4 S     0.582  0.847  0.502         6        7        3</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span>     5 D     0.591  0.297  0.768         7        3        7</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>     6 N     0.087<span style='text-decoration: underline;'>8</span> 0.329  0.540         2        4        4</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span>     7 K     0.060<span style='text-decoration: underline;'>5</span> 0.953  0.811         1        9        8</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span>     8 V     0.258  0.264  0.062<span style='text-decoration: underline;'>1</span>        4        2        1</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span>     9 U     0.660  0.335  0.747         8        5        6</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span>    10 M     0.170  0.019<span style='text-decoration: underline;'>2</span> 0.154         3        1        2</span></span></code></pre>

</div>

OK, but can we make this look a little bit nicer by having the columns be like `x_1`, then `x_1_rank` and so on?

Sure! We can take advantage of the fact that we've got a pretty systematic naming scheme in this case.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>new_col_order</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/sort.html'>sort</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/names.html'>names</a></span><span class='o'>(</span><span class='nv'>dat_more_rank</span><span class='o'>)</span><span class='o'>)</span></span>
<span></span>
<span><span class='nv'>new_col_order</span></span><span><span class='c'>#&gt; [1] "code"     "id"       "x_1"      "x_1_rank" "x_2"      "x_2_rank" "x_3"     </span></span>
<span><span class='c'>#&gt; [8] "x_3_rank"</span></span></code></pre>

</div>

We can use `relocate` to specify a new column order:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat_more_rank</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/relocate.html'>relocate</a></span><span class='o'>(</span></span>
<span>    <span class='nv'>id</span>,</span>
<span>    <span class='nv'>code</span>,</span>
<span>    <span class='nv'>new_col_order</span></span>
<span>  <span class='o'>)</span></span><span><span class='c'>#&gt; Note: Using an external vector in selections is ambiguous.</span></span>
<span><span class='c'>#&gt; <span style='color: #0000BB;'>ℹ</span> Use `all_of(new_col_order)` instead of `new_col_order` to silence this message.</span></span>
<span><span class='c'>#&gt; <span style='color: #0000BB;'>ℹ</span> See &lt;https://tidyselect.r-lib.org/reference/faq-external-vector.html&gt;.</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>This message is displayed once per session.</span></span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 8</span></span></span>
<span><span class='c'>#&gt;       id code     x_1 x_1_rank    x_2 x_2_rank    x_3 x_3_rank</span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span>     1 X     0.395         5 0.977        10 0.822         9</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span>     2 C     0.963        10 0.940         8 0.746         5</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>     3 E     0.838         9 0.823         6 0.932        10</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>     4 S     0.582         6 0.847         7 0.502         3</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span>     5 D     0.591         7 0.297         3 0.768         7</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>     6 N     0.087<span style='text-decoration: underline;'>8</span>        2 0.329         4 0.540         4</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span>     7 K     0.060<span style='text-decoration: underline;'>5</span>        1 0.953         9 0.811         8</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span>     8 V     0.258         4 0.264         2 0.062<span style='text-decoration: underline;'>1</span>        1</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span>     9 U     0.660         8 0.335         5 0.747         6</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span>    10 M     0.170         3 0.019<span style='text-decoration: underline;'>2</span>        1 0.154         2</span></span></code></pre>

</div>

Huh, apparently we need to use `all_of`, since using an external vector is ambiguous! Right you are, tidyverse team.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat_more_rank</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/relocate.html'>relocate</a></span><span class='o'>(</span></span>
<span>    <span class='nv'>id</span>,</span>
<span>    <span class='nv'>code</span>,</span>
<span>    <span class='nf'><a href='https://tidyselect.r-lib.org/reference/all_of.html'>all_of</a></span><span class='o'>(</span><span class='nv'>new_col_order</span><span class='o'>)</span></span>
<span>  <span class='o'>)</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 8</span></span></span>
<span><span class='c'>#&gt;       id code     x_1 x_1_rank    x_2 x_2_rank    x_3 x_3_rank</span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span>     1 X     0.395         5 0.977        10 0.822         9</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span>     2 C     0.963        10 0.940         8 0.746         5</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>     3 E     0.838         9 0.823         6 0.932        10</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>     4 S     0.582         6 0.847         7 0.502         3</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span>     5 D     0.591         7 0.297         3 0.768         7</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>     6 N     0.087<span style='text-decoration: underline;'>8</span>        2 0.329         4 0.540         4</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span>     7 K     0.060<span style='text-decoration: underline;'>5</span>        1 0.953         9 0.811         8</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span>     8 V     0.258         4 0.264         2 0.062<span style='text-decoration: underline;'>1</span>        1</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span>     9 U     0.660         8 0.335         5 0.747         6</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span>    10 M     0.170         3 0.019<span style='text-decoration: underline;'>2</span>        1 0.154         2</span></span></code></pre>

</div>

Or alternatively:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat_more_rank</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/relocate.html'>relocate</a></span><span class='o'>(</span></span>
<span>    <span class='nv'>id</span>,</span>
<span>    <span class='nv'>code</span>,</span>
<span>    <span class='nf'><a href='https://rdrr.io/r/base/sort.html'>sort</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/names.html'>names</a></span><span class='o'>(</span><span class='nv'>.</span><span class='o'>)</span><span class='o'>)</span></span>
<span>  <span class='o'>)</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 8</span></span></span>
<span><span class='c'>#&gt;       id code     x_1 x_1_rank    x_2 x_2_rank    x_3 x_3_rank</span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span>     1 X     0.395         5 0.977        10 0.822         9</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span>     2 C     0.963        10 0.940         8 0.746         5</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>     3 E     0.838         9 0.823         6 0.932        10</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>     4 S     0.582         6 0.847         7 0.502         3</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span>     5 D     0.591         7 0.297         3 0.768         7</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>     6 N     0.087<span style='text-decoration: underline;'>8</span>        2 0.329         4 0.540         4</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span>     7 K     0.060<span style='text-decoration: underline;'>5</span>        1 0.953         9 0.811         8</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span>     8 V     0.258         4 0.264         2 0.062<span style='text-decoration: underline;'>1</span>        1</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span>     9 U     0.660         8 0.335         5 0.747         6</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span>    10 M     0.170         3 0.019<span style='text-decoration: underline;'>2</span>        1 0.154         2</span></span></code></pre>

</div>

`relocate` is a relatively new operation, it just exists to relocate existing columns, and won't remove other columns. In the past I would have done something like:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>dat_more_rank</span> <span class='o'><a href='https://magrittr.tidyverse.org/reference/pipe.html'>%&gt;%</a></span> </span>
<span>  <span class='nf'><a href='https://dplyr.tidyverse.org/reference/select.html'>select</a></span><span class='o'>(</span></span>
<span>    <span class='nv'>id</span>,</span>
<span>    <span class='nv'>code</span>,</span>
<span>    <span class='nf'><a href='https://tidyselect.r-lib.org/reference/all_of.html'>all_of</a></span><span class='o'>(</span><span class='nv'>new_col_order</span><span class='o'>)</span>,</span>
<span>    <span class='nf'><a href='https://tidyselect.r-lib.org/reference/everything.html'>everything</a></span><span class='o'>(</span><span class='o'>)</span></span>
<span>  <span class='o'>)</span></span><span><span class='c'>#&gt; <span style='color: #555555;'># A tibble: 10 × 8</span></span></span>
<span><span class='c'>#&gt;       id code     x_1 x_1_rank    x_2 x_2_rank    x_3 x_3_rank</span></span>
<span><span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 1</span>     1 X     0.395         5 0.977        10 0.822         9</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 2</span>     2 C     0.963        10 0.940         8 0.746         5</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 3</span>     3 E     0.838         9 0.823         6 0.932        10</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 4</span>     4 S     0.582         6 0.847         7 0.502         3</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 5</span>     5 D     0.591         7 0.297         3 0.768         7</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 6</span>     6 N     0.087<span style='text-decoration: underline;'>8</span>        2 0.329         4 0.540         4</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 7</span>     7 K     0.060<span style='text-decoration: underline;'>5</span>        1 0.953         9 0.811         8</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 8</span>     8 V     0.258         4 0.264         2 0.062<span style='text-decoration: underline;'>1</span>        1</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'> 9</span>     9 U     0.660         8 0.335         5 0.747         6</span></span>
<span><span class='c'>#&gt; <span style='color: #555555;'>10</span>    10 M     0.170         3 0.019<span style='text-decoration: underline;'>2</span>        1 0.154         2</span></span></code></pre>

</div>

With that `everything` part there being to capture any columns I forgot about, because `select` will only keep the specified columns.

There are many ways we can do these kinds of things, but to me this felt like a nice fun example.

# End

Anywho, that's a bit of tidyverse magic, maybe it'll be useful for you!

