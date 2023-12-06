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
rmd_hash: 11c0a0eece8ef418

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

# Thanks

Thank you very much to the maintainers of [`cli`](https://cli.r-lib.org/index.html) ([Gábor Csárdi](https://github.com/gaborcsardi)), and [`rlang`](https://rlang.r-lib.org/) ([Lionel Henry](https://github.com/lionel-)), and the team behind these packages. These

# End

That's pretty much all I wanted to say on this.

# PS: Let's actually check the error messages

My friend [Adam Gruer](https://adamgruer.rbind.io/) read this blog post and asked:

> might a user get confused when then the error is reported as coming from the check_function rather than the function they called? What are your thoughts?

And that made me realise that I hadn't looked at the error messages again, and it also made me realise I've seen this problem before and been unsure how to solve it. This prompted more than a shallow dive into error code, which I think is worthwhile unpacking.

OK, err, so, how do these error messages look?

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'>vector_to_square</span><span class='o'>(</span><span class='m'>1</span><span class='o'>:</span><span class='m'>4</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;      [,1] [,2]</span></span>
<span><span class='c'>#&gt; [1,]    1    2</span></span>
<span><span class='c'>#&gt; [2,]    3    4</span></span>
<span></span><span><span class='nf'>vector_to_square</span><span class='o'>(</span><span class='m'>1</span><span class='o'>:</span><span class='m'>5</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00; font-weight: bold;'>Error</span><span style='font-weight: bold;'> in `check_if_squarable()`:</span></span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00;'>!</span> Provided vector is not of a squarable length</span></span>
<span><span class='c'>#&gt; `x` is of length 5</span></span>
<span><span class='c'>#&gt; This cannot be represented as a square</span></span>
<span><span class='c'>#&gt; Square root of `dim(x)` is: 2.236.</span></span>
<span></span><span><span class='nf'>vector_to_square</span><span class='o'>(</span><span class='nv'>LETTERS</span><span class='o'>[</span><span class='m'>1</span><span class='o'>:</span><span class='m'>4</span><span class='o'>]</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00; font-weight: bold;'>Error</span><span style='font-weight: bold;'> in `check_if_not_numeric()`:</span></span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00;'>!</span> Provided vector must be <span style='color: #0000BB;'>&lt;numeric&gt;</span>, not <span style='color: #0000BB;'>&lt;character&gt;</span></span></span>
<span><span class='c'>#&gt; We see that `is.numeric(x)` returns <span style='color: #0000BB;'>&lt;character&gt;</span></span></span>
<span></span></code></pre>

</div>

In the error messages, the error is about the checking function, `check_if_squarable()` or `check_if_not_numeric()`. This could be confusing to the user as the error doesn't appear to be coming from the `vector_to_square()` function. Thankfully, the rlang team has thought about this, and they have a helpful vignette, ["Including function calls in error messages"](https://rlang.r-lib.org/reference/topic-error-call.html).

They bring up two really great points. Both of which I hadn't really thought about solving. I just thought this was the compromise.

The first is how to make sure the input checking function references the function it was called in. This is why we see the error being about `check_if_squarable()`, and not `vector_to_square()`, which is what we want.

This is solved by passing the `check_` functions a `call` environment. So we add `call = rlang::caller_env()` as an argument:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>check_if_squarable</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span>, </span>
<span>                               <span class='c'># Add this</span></span>
<span>                               <span class='nv'>call</span> <span class='o'>=</span> <span class='nf'>rlang</span><span class='nf'>::</span><span class='nf'><a href='https://rlang.r-lib.org/reference/stack.html'>caller_env</a></span><span class='o'>(</span><span class='o'>)</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  <span class='nv'>dims</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='nv'>squarable_length</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/Round.html'>floor</a></span><span class='o'>(</span><span class='nv'>dims</span><span class='o'>)</span> <span class='o'>==</span> <span class='nv'>dims</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='o'>!</span><span class='nv'>squarable_length</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      message <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector is not of a squarable length"</span>,</span>
<span>        <span class='s'>"&#123;.var x&#125; is of length &#123;.num &#123;length(x)&#125;&#125;"</span>,</span>
<span>        <span class='s'>"This cannot be represented as a square"</span>,</span>
<span>        <span class='s'>"Square root of &#123;.var dim(x)&#125; is: &#123;.num &#123;round(dims, 3)&#125;&#125;."</span></span>
<span>        <span class='o'>)</span>,</span>
<span>      <span class='c'># And this</span></span>
<span>      call <span class='o'>=</span> <span class='nv'>call</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>&#125;</span></span>
<span><span class='o'>&#125;</span></span>
<span></span>
<span><span class='nv'>check_if_not_numeric</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span>, </span>
<span>                                 <span class='c'># Add this</span></span>
<span>                                 <span class='nv'>call</span> <span class='o'>=</span> <span class='nf'>rlang</span><span class='nf'>::</span><span class='nf'><a href='https://rlang.r-lib.org/reference/stack.html'>caller_env</a></span><span class='o'>(</span><span class='o'>)</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='o'>!</span><span class='nf'><a href='https://rdrr.io/r/base/numeric.html'>is.numeric</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span> <span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      message <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector must be &#123;.cls numeric&#125;, not &#123;.cls &#123;class(x)&#125;&#125;"</span>,</span>
<span>        <span class='s'>"We see that &#123;.run is.numeric(x)&#125; returns &#123;.cls &#123;class(x)&#125;&#125;"</span></span>
<span>      <span class='o'>)</span>,</span>
<span>      <span class='c'># And this</span></span>
<span>      call <span class='o'>=</span> <span class='nv'>call</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>&#125;</span></span>
<span>  </span>
<span><span class='o'>&#125;</span></span></code></pre>

</div>

Let's see what that looks like now:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'>vector_to_square</span><span class='o'>(</span><span class='m'>1</span><span class='o'>:</span><span class='m'>4</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;      [,1] [,2]</span></span>
<span><span class='c'>#&gt; [1,]    1    2</span></span>
<span><span class='c'>#&gt; [2,]    3    4</span></span>
<span></span><span><span class='nf'>vector_to_square</span><span class='o'>(</span><span class='m'>1</span><span class='o'>:</span><span class='m'>5</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00; font-weight: bold;'>Error</span><span style='font-weight: bold;'> in `vector_to_square()`:</span></span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00;'>!</span> Provided vector is not of a squarable length</span></span>
<span><span class='c'>#&gt; `x` is of length 5</span></span>
<span><span class='c'>#&gt; This cannot be represented as a square</span></span>
<span><span class='c'>#&gt; Square root of `dim(x)` is: 2.236.</span></span>
<span></span><span><span class='nf'>vector_to_square</span><span class='o'>(</span><span class='nv'>LETTERS</span><span class='o'>[</span><span class='m'>1</span><span class='o'>:</span><span class='m'>4</span><span class='o'>]</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00; font-weight: bold;'>Error</span><span style='font-weight: bold;'> in `vector_to_square()`:</span></span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00;'>!</span> Provided vector must be <span style='color: #0000BB;'>&lt;numeric&gt;</span>, not <span style='color: #0000BB;'>&lt;character&gt;</span></span></span>
<span><span class='c'>#&gt; We see that `is.numeric(x)` returns <span style='color: #0000BB;'>&lt;character&gt;</span></span></span>
<span></span></code></pre>

</div>

OK much better, the error now starts with: "Error in `vector_to_square()`".

Thanks [Adam Gruer](https://adamgruer.rbind.io/) again for pointing this out!

The second problem, which I hadn't really considered when writing this, but was familiar, is to do with supplying argument names. We don't notice this in our case, since `vector_to_square()` and our `check_*` functions use `x` as the argument.

To demonstrate the problem, let's change the arguments of `vector_to_square` from `x` to `data`, and leave `check_if_squarable()` and co as is.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>vector_to_square</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>data</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  </span>
<span>  <span class='nf'>check_if_squarable</span><span class='o'>(</span><span class='nv'>data</span><span class='o'>)</span></span>
<span>  <span class='nf'>check_if_not_numeric</span><span class='o'>(</span><span class='nv'>data</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='nv'>dims</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='o'>(</span><span class='nv'>data</span><span class='o'>)</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='nf'><a href='https://rdrr.io/r/base/matrix.html'>matrix</a></span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>data</span>,</span>
<span>         nrow <span class='o'>=</span> <span class='nv'>dims</span>,</span>
<span>         ncol <span class='o'>=</span> <span class='nv'>dims</span>,</span>
<span>         byrow <span class='o'>=</span> <span class='kc'>TRUE</span><span class='o'>)</span></span>
<span><span class='o'>&#125;</span></span></code></pre>

</div>

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'>vector_to_square</span><span class='o'>(</span>data <span class='o'>=</span> <span class='m'>1</span><span class='o'>:</span><span class='m'>4</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;      [,1] [,2]</span></span>
<span><span class='c'>#&gt; [1,]    1    2</span></span>
<span><span class='c'>#&gt; [2,]    3    4</span></span>
<span></span><span><span class='nf'>vector_to_square</span><span class='o'>(</span>data <span class='o'>=</span> <span class='m'>1</span><span class='o'>:</span><span class='m'>5</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00; font-weight: bold;'>Error</span><span style='font-weight: bold;'> in `vector_to_square()`:</span></span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00;'>!</span> Provided vector is not of a squarable length</span></span>
<span><span class='c'>#&gt; `x` is of length 5</span></span>
<span><span class='c'>#&gt; This cannot be represented as a square</span></span>
<span><span class='c'>#&gt; Square root of `dim(x)` is: 2.236.</span></span>
<span></span><span><span class='nf'>vector_to_square</span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>LETTERS</span><span class='o'>[</span><span class='m'>1</span><span class='o'>:</span><span class='m'>4</span><span class='o'>]</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00; font-weight: bold;'>Error</span><span style='font-weight: bold;'> in `vector_to_square()`:</span></span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00;'>!</span> Provided vector must be <span style='color: #0000BB;'>&lt;numeric&gt;</span>, not <span style='color: #0000BB;'>&lt;character&gt;</span></span></span>
<span><span class='c'>#&gt; We see that `is.numeric(x)` returns <span style='color: #0000BB;'>&lt;character&gt;</span></span></span>
<span></span></code></pre>

</div>

Now we only see `x` mentioned in the errors, not `data`, which is what was supplied in `vector_to_square`.

A small detail, perhaps? But I think it's actually kind of a big problem! I don't want my users to have to try and reckon/think about what `x` is, or have to imagine that I've written some fancy code checking functions. I just want them to know where the error came from.

In more complex functions this could be even more confusing. Thankfully, yet again, the rlang team has thought about this, in [Input checkers and `caller_arg()`](https://rlang.r-lib.org/reference/topic-error-call.html#input-checkers-and-caller-arg-), they suggest adding `arg = rlang::caller_arg(x)` to our checking function, which helps capture the name of the argument that is passed to the check functions. Super neat.

Let's demonstrate this - we change uses of `x` to using `arg` in the error message.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nv'>check_if_squarable</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span>, </span>
<span>                               <span class='nv'>arg</span> <span class='o'>=</span> <span class='nf'>rlang</span><span class='nf'>::</span><span class='nf'><a href='https://rlang.r-lib.org/reference/caller_arg.html'>caller_arg</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span>,</span>
<span>                               <span class='nv'>call</span> <span class='o'>=</span> <span class='nf'>rlang</span><span class='nf'>::</span><span class='nf'><a href='https://rlang.r-lib.org/reference/stack.html'>caller_env</a></span><span class='o'>(</span><span class='o'>)</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  </span>
<span>  <span class='nv'>x_len</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/length.html'>length</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span></span>
<span>  <span class='nv'>dims</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/MathFun.html'>sqrt</a></span><span class='o'>(</span><span class='nv'>x_len</span><span class='o'>)</span></span>
<span>  </span>
<span>  <span class='nv'>squarable_length</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/Round.html'>floor</a></span><span class='o'>(</span><span class='nv'>dims</span><span class='o'>)</span> <span class='o'>==</span> <span class='nv'>dims</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='o'>!</span><span class='nv'>squarable_length</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      message <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector is not of a squarable length"</span>,</span>
<span>        <span class='s'>"&#123;.arg &#123;arg&#125;&#125; is of length &#123;.num &#123;x_len&#125;&#125;"</span>,</span>
<span>        <span class='s'>"This cannot be represented as a square"</span></span>
<span>        <span class='o'>)</span>,</span>
<span>      call <span class='o'>=</span> <span class='nv'>call</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>&#125;</span></span>
<span><span class='o'>&#125;</span></span>
<span></span>
<span><span class='nv'>check_if_not_numeric</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>x</span>, </span>
<span>                                 <span class='nv'>arg</span> <span class='o'>=</span> <span class='nf'>rlang</span><span class='nf'>::</span><span class='nf'><a href='https://rlang.r-lib.org/reference/caller_arg.html'>caller_arg</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span>,</span>
<span>                                 <span class='nv'>call</span> <span class='o'>=</span> <span class='nf'>rlang</span><span class='nf'>::</span><span class='nf'><a href='https://rlang.r-lib.org/reference/stack.html'>caller_env</a></span><span class='o'>(</span><span class='o'>)</span><span class='o'>)</span><span class='o'>&#123;</span></span>
<span>  </span>
<span>  <span class='kr'>if</span> <span class='o'>(</span><span class='o'>!</span><span class='nf'><a href='https://rdrr.io/r/base/numeric.html'>is.numeric</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span> <span class='o'>&#123;</span></span>
<span>    <span class='nf'>cli</span><span class='nf'>::</span><span class='nf'><a href='https://cli.r-lib.org/reference/cli_abort.html'>cli_abort</a></span><span class='o'>(</span></span>
<span>      message <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span></span>
<span>        <span class='s'>"Provided vector, &#123;.arg &#123;arg&#125;&#125;, must be &#123;.cls numeric&#125;, not &#123;.cls &#123;class(x)&#125;&#125;"</span>,</span>
<span>        <span class='s'>"We see that &#123;.run is.numeric(&#123;.arg &#123;arg&#125;&#125;)&#125; returns &#123;.cls &#123;class(x)&#125;&#125;"</span></span>
<span>      <span class='o'>)</span>,</span>
<span>      call <span class='o'>=</span> <span class='nv'>call</span></span>
<span>    <span class='o'>)</span></span>
<span>  <span class='o'>&#125;</span></span>
<span>  </span>
<span><span class='o'>&#125;</span></span></code></pre>

</div>

(in demo-ing this, I realised I don't need to include the square root of the length to explain to the user, so I deleted that part of the message)

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span><span class='nf'>vector_to_square</span><span class='o'>(</span>data <span class='o'>=</span> <span class='m'>1</span><span class='o'>:</span><span class='m'>4</span><span class='o'>)</span></span>
<span><span class='c'>#&gt;      [,1] [,2]</span></span>
<span><span class='c'>#&gt; [1,]    1    2</span></span>
<span><span class='c'>#&gt; [2,]    3    4</span></span>
<span></span><span><span class='nf'>vector_to_square</span><span class='o'>(</span>data <span class='o'>=</span> <span class='m'>1</span><span class='o'>:</span><span class='m'>5</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00; font-weight: bold;'>Error</span><span style='font-weight: bold;'> in `vector_to_square()`:</span></span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00;'>!</span> Provided vector is not of a squarable length</span></span>
<span><span class='c'>#&gt; `data` is of length 5</span></span>
<span><span class='c'>#&gt; This cannot be represented as a square</span></span>
<span></span><span><span class='nf'>vector_to_square</span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>LETTERS</span><span class='o'>[</span><span class='m'>1</span><span class='o'>:</span><span class='m'>4</span><span class='o'>]</span><span class='o'>)</span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00; font-weight: bold;'>Error</span><span style='font-weight: bold;'> in `vector_to_square()`:</span></span></span>
<span><span class='c'>#&gt; <span style='color: #BBBB00;'>!</span> Provided vector, `data`, must be <span style='color: #0000BB;'>&lt;numeric&gt;</span>, not <span style='color: #0000BB;'>&lt;character&gt;</span></span></span>
<span><span class='c'>#&gt; We see that `` is.numeric(`data`) `` returns <span style='color: #0000BB;'>&lt;character&gt;</span></span></span>
<span></span></code></pre>

</div>

This now uses `data` instead of `x`, which is what we want!

In writing this blog post I've now realised that my pattern of creating `check_` functions for input checking now also results in a potentially undesirable error for the user where they won't know where the error has come from. I'll need to make a note to change this in a lot of my packages.

In the future I'd really like to dive a bit deeper into adding classes or subclasses of errors and understand the benefits. There's a [tidyverse design page on error constructors](https://design.tidyverse.org/err-constructor.html) that I should read more deeply. I'm pretty sure this would be helpful in things like [`greta`](https://greta-stats.org/).

