---
title: "Magically build things on the cloud"
author: ''
date: '2018-11-08'
slug: magic-tic
draft: true
categories:
  - rstats
tags: []
---

# Notes on getting started with bookdown and travis CI

I wanted my bookdown book to not generate HTML content that I had to track with git because that would BLOAT out the repo.

Not sure why you want to do that? Well, imagine that you are 
working on a book, or a package, and as part of that process, you always do this one thing that's the same - you click "build book", or you "render" the book, or perhaps you run `pkgdown::build_site()`. Then, you push quite possible >100 HTML files. 

Now, sure, that's a nice green stamp on your github heatmap, and hey, I'm a fan of that, it's a nice way to measure productibity. But also, we don't really care that much about the HTML - this is not something we wrote, we have some code to write it for us, so that we don't have to. In short, there are three downsides to committing the HTML every time:

1. You probably don't care about the content - so why bother version controlling it? We use version control to manage our content. I can not think of one single time where I have gone back to my repo and found some HTML code that broke everything.

2. holding all that HTML code in your repo on your computer is going to make it an enourmous git repo - say you have 5Mb of content generated with each HTML document section, and you build it like, 100 times - the diff of that, the bit that git stores and version controls - could be largish, maybe 250Mb. Sure, I've got apps on my phone that are bigger, but we should try and stay lean where possible.

3. It's repetitive, and we could forget to do something, so, let's get the machine to do it for us.

And a bonus reason, you get some ninja hacker points.

So, to recap. Here is what we want.

1. To not click "build" and push a bunch of HTML to git
2. Once we push things to github, it should trigger a build, as if we ran the command "pkgdown::build_site", or "bookdown::render()".

That, sounds like a thing, right?

Well, it is. 

The `tic` package helps with exactly this. Now there are other ways of doing this, but `tic` takes away the headaches and the minutae of the details. Plus it's written by the great coder Kurill Mueller. Who you might remember from packages "tibble", "dplyr", "tidyr", and my personal favourite, "here", and more! He's a key contributor, and a really nice guy.

So, this is a blog post that describes how I went about setting up `tic()`, so that I can simply push my changes to the repo, and not "build", and then push a TON of HTML and whatnot.

For the sakes of the blogpost, Let's assume you have a [standard bookdown template setup]().

Not sure how to get started with that? Well, rstudio have made it super easy.

<screenshot>

Then, you need to do the following:

`usethis::use_travis()`

This sets up this thing called [travis](), which is basically this magic robot in the cloud that runs things once you push them to the cloud. If you haven't heard of travis, you should definitely check our [Julia Silge's blog post on setting up travis]().

To provide special instructions to the robot in the sky (travis). To help with this, we can use the [`travis` R package](), provided by rOpenSci.

so, type this:

`travis::use_tic()`

So, from here we need to create the `tic` file, which helps make sure that everything runs OK in the cloud.

Then, run 

```
tic::tic()
```

This will then tell you that you need to set up a DESCRIPTION file.

To do this, run

```
usethis::use_description
```

And then change it to something as minimal as:

https://github.com/krlmlr/tic.bookdown/blob/master/DESCRIPTION

Then make sure you add the `git2r` package to be installed around line six of travis.yml

< gif >

And you're neeaerly done.

From here, you need to change up your settings on your github repo - and tell it to build from github pages.

<screenshot>

And that's it!

Now, I realise that this is a lot of steps, so I decided to create a little screencast of me doing this. You can find it, [here]().

# Thankyous

This post would not have been possible without the great help of the awesome [Maëlle Salmon](). Maëlle was a huge help to getting me up and running with `tic`, thanks!

Finally, of course, thank you to Kuril for wtiting `tic`.

