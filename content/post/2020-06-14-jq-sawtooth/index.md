---
title: 'Just Quickly: Removing Sawtooth Patterns in Line Graphs'
author: Nicholas Tierney
date: '2020-06-14'
slug: jq-sawtooth
categories:
  - rstats
  - data visualisation
tags:
  - rstats
output: hugodown::md_document
rmd_hash: ef66bc842f294e1f

---

Sometimes you come across a plot that looks like the following:

<div class="highlight">

<img src="figs/ozbaby-example-1.png" width="700px" style="display: block; margin: auto;" />

</div>

And you might think:

> Something does not look right but I have no idea what is going on here

And that's OK.

So, what's the problem with the plot, and how do you solve it?

Well, the problem is we have these "sawtooth" patterns in the data, where the data goes up and down.

Typically, we can solve this problem by including some grouping characteristic into the data visualisation.

It is also worth noting that this doesn't always mean a plot is bad - this could actually be the exact type of plot that you might expect to see (for example in a time series with very high periodicity, perhaps).

But, in our case, we need to understand what our data is first, and what we expect. We are looking at [ozbabynames](https://github.com/ropenscilabs/ozbabynames) - the names at birth of people in Australia. So we are plotting the number of names of a person at birth for each year. In our example we can look at the occurrences of the name, "Kim", like so:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>ggplot</span>(<span class='k'>oz_kim</span>,
       <span class='nf'>aes</span>(x = <span class='k'>year</span>,
           y = <span class='k'>count</span>)) <span class='o'>+</span> 
  <span class='nf'>geom_line</span>()
</code></pre>
<img src="figs/oz-kim-1.png" width="700px" style="display: block; margin: auto;" />

</div>

We don't expect the name "kim" to suddenly crash down each year - especially since this looks to be an exact vertical drop.

So what do we do?

This vis problem often means there is some grouping characteristic missing from the graphic. For example, in this case, "sex" is not shown in the data. In showing it, we get:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'><a href='http://colorspace.R-Forge.R-project.org'>colorspace</a></span>)
  <span class='nf'>ggplot</span>(<span class='k'>oz_kim</span>,
         <span class='nf'>aes</span>(x = <span class='k'>year</span>,
             y = <span class='k'>count</span>,
             colour = <span class='k'>sex</span>)) <span class='o'>+</span> 
  <span class='nf'>geom_line</span>() <span class='o'>+</span>
  <span class='nf'><a href='http://colorspace.R-Forge.R-project.org//reference/scale_colour_discrete_qualitative.html'>scale_colour_discrete_qualitative</a></span>()
</code></pre>
<img src="figs/add-sex-1.png" width="700px" style="display: block; margin: auto;" />

</div>

So we see that there is still some sawtooth patterns going on. Let's look at the data to see if there are other variables we are missing:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='k'>oz_kim</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 164 x 5</span></span>
<span class='c'>#&gt;    name  sex     year count state          </span>
<span class='c'>#&gt;    <span style='color: #555555;font-style: italic;'>&lt;chr&gt;</span><span> </span><span style='color: #555555;font-style: italic;'>&lt;chr&gt;</span><span>  </span><span style='color: #555555;font-style: italic;'>&lt;int&gt;</span><span> </span><span style='color: #555555;font-style: italic;'>&lt;int&gt;</span><span> </span><span style='color: #555555;font-style: italic;'>&lt;chr&gt;</span><span>          </span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span><span> Kim   Female  </span><span style='text-decoration: underline;'>2</span><span>017     2 South Australia</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span><span> Kim   Female  </span><span style='text-decoration: underline;'>2</span><span>016     1 South Australia</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span><span> Kim   Female  </span><span style='text-decoration: underline;'>2</span><span>015     1 South Australia</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span><span> Kim   Female  </span><span style='text-decoration: underline;'>2</span><span>014     2 South Australia</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span><span> Kim   Male    </span><span style='text-decoration: underline;'>2</span><span>014     1 South Australia</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span><span> Kim   Female  </span><span style='text-decoration: underline;'>2</span><span>012     1 South Australia</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span><span> Kim   Female  </span><span style='text-decoration: underline;'>2</span><span>011     3 South Australia</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span><span> Kim   Female  </span><span style='text-decoration: underline;'>2</span><span>010     1 South Australia</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span><span> Kim   Female  </span><span style='text-decoration: underline;'>2</span><span>009     1 South Australia</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span><span> Kim   Female  </span><span style='text-decoration: underline;'>2</span><span>008     3 South Australia</span></span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 154 more rows</span></span></code></pre>

</div>

Aha! We can see that there is another grouping characteristic going on: State.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>ggplot</span>(<span class='k'>oz_kim</span>,
       <span class='nf'>aes</span>(x = <span class='k'>year</span>,
           y = <span class='k'>count</span>,
           colour = <span class='k'>sex</span>)) <span class='o'>+</span> 
  <span class='nf'>geom_line</span>() <span class='o'>+</span> 
  <span class='nf'>facet_wrap</span>(<span class='o'>~</span><span class='k'>state</span>) <span class='o'>+</span>
  <span class='nf'><a href='http://colorspace.R-Forge.R-project.org//reference/scale_colour_discrete_qualitative.html'>scale_colour_discrete_qualitative</a></span>()
</code></pre>
<img src="figs/add-sex-state-1.png" width="700px" style="display: block; margin: auto;" />

</div>

So, **how to remove sawtooth patterns in a plot?**

1.  Understand what sort of graphic you are expecting
2.  Explore and potentially include all grouping features into the graphic

