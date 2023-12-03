---
title: Long Error Messages are a Code Smell
author: Nicholas Tierney
date: '2023-12-04'
slug: long-errors-smell
categories:
  - error
  - functions
  - rbloggers
  - research software engineer
  - rstats
tags:
  - data science
  - error
  - functions
  - rbloggers
  - research software engineer
  - rstats
draft: yes
output: hugodown::md_document
rmd_hash: 033471536d16c00b

---

Error messages are really important, and really hard to write well. When software authors take the time to check your inputs and share reasons why it fails, they are doing you a favour. However, I think that there is an opportunity when writing this code, for you to wrap these error messages up into a function.

My reasoning is that the code that does the checking can sometimes take up a decent amount of the function body code. And this distracts from the intention of the function.

Functions should be able to be individually reasoned with, and so including large sections of code that contain error checking distracts from the parts of the function that you might want to reason with.

## What's the solution?

I think that the better option is to write `check_*` functions. These could be something like, `check_array_is_square(x)`.

## Show me the difference

To demonstrate this, let's take some example code that converts a vector into a square matrix

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>vector_to_square</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  <span class='nv'>dims</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/Round.html'>floor</a></span><span class='o'>(</span><span class='nv'>dims</span><span class='o'>)</span> <span class='o'>!=</span> <span class='nv'>dims</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector is not of a squarable length"</span>,</span>
<span>        <span class='s'>"&#123;.var x&#125; is of length &#123;.num &#123;length(x)&#125;&#125;"</span>,</span>
<span>        <span class='s'>"This cannot be represented as a square"</span>,</span>
<span>        <span class='s'>"Square root of &#123;.var dim(x)&#125; is: &#123;.num &#123;round(sqrt(length(x)), 3)&#125;&#125;."</span></span>
<span>        <span class='o'>)</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>&#125;</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='o'>!</span><span class='nf'><a href='https://rdrr.io/r/base/numeric.html'>is.numeric</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span> <span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector must be &#123;.cls numeric&#125;, not &#123;.cls &#123;class(x)&#125;&#125;"</span>,</span>
<span>        <span class='s'>"We see that &#123;.run is.numeric(x)&#125; returns &#123;.cls &#123;class(x)&#125;&#125;"</span></span>
<span>      <span class='o'>)</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>&#125;</span></span>
<span>  </span>
<span>  <span class='nf'><a href='https://rdrr.io/r/base/matrix.html'>matrix</a></span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>x</span>,</span>
<span>         nrow <span class='o'>=</span> <span class='nv'>dims</span>,</span>
<span>         ncol <span class='o'>=</span> <span class='nv'>dims</span>,</span>
<span>         byrow <span class='o'>=</span> <span class='kc'>TRUE</span><span class='o'>)</span></span>
<span><span class='o'>&#125;</span></span></code></pre>

</div>

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'>vector_to_square</span><span class='o'>(</span><span class='m'>1</span><span class='o'>:</span><span class='m'>9</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;      [,1] [,2] [,3]</span></span>
<span><span class='c'>#&gt; [1,]    1    2    3</span></span>
<span><span class='c'>#&gt; [2,]    4    5    6</span></span>
<span><span class='c'>#&gt; [3,]    7    8    9</span></span>
<span></span><span><span class='nf'>vector_to_square</span><span class='o'>(</span><span class='m'>1</span><span class='o'>:</span><span class='m'>10</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00; font-weight: bold;'>Error</span><span style='font-weight: bold;'> in `vector_to_square()`:</span></span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00;'>!</span> Provided vector is not of a squarable length</span></span>
<span><span class='c'>#&gt; `x` is of length 10</span></span>
<span><span class='c'>#&gt; This cannot be represented as a square</span></span>
<span><span class='c'>#&gt; Square root of `dim(x)` is: 3.162.</span></span>
<span></span><span><span class='nf'>vector_to_square</span><span class='o'>(</span><span class='nv'>LETTERS</span><span class='o'>[</span><span class='m'>1</span><span class='o'>:</span><span class='m'>4</span><span class='o'>]</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00; font-weight: bold;'>Error</span><span style='font-weight: bold;'> in `vector_to_square()`:</span></span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00;'>!</span> Provided vector must be <span style='color: #0000BB;'>&lt;numeric&gt;</span>, not <span style='color: #0000BB;'>&lt;character&gt;</span></span></span>
<span><span class='c'>#&gt; We see that `is.numeric(x)` returns <span style='color: #0000BB;'>&lt;character&gt;</span></span></span>
<span></span></code></pre>

</div>

Here we have some defensive code that does two things:

-   checks if the inputs can be a square matrix
-   checks if inputs are not numeric

While it is good to have these defensive error messages, I think it's better to write these up as separate functions. Like so:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span></span>
<span><span class='nv'>check_if_squarable</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  </span>
<span>  <span class='nv'>dims</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/Round.html'>floor</a></span><span class='o'>(</span><span class='nv'>dims</span><span class='o'>)</span> <span class='o'>!=</span> <span class='nv'>dims</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector is not of a squarable length"</span>,</span>
<span>        <span class='s'>"&#123;.var x&#125; is of length &#123;.num &#123;length(x)&#125;&#125;"</span>,</span>
<span>        <span class='s'>"This cannot be represented as a square"</span>,</span>
<span>        <span class='s'>"Square root of &#123;.var dim(x)&#125; is: &#123;.num &#123;round(sqrt(length(x)), 3)&#125;&#125;."</span></span>
<span>        <span class='o'>)</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>&#125;</span></span>
<span><span class='o'>&#125;</span></span>
<span></span>
<span><span class='nv'>check_if_not_numeric</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='o'>!</span><span class='nf'><a href='https://rdrr.io/r/base/numeric.html'>is.numeric</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span> <span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector must be &#123;.cls numeric&#125;, not &#123;.cls &#123;class(x)&#125;&#125;"</span>,</span>
<span>        <span class='s'>"We see that &#123;.run is.numeric(x)&#125; returns &#123;.cls &#123;class(x)&#125;&#125;"</span></span>
<span>      <span class='o'>)</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>&#125;</span></span>
<span>  </span>
<span><span class='o'>&#125;</span></span></code></pre>

</div>

These error messages can then be put into the function like so:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>vector_to_square</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  </span>
<span>  <span class='nf'>check_if_squarable</span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span></span>
<span>  <span class='nf'>check_if_not_numeric</span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='nv'>dims</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='nf'><a href='https://rdrr.io/r/base/matrix.html'>matrix</a></span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>x</span>,</span>
<span>         nrow <span class='o'>=</span> <span class='nv'>dims</span>,</span>
<span>         ncol <span class='o'>=</span> <span class='nv'>dims</span>,</span>
<span>         byrow <span class='o'>=</span> <span class='kc'>TRUE</span><span class='o'>)</span></span>
<span><span class='o'>&#125;</span></span></code></pre>

</div>

