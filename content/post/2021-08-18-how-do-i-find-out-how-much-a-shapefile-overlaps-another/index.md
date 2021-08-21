---
title: How Much One Shapefile Overlaps Another?
author: Nicholas Tierney
date: '2021-08-21'
slug: how-much-one-shapefile-overlaps-another
categories: [rstats, spatial, gis, sf]
tags: [rstats, spatial, gis, sf]
output: hugodown::md_document
rmd_hash: b85de0f9788de07d

---

I'm on a path at the moment to understand how to calculate the overlap between two areas.

I found a nice [thread on GIS stack exchange](https://gis.stackexchange.com/questions/362466/calculate-percentage-overlap-of-2-sets-of-polygons-in-r) that explained an answer (from user [Sandy AB](https://gis.stackexchange.com/users/161806/sandy-ab))

I was after something with pictures, so here's an example of that.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://tidyverse.tidyverse.org'>tidyverse</a></span><span class='o'>)</span>
<span class='c'>#&gt; ── <span style='font-weight: bold;'>Attaching packages</span> ───────────────────────────── tidyverse 1.3.1 ──</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>ggplot2</span> 3.3.5     <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>purrr  </span> 0.3.4</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>tibble </span> 3.1.3     <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>dplyr  </span> 1.0.7</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>tidyr  </span> 1.1.3     <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>stringr</span> 1.4.0</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>readr  </span> 2.0.0     <span style='color: #00BB00;'>✔</span> <span style='color: #0000BB;'>forcats</span> 0.5.1</span>
<span class='c'>#&gt; ── <span style='font-weight: bold;'>Conflicts</span> ──────────────────────────────── tidyverse_conflicts() ──</span>
<span class='c'>#&gt; <span style='color: #BB0000;'>✖</span> <span style='color: #0000BB;'>dplyr</span>::<span style='color: #00BB00;'>filter()</span> masks <span style='color: #0000BB;'>stats</span>::filter()</span>
<span class='c'>#&gt; <span style='color: #BB0000;'>✖</span> <span style='color: #0000BB;'>dplyr</span>::<span style='color: #00BB00;'>lag()</span>    masks <span style='color: #0000BB;'>stats</span>::lag()</span>
<span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://r-spatial.github.io/sf/'>sf</a></span><span class='o'>)</span>
<span class='c'>#&gt; Linking to GEOS 3.8.1, GDAL 3.2.1, PROJ 7.2.1</span>
<span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://github.com/ateucher/rmapshaper'>rmapshaper</a></span><span class='o'>)</span>
<span class='c'>#&gt; Registered S3 method overwritten by 'geojsonlint':</span>
<span class='c'>#&gt;   method         from </span>
<span class='c'>#&gt;   print.location dplyr</span>

<span class='nv'>brisbane_sf</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://r-spatial.github.io/sf/reference/st_read.html'>read_sf</a></span><span class='o'>(</span><span class='nf'>here</span><span class='nf'>::</span><span class='nf'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='o'>(</span><span class='s'>"content/post/2021-08-18-how-do-i-find-out-how-much-a-shapefile-overlaps-another/data/brisbane-sla4.shp"</span><span class='o'>)</span><span class='o'>)</span>

<span class='nv'>brisbane_simpler_sf</span> <span class='o'>&lt;-</span> <span class='nv'>brisbane_sf</span> <span class='o'>%&gt;%</span> <span class='nf'><a href='https://rdrr.io/pkg/rmapshaper/man/ms_simplify.html'>ms_simplify</a></span><span class='o'>(</span>keep <span class='o'>=</span> <span class='m'>0.01</span><span class='o'>)</span></code></pre>

</div>

## The Data

I started by getting some spatial areas from [this link](https://www.abs.gov.au/statistics/standards/australian-statistical-geography-standard-asgs-edition-3/jul2021-jun2026/access-and-downloads/digital-boundary-files)

I selected, "statistical local areas level 4 2021":

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>knitr</span><span class='nf'>::</span><span class='nf'><a href='https://rdrr.io/pkg/knitr/man/include_graphics.html'>include_graphics</a></span><span class='o'>(</span><span class='nf'>here</span><span class='nf'>::</span><span class='nf'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='o'>(</span><span class='s'>"content/post/2021-08-18-how-do-i-find-out-how-much-a-shapefile-overlaps-another/figs/shapefile-choose.png"</span><span class='o'>)</span><span class='o'>)</span>
</code></pre>
<img src="/Users/njtierney/github/njtierney/web/njtierney.com/content/post/2021-08-18-how-do-i-find-out-how-much-a-shapefile-overlaps-another/figs/shapefile-choose.png" width="700px" style="display: block; margin: auto;" />

</div>

I then did a bit of processing to shrink the data down to just my hometown, Brisbane.

## The Data Vis

Here's my hometown, Brisbane:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>ggplot</span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'>geom_sf</span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>brisbane_sf</span>,
          fill <span class='o'>=</span> <span class='s'>"forestgreen"</span><span class='o'>)</span> 
</code></pre>
<img src="figs/gg-full-brisbane-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Let's say we have a simpler shape file that we want to compare to this, and we want to know how much area we are losing. Here's the "simpler" brisbane in red.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>ggplot</span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'>geom_sf</span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>brisbane_simpler_sf</span>, 
          fill <span class='o'>=</span> <span class='s'>"firebrick"</span><span class='o'>)</span>
</code></pre>
<img src="figs/gg-simple-brisbane-1.png" width="700px" style="display: block; margin: auto;" />

</div>

We can overlay them to see how similar they are, full Brisbane in green, and new brisbane in red:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>ggplot</span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'>geom_sf</span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>brisbane_sf</span>,
          fill <span class='o'>=</span> <span class='s'>"forestgreen"</span>, alpha <span class='o'>=</span> <span class='m'>0.5</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'>geom_sf</span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>brisbane_simpler_sf</span>, 
          fill <span class='o'>=</span> <span class='s'>"firebrick"</span>,
          alpha <span class='o'>=</span> <span class='m'>0.5</span><span class='o'>)</span>
</code></pre>
<img src="figs/gg-overlap-brisbane-1.png" width="700px" style="display: block; margin: auto;" />

</div>

So how do we calculate the difference between the two? We can use `st_intersection` to find where both shapes overlap.

Visualised:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://r-spatial.github.io/sf/reference/geos_binary_ops.html'>st_intersection</a></span><span class='o'>(</span><span class='nv'>brisbane_sf</span>, <span class='nv'>brisbane_simpler_sf</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
  <span class='nf'>ggplot</span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span>
  <span class='nf'>geom_sf</span><span class='o'>(</span>fill <span class='o'>=</span> <span class='s'>"brown"</span>, alpha <span class='o'>=</span> <span class='m'>0.5</span><span class='o'>)</span>
<span class='c'>#&gt; although coordinates are longitude/latitude, st_intersection assumes that they are planar</span>
<span class='c'>#&gt; Warning: attribute variables are assumed to be spatially constant throughout all geometries</span>
</code></pre>
<img src="figs/gg-only-overlap-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Compare this again to the previous plot - these are the brown sections.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>ggplot</span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'>geom_sf</span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>brisbane_sf</span>,
          fill <span class='o'>=</span> <span class='s'>"forestgreen"</span>, alpha <span class='o'>=</span> <span class='m'>0.5</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'>geom_sf</span><span class='o'>(</span>data <span class='o'>=</span> <span class='nv'>brisbane_simpler_sf</span>, 
          fill <span class='o'>=</span> <span class='s'>"firebrick"</span>,
          alpha <span class='o'>=</span> <span class='m'>0.5</span><span class='o'>)</span>
</code></pre>
<img src="figs/gg-overlap-brisbane-2-1.png" width="700px" style="display: block; margin: auto;" />

</div>

But say we want to calculate the ratio of how similar these two shape files are - how do we do that?

The main pieces of what we are want are two things:

1.  The area of the shapefile for the original Brisbane file
2.  The area of the overlap between the two files
3.  The ratio of the area in the original shapefile, and the area of the overlapping area

Then we calculate the area of the overlap compared to the original Brisbane file, and this will tell us the percentage of overlap!

## The area of the shapefile for the original Brisbane file

Here are the steps:

1.  calculate area with `st_area`
2.  Reduce the size of the data - then only keep the relevant data (just keeping the SA4 name (`SA4_NAME21`), and then area, then drop the geometry column).

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>brisbane_sf_areas</span> <span class='o'>&lt;-</span> <span class='nv'>brisbane_sf</span> <span class='o'>%&gt;%</span> 
  <span class='nf'>mutate</span><span class='o'>(</span>brisbane_original_area <span class='o'>=</span> <span class='nf'><a href='https://r-spatial.github.io/sf/reference/geos_measures.html'>st_area</a></span><span class='o'>(</span><span class='nv'>.</span><span class='o'>)</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
  <span class='nf'>select</span><span class='o'>(</span><span class='nv'>SA4_NAME21</span>, <span class='nv'>brisbane_original_area</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
  <span class='nf'><a href='https://r-spatial.github.io/sf/reference/st_geometry.html'>st_drop_geometry</a></span><span class='o'>(</span><span class='o'>)</span>
  
<span class='nv'>brisbane_sf_areas</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 × 2</span></span>
<span class='c'>#&gt;   SA4_NAME21          brisbane_original_area</span>
<span class='c'>#&gt; <span style='color: #555555;'>*</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>                                <span style='color: #555555;'>[m^2]</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span> Brisbane Inner City               81738079</span></code></pre>

</div>

## The area of the overlap between the two files

1.  Calculate the intersection of these two shape files (`st_intersection`)
2.  Calculate that area (`st_area`)
3.  Then only keep the relevant data again

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>intersection_area</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://r-spatial.github.io/sf/reference/geos_binary_ops.html'>st_intersection</a></span><span class='o'>(</span><span class='nv'>brisbane_sf</span>, <span class='nv'>brisbane_simpler_sf</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>mutate</span><span class='o'>(</span>intersect_area <span class='o'>=</span> <span class='nf'><a href='https://r-spatial.github.io/sf/reference/geos_measures.html'>st_area</a></span><span class='o'>(</span><span class='nv'>.</span><span class='o'>)</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>select</span><span class='o'>(</span><span class='nv'>SA4_NAME21</span>, <span class='nv'>intersect_area</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
    <span class='nf'><a href='https://r-spatial.github.io/sf/reference/st_geometry.html'>st_drop_geometry</a></span><span class='o'>(</span><span class='o'>)</span>
<span class='c'>#&gt; although coordinates are longitude/latitude, st_intersection assumes that they are planar</span>
<span class='c'>#&gt; Warning: attribute variables are assumed to be spatially constant throughout all geometries</span>

<span class='nv'>intersection_area</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 × 2</span></span>
<span class='c'>#&gt;   SA4_NAME21          intersect_area</span>
<span class='c'>#&gt; <span style='color: #555555;'>*</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>                        <span style='color: #555555;'>[m^2]</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span> Brisbane Inner City       76638525</span></code></pre>

</div>

## The ratio of the area in the original shapefile, and the area of the overlapping areas

Now we have our pieces, now let us add these columns of the other data back to it:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>intersection_area</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>left_join</span><span class='o'>(</span><span class='nv'>brisbane_sf_areas</span>, 
              by <span class='o'>=</span> <span class='s'>"SA4_NAME21"</span><span class='o'>)</span> 
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 × 3</span></span>
<span class='c'>#&gt;   SA4_NAME21          intersect_area brisbane_original_area</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>                        <span style='color: #555555;'>[m^2]</span>                  <span style='color: #555555;'>[m^2]</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span> Brisbane Inner City       76638525               81738079</span></code></pre>

</div>

And then calculate the ratio - which will tell us how much the simpler shape file overlaps the original ratio.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>intersection_area</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>left_join</span><span class='o'>(</span><span class='nv'>brisbane_sf_areas</span>, 
              by <span class='o'>=</span> <span class='s'>"SA4_NAME21"</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>mutate</span><span class='o'>(</span>weight <span class='o'>=</span> <span class='nv'>intersect_area</span> <span class='o'>/</span> <span class='nv'>brisbane_original_area</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 × 4</span></span>
<span class='c'>#&gt;   SA4_NAME21          intersect_area brisbane_original_area   weight</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>                        <span style='color: #555555;'>[m^2]</span>                  <span style='color: #555555;'>[m^2]</span>      <span style='color: #555555;'>[1]</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span> Brisbane Inner City       76638525               81738079 0.937611</span></code></pre>

</div>

And, because I think it's good practice, all together as a function:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>calculate_spatial_overlap</span> <span class='o'>&lt;-</span> <span class='kr'>function</span><span class='o'>(</span><span class='nv'>shape_1</span>,
                                      <span class='nv'>shape_2</span><span class='o'>)</span> <span class='o'>&#123;</span>
  
  
  <span class='nv'>intersection_area</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://r-spatial.github.io/sf/reference/geos_binary_ops.html'>st_intersection</a></span><span class='o'>(</span><span class='nv'>shape_1</span>, <span class='nv'>shape_2</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>mutate</span><span class='o'>(</span>intersect_area <span class='o'>=</span> <span class='nf'><a href='https://r-spatial.github.io/sf/reference/geos_measures.html'>st_area</a></span><span class='o'>(</span><span class='nv'>.</span><span class='o'>)</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>select</span><span class='o'>(</span><span class='nv'>SA4_NAME21</span>, <span class='nv'>intersect_area</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
    <span class='nf'><a href='https://r-spatial.github.io/sf/reference/st_geometry.html'>st_drop_geometry</a></span><span class='o'>(</span><span class='o'>)</span>
  
  <span class='c'># Create a fresh area variable for counties</span>
  <span class='nv'>shape_2_areas</span> <span class='o'>&lt;-</span> <span class='nv'>shape_2</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>mutate</span><span class='o'>(</span>original_area <span class='o'>=</span> <span class='nf'><a href='https://r-spatial.github.io/sf/reference/geos_measures.html'>st_area</a></span><span class='o'>(</span><span class='nv'>.</span><span class='o'>)</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>select</span><span class='o'>(</span><span class='nv'>original_area</span>, <span class='nv'>SA4_NAME21</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
    <span class='nf'><a href='https://r-spatial.github.io/sf/reference/st_geometry.html'>st_drop_geometry</a></span><span class='o'>(</span><span class='o'>)</span>
  
  <span class='nv'>intersection_area</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>left_join</span><span class='o'>(</span><span class='nv'>shape_2_areas</span>, 
              by <span class='o'>=</span> <span class='s'>"SA4_NAME21"</span><span class='o'>)</span> <span class='o'>%&gt;%</span> 
    <span class='nf'>mutate</span><span class='o'>(</span>weight <span class='o'>=</span> <span class='nv'>intersect_area</span> <span class='o'>/</span> <span class='nv'>original_area</span><span class='o'>)</span>
  
<span class='o'>&#125;</span></code></pre>

</div>

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>calculate_spatial_overlap</span><span class='o'>(</span>
    <span class='nv'>brisbane_sf</span>, <span class='nv'>brisbane_simpler_sf</span>
  <span class='o'>)</span>
<span class='c'>#&gt; although coordinates are longitude/latitude, st_intersection assumes that they are planar</span>
<span class='c'>#&gt; Warning: attribute variables are assumed to be spatially constant throughout all geometries</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 × 4</span></span>
<span class='c'>#&gt;   SA4_NAME21          intersect_area original_area    weight</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>                        <span style='color: #555555;'>[m^2]</span>         <span style='color: #555555;'>[m^2]</span>       <span style='color: #555555;'>[1]</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span> Brisbane Inner City       76638525      81674484 0.9383411</span></code></pre>

</div>

# Why do this?

Our use case in this example was to calculate the difference between shapefiles so we could then use this overalapping difference as a weight in subsequent measurements.

But you can do more with this - the example from [the GIS stack exchange thread](https://gis.stackexchange.com/questions/362466/calculate-percentage-overlap-of-2-sets-of-polygons-in-r) was trying to calculate the amount of overlap of a lot of smaller shape files on another shapefile - a measurement often referred to as "coverage".

And that's it - maybe that's helpful, maybe I've done something wrong,

# Thanks

Thank you again to user [Sandy AB](https://gis.stackexchange.com/users/161806/sandy-ab) from Stack Exchange, who posted the answer!

