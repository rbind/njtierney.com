---
title: 'Just Quickly: Rowwise Median in Dplyr'
author: Nicholas Tierney
date: '2020-06-08'
slug: jq-rowwise-median
categories:
  - rstats
tags:
  - rstats
  - dplyr
output: 
  hugodown::md_document:
    tidyverse_style: false
rmd_hash: 48762cb7c0366caa

---

The [tidyverse](https://tidyverse.org/) team recently completed a 1.0.0 release for `dplyr`, which was a pretty big deal, and it included a bunch of new features. One of the things that I really enjoyed was that they wrote a [series of blog posts](https://www.tidyverse.org/blog/2020/06/dplyr-1-0-0/) describing new features in the release. This was great, because we got to see what was coming up, and great because people tried them out and gave them feedback. Then, the tidyverse listened, and changed behaviour based on feedback from the community.

Isn't that great?

Let's celebrate something from the tidyverse today: `rowwise`. This function has actually been around for a while, but I never really used it, for some reason. A student recently had an issue where they had data like this:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>library</span>(<span class='k'><a href='https://tidyverse.tidyverse.org/reference'>tidyverse</a></span>)
<span class='c'>#&gt; -- Attaching packages ------------------------------------------------- tidyverse 1.3.0 --</span>
<span class='c'>#&gt; v ggplot2 3.3.1     v purrr   0.3.4</span>
<span class='c'>#&gt; v tibble  3.0.1     v dplyr   1.0.0</span>
<span class='c'>#&gt; v tidyr   1.1.0     v stringr 1.4.0</span>
<span class='c'>#&gt; v readr   1.3.1     v forcats 0.5.0</span>
<span class='c'>#&gt; -- Conflicts ---------------------------------------------------- tidyverse_conflicts() --</span>
<span class='c'>#&gt; x dplyr::filter() masks stats::filter()</span>
<span class='c'>#&gt; x dplyr::lag()    masks stats::lag()</span>
<span class='k'>income</span> <span class='o'>&lt;-</span> <span class='nf'>tibble</span>(income_range = <span class='nf'>c</span>(<span class='s'>"0-74"</span>,
                                  <span class='s'>"75-145"</span>,
                                  <span class='s'>"150-325"</span>,
                                  <span class='s'>"325+"</span>),
                 count = <span class='nf'>c</span>(<span class='m'>125</span>,
                           <span class='m'>170</span>, 
                           <span class='m'>215</span>,
                           <span class='m'>250</span>))

<span class='k'>income</span>
<span class='c'>#&gt; # A tibble: 4 x 2</span>
<span class='c'>#&gt;   income_range count</span>
<span class='c'>#&gt;   &lt;chr&gt;        &lt;dbl&gt;</span>
<span class='c'>#&gt; 1 0-74           125</span>
<span class='c'>#&gt; 2 75-145         170</span>
<span class='c'>#&gt; 3 150-325        215</span>
<span class='c'>#&gt; 4 325+           250</span></code></pre>

</div>

They wanted to calculate the median of `income_range`.

This presents an interesting problem, with a few steps:

1.  Separate the range values into two columns.
2.  Calculate the median of each of those pairs of numbers.

We can get ourselves into a better position by separating out `income_range` into two columns, `lower`, and `upper`, and converting the contents. `separate` is kind of magic here, and while you can specify a specific thing that separates the numbers, `separate` has a nice bit og magic that just finds the most likely character to separate on.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>income_sep</span> <span class='o'>&lt;-</span> <span class='k'>income</span> <span class='o'>%&gt;%</span> 
  <span class='nf'>separate</span>(col = <span class='k'>income_range</span>,
           into = <span class='nf'>c</span>(<span class='s'>"lower"</span>, <span class='s'>"upper"</span>),
           convert = <span class='m'>TRUE</span>)
<span class='k'>income_sep</span>
<span class='c'>#&gt; # A tibble: 4 x 3</span>
<span class='c'>#&gt;   lower upper count</span>
<span class='c'>#&gt;   &lt;int&gt; &lt;int&gt; &lt;dbl&gt;</span>
<span class='c'>#&gt; 1     0    74   125</span>
<span class='c'>#&gt; 2    75   145   170</span>
<span class='c'>#&gt; 3   150   325   215</span>
<span class='c'>#&gt; 4   325    NA   250</span></code></pre>

</div>

So now have a lower and an upper range of values, and we want to calculate the median of these.

This...gets a little bit tricky.

At first instinct, you might try something like this:

> calculate the median based on the lower and upper columns:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>income_sep</span> <span class='o'>%&gt;%</span> 
  <span class='nf'>mutate</span>(med = <span class='nf'>median</span>(<span class='nf'>c</span>(<span class='k'>lower</span>, <span class='k'>upper</span>), na.rm = <span class='m'>TRUE</span>))
<span class='c'>#&gt; # A tibble: 4 x 4</span>
<span class='c'>#&gt;   lower upper count   med</span>
<span class='c'>#&gt;   &lt;int&gt; &lt;int&gt; &lt;dbl&gt; &lt;int&gt;</span>
<span class='c'>#&gt; 1     0    74   125   145</span>
<span class='c'>#&gt; 2    75   145   170   145</span>
<span class='c'>#&gt; 3   150   325   215   145</span>
<span class='c'>#&gt; 4   325    NA   250   145</span></code></pre>

</div>

But this doesn't give us what we want. It just gives us the median of the vector, I think?

Anyway, how do we solve this?

`rowwise()`:

We can now call `rowwise()` and calculate the median based on the `lower` and `upper`, and it will consider each row and take the median of those two numbers:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>income_sep</span> <span class='o'>%&gt;%</span> 
  <span class='nf'>rowwise</span>() <span class='o'>%&gt;%</span> 
  <span class='nf'>mutate</span>(med = <span class='nf'>median</span>(<span class='nf'>c</span>(<span class='k'>lower</span>, <span class='k'>upper</span>), na.rm = <span class='m'>TRUE</span>))
<span class='c'>#&gt; # A tibble: 4 x 4</span>
<span class='c'>#&gt; # Rowwise: </span>
<span class='c'>#&gt;   lower upper count   med</span>
<span class='c'>#&gt;   &lt;int&gt; &lt;int&gt; &lt;dbl&gt; &lt;dbl&gt;</span>
<span class='c'>#&gt; 1     0    74   125   37 </span>
<span class='c'>#&gt; 2    75   145   170  110 </span>
<span class='c'>#&gt; 3   150   325   215  238.</span>
<span class='c'>#&gt; 4   325    NA   250  325</span></code></pre>

</div>

And I think that's pretty neat.

Thanks, tidyverse team!

