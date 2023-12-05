---
title: Long Error Messages are a Code Smell
author: Nicholas Tierney
date: '2023-12-06'
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
output: hugodown::md_document
rmd_hash: f986f814db59696b

---

<figure>
<img src="img/no-parking.jpg" alt="An amusing “no parking” sign that threatens to transport your car to another universe" />
<figcaption aria-hidden="true">An amusing “no parking” sign that threatens to transport your car to another universe</figcaption>
</figure>

Error messages are really important, and really hard to write well. When software authors take the time to check your inputs, and share reasons why it fails, they are doing you a favour. So, writing error messages is good. However, I think that long error messages, (or warnings, or messages, etc), are a code smell. A code smells is:

> ...any characteristic in the source code of a program that possibly indicates a deeper problem. -- [Wiki article on code smells](https://en.wikipedia.org/wiki/Code_smell)

And the code smell looks like this:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>vector_to_square</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  <span class='nv'>dims</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='nv'>squarable_length</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/Round.html'>floor</a></span><span class='o'>(</span><span class='nv'>dims</span><span class='o'>)</span> <span class='o'>==</span> <span class='nv'>dims</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='o'>!</span><span class='nv'>squarable_length</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector is not of a squarable length"</span>,</span>
<span>        <span class='s'>"&#123;.var x&#125; is of length &#123;.num &#123;length(x)&#125;&#125;"</span>,</span>
<span>        <span class='s'>"This cannot be represented as a square"</span>,</span>
<span>        <span class='s'>"Square root of &#123;.var dim(x)&#125; is: &#123;.num &#123;round(dims, 3)&#125;&#125;."</span></span>
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

I think there is a problem when writing long error messages straight into the body of a function. I believe wrapping error functions into a function reduces repetition, and makes the code clearer to understand, since you don't need to wade through error checking code.

## What's the solution?

I think that the better option is to write `check_*` functions. These could be something like, `check_array_is_square(x)`.

## Show me the difference

To demonstrate this, let's take a look again at the example code from above, a function that converts a vector into a square matrix:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>vector_to_square</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  <span class='nv'>dims</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='nv'>squarable_length</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/Round.html'>floor</a></span><span class='o'>(</span><span class='nv'>dims</span><span class='o'>)</span> <span class='o'>==</span> <span class='nv'>dims</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='o'>!</span><span class='nv'>squarable_length</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector is not of a squarable length"</span>,</span>
<span>        <span class='s'>"&#123;.var x&#125; is of length &#123;.num &#123;length(x)&#125;&#125;"</span>,</span>
<span>        <span class='s'>"This cannot be represented as a square"</span>,</span>
<span>        <span class='s'>"Square root of &#123;.var dim(x)&#125; is: &#123;.num &#123;round(dims, 3)&#125;&#125;."</span></span>
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

Now let's demonstrate its use, and what the errors look like:

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

So we have some defensive code (code that checks inputs), which does two things:

-   checks if the inputs can be a square matrix
-   checks if inputs are not numeric

While we like these error messages, wrapping up the error messags as functions clarifies the intent of them, and avoids repitition:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>check_if_squarable</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  <span class='nv'>dims</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='nv'>squarable_length</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/Round.html'>floor</a></span><span class='o'>(</span><span class='nv'>dims</span><span class='o'>)</span> <span class='o'>==</span> <span class='nv'>dims</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='o'>!</span><span class='nv'>squarable_length</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector is not of a squarable length"</span>,</span>
<span>        <span class='s'>"&#123;.var x&#125; is of length &#123;.num &#123;length(x)&#125;&#125;"</span>,</span>
<span>        <span class='s'>"This cannot be represented as a square"</span>,</span>
<span>        <span class='s'>"Square root of &#123;.var dim(x)&#125; is: &#123;.num &#123;round(dims, 3)&#125;&#125;."</span></span>
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

I think that the main benefit here is *readability* - I don't have to read some error checking code, I can read the function name `check_if_squarable(x)` - this tells me it checks if it is squareable, and `check_if_not_numeric(x)` does what it says on the tin. This means I don't have to spend time reading error checking code, which can sometimes be quite complex. I can focus on those functions intention. This means I can summarise this function like so:

-   Do some checking of the inputs

-   Get the dimensions for a square matrix

-   Make the matrix, filling in by row

There are a couple of other benefits to this:

-   All my checking functions can get re-used in the other work I do
-   I can find common cases of checking and improve them.
-   I see two checking functions, and it invites me to think about other checks that I might want to perform.

# On writing error messages

I do think that writing good error messages is hard. Something that helped me think about this differently was something from the [tidyverse style guide on error messages](https://style.tidyverse.org/error-messages.html):

> An error message should start with a general statement of the problem then give a concise description of what went wrong. Consistent use of punctuation and formatting makes errors easier to parse

They also recommend using [`cli::cli_abort()`](https://cli.r-lib.org/reference/cli_abort.html), which we used above. I've really enjoyed using this over `stop`, because, well, again, the tidyverse team summarises the reasons well, [`cli::cli_abort()`](https://cli.r-lib.org/reference/cli_abort.html) is good because it:

> -   Makes it easy to generate bulleted lists.
>
> -   Uses glue style interpolation to insert data into the error.
>
> -   Supports a wide range of [inline markup](https://cli.r-lib.org/reference/inline-markup.html).
>
> -   Provides convenient tools to [chain errors together](https://rlang.r-lib.org/reference/topic-error-chaining.html).
>
> -   Can control the [name of the function](https://rlang.r-lib.org/reference/topic-error-call.html) shown in the error.

You should read the [whole section on error messages](https://style.tidyverse.org/error-messages.html), they've got great advice on how to write good error messages.

# Functions are good

I will wrap up by emphasising a point about using functions. **Good functions can be individually reasoned with**, and used repeatedly across your work. This means you can write them once, use them many times, and only need to make changes to one place, rather than in many. Writing functions helps abstract away details, and helps clarify your code. They are an important building block for writing good code that has content that can be easily reasoned with, and extended.

# End

That's pretty much all I wanted to say on this.

