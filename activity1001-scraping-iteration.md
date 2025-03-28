Activity 10.1 - Iterating
================
Alexa Kraklau

In this activity, we will build a scraping function that uses TMDB (an
open movie database project compared to IMDb). The web is a great
resource of information, especially with the ever increasing amount of
data available on the web. Copying and pasting this information is time
consuming and susceptible to many errors. Web scraping is a way to
extract this information automatically and transform it into a
structured data set. In this activity we will use screen scraping in
which we extract data from the source code of a website with an HTML
parser (this could also be done with regular expression matching).

Web scraping is not unique to R. Python, Perl, and Java are also
efficient tools for this. However, this is a course in using R…

## Go go, SelectorGadget!

![Inspector Gadget
gif](https://media.giphy.com/media/z9bcWnpU1QnXG/giphy.gif)

As we will see, most data on the web is available as HTML (hypertext
markup language). And, although it is structured (hierarchical/tree
based), HTML data is often not available in an analysis-ready format
(i.e., tidy).

`{rvest}` is a package by the tidyverse team that makes basic processing
and manipulation of HTML data more straight forward. However, it is not
loaded with the tidyverse so you will need to load it separately. In the
space below, load `{tidyverse}` and `{rvest}`.

``` r
install.packages("robotstxt")
```

    ## Installing package into '/home/kraklaua/R/x86_64-pc-linux-gnu-library/4.1'
    ## (as 'lib' is unspecified)

    ## Error in contrib.url(repos, type): trying to use CRAN without setting a mirror

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ ggplot2 3.3.4     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.2     ✓ dplyr   1.0.7
    ## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
    ## ✓ readr   1.4.0     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(rvest)
```

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

The core functions that you may find useful are:

-   `read_html`: reads HTML data from a URL or character string
-   `html_nodes`: selects specified nodes from the HTML document using
    CSS selectors
-   `html_table`: parses an HTML table into a data frame
-   `html_text`: extracts tag pairs’ content
-   `html_name`: extracts tags’ names.
-   `html_attrs`: extracts all of each tag’s attributes
-   `html_attr`: extracts tags’ attribute value by name

Also, Hadley offers a
[vignette](https://rvest.tidyverse.org/articles/selectorgadget.html) for
a handy tool browser add-in (SelectorGadget) that helps determine the
minimal CSS selector you need to extract certain web page elements.
However, you might be comfortable inspecting (e.g., Ctrl + Shift + I)
the HTML structure of a webpage. Below are instructions to get started
using SelectorGadget, but you are welcome to use the Inspect tool:

1.  Install SelectorGadget in your web browser,
2.  Go to TMDB’s [Squid Game](https://www.themoviedb.org/tv/93405) page,
3.  Open SelectorGadget (a box will open in the bottom-right corner of
    your browser),
4.  Explore different page elements (e.g., a movie’s title). When you
    click on different items using SelectorGadget, they should highlight
    in green, generates a minimal CSS selector (e.g., `.10 a`), and
    highlight everything that is matched by that selector in yellow (and
    tell you how many items on the page match that selection).
5.  Click on “Clear” to de-select it.
6.  You can also search for different elements. Type `.panel` when you
    have nothing else selected (i.e., where it says “No valid path
    found.”).
7.  Click or type to explore at least these elements:

-   Title
-   Year
-   Content Rating

What are the minimal CSS selectors for the following elements: a show’s
title, year, and content rating?

First, we should make sure that we are allowed to.

In your **Console**, install `{robotstxt}`.

``` r
library(robotstxt)
paths_allowed("http://www.tmdb.com")
```

    ## www.tmdb.com

    ## Warning in request_handler_handler(request = request, handler =
    ## on_file_type_mismatch, : Event: on_file_type_mismatch

    ## 

    ## [1] TRUE

Compare this to, for example:

``` r
paths_allowed("http://www.facebook.com")
```

    ##  www.facebook.com

    ## [1] FALSE

Are you able to scrape data from TMDB or facebook?

Data can be scraped from TMDB but cannot be scraped from Facebook.

Below is the start of a `scrape_tmdb` function. The goal of this
function is that when provided with a TV show’s TMDB’s URL, it will
scrape the show’s title, year, content rating, and user score. User
score will be the harder part of this so I have provided it for you.
Also, I show you how to make the output of this function organized in a
tibble. Use this TMBD page to help you determine the different
attributes: `https://www.themoviedb.org/tv/93405`

``` r
scrape_tmdb <- function(url){
  
  page <- read_html(url)
  
  # Update this part of the function to scrape the appropriate information
  
  # Scrape TMDB score
  score <- page %>% 
    html_node("span.icon") %>% 
    html_attr("class") %>% 
    str_replace(".+r(.+)", "\\1") %>% # `\\1` refers to string matched by the first capture group `(.+)`
    as.numeric()
  
  # Create a tibble containing all the scraped information
  tibble(
    title = title,
    year = year,
    rating = rating,
    score = score
  )
  
}
```

Use your function to obtain the information from this TV Show:
`https://www.themoviedb.org/tv/2382`

## Gathering the data

Currently we have a function that can produce TV show information if
provided with a URL, but where can we get a list of URLs of the most
popular TV shows on TMDB? Using inspect and/or SelectorGadget, explore
how you might get the URLs from here:
<https://www.themoviedb.org/tv/top-rated>

Write the code that reads in a list/vector of most popular TV shows on
TMDB, then looks in the appropriate field to find the `"href"` (hint:
`html_attr`) to obtain this list of URLs. Call this character vector
`tmdb_tv_urls` and it should have the top 20 tv shows.

After you have the vector of show subpage locations, you will need to
*update* this vector to have each entry begin with
`"http://www.themoviedb.com"` so that you have valid URLs (hint: look
into `paste`, `paste0`, or my favorite `glue::glue`)

Test `scrape_show_info` on the first few URLs (e.g., `tmdb_tv_urls[1]`
is the *first* URL).

Now we will scrap the show information for every page for the top 100 TV
shows. We could do this by typing `scrap_show_info(tmdb_tv_urls[i])` for
each of the 100 shows, but this would be extremely inefficient and we
would probably accidentally scrape some TV shows multiple times while
skipping others.

To do this, you will explore two methods:

1.  Using a `for` loop
2.  Using `purrr::map` functions

## `for` loop

Remember that with `for` loops, we first need to create an object that
will hold the output from the loop. First create an *empty* `tibble`
called `tmdb_for_loop` with four columns: title, year, content rating,
and user score.

-   `title` which is a character column,
-   `year` which is a numeric column,
-   `rating` which is a numeric column, and
-   `score` which is a numeric column.

Then, loop through the entries in your vector of `tmdb_tv_urls`. In your
loop, overwrite `tmdb_for_loop` with by *bind*ing the *rows* of the
previous tibble with the next TV show’s information tibble.

IF you get `HTTP Error 429 (Too many requests)`, you will want to slow
down your website hit rate. To do this, add `Sys.sleep(runif(1))` to the
beginning of the `scrape_show_info` function (do it above and re-run the
code). This will add a random length pause (some equally likely random
length between 0 and 1 seconds) to your function. We only need to do
this because we are scraping from a website - this is not necessary for
all functions.

Verify that you have a complete tibble by visualizing the `score` for TV
shows that made it on this top 20 list.

## `purrr::map`

`{purrr}` has a wealth of functions that make iterating functions more
streamlined.

-   [purrr site](https://purrr.tidyverse.org/)

What we want to do is *map* the `scrape_show_info` function to each
element of our character vector `tmdb_tv_urls` to output a dataframe.
This is a purrr-fect (sorry) opportunity to use `map_df`…

![Sorry, not
sorry](https://media.giphy.com/media/xUA7aVAw3xQ4pzYkiA/giphy.gif)

With the `map*` functions, you specify a list or vector that you want to
apply a function to and it returns an object of the same length and the
input. There are several types of pre-identified outputs, but we want to
return a dataframe/tibble so we will use `map_df` Create a new object
called `tmdb_map` that contains the show information for the top 100 TV
shows using `map_df`

Verify that your `tmdb_for_loop` and `tmdb_map` objects contain the
exact same information using the `waldo::compare` function.

## Update your function

Update your `scrape_show_info` function (do it above) to contain as much
additional information that you can. Also, can you scrape more than the
first 20 shows? Create a `tmdb_popular_shows` tibble that contains all
this show information for the top 100 TV shows. Save this tibble as a
`.csv` file in a `/data` folder within your RStudio Project.

## Attribution

This activity is inspired by one of Mine Çetinkaya-Rundel’s [STA
199](http://www2.stat.duke.edu/courses/Spring18/Sta199/) labs.
