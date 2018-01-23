The idea of using purrr is to solve iteration problems by using:

1.  functions that write for loops for you
2.  with consistent syntax & output, and
3.  convenience shortcuts for specifying functions to iterate

Setup
-----

Checking if you have the packages

    library(tidyverse)
    library(repurrrsive) #devtools::install_github("jennybc/repurrrsive")

We will working with the sw\_people dataset that is in repurrrsive
package. It's a dataset about Star Wars characters, queried from the
[Star Wars API](https://swapi.co/).

Now, let's get our hands dirty and look at the data.

First, let's examine sw\_people.

    class(sw_people)

    ## [1] "list"

Okay, now we know that sw\_people is a list. Let's see how many elements
are there in sw\_people.

    length(sw_people)

    ## [1] 87

YOUR TURN:

1.  Who is the first person listed in sw\_people?
2.  What information is given for the first person?
3.  What is the difference between sw\_people\[1\] and sw\_people
    \[\[1\]\]

Wow, there is quite a lot of information for Luke and there're more than
one character in Star Wars! I'm interested to see how many starships has
each character been in.

First, let's start small by finding out how many starships does **Luke**
have.

    length(sw_people[[1]]$starships)

    ## [1] 2

So, if I want to know how many starships other characters have, what
should I change?

    length(sw_people[[2]]$starships)
    length(sw_people[[3]]$starships)
    length(sw_people[[4]]$starships)

But there are so many characters in sw\_people, how can we do that more
systemtically?

map()!
======

`map()` helps to solve an iteration problem like "for each element of .x
for .f" by using **map(.x, .f, ...)**.

In `map()`, there are two important inputs.

-   .x: the element you want to loop through: a vector, a list or a data
    frame (for each column)
-   .f: a function that you want to be applied to each element

In our example above, we want to solve: for **each person** in
sw\_people, the **number of starships** they have. As noted above, we
can use `length(x$starships)` to get the count of number of starships
they have.

    map(sw_people, ~length(.x$starships)) %>% head()

    ## [[1]]
    ## [1] 2
    ## 
    ## [[2]]
    ## [1] 0
    ## 
    ## [[3]]
    ## [1] 0
    ## 
    ## [[4]]
    ## [1] 1
    ## 
    ## [[5]]
    ## [1] 0
    ## 
    ## [[6]]
    ## [1] 0

For those who are familiar with for loops or s/lapply(), you will know
that we can get the same numbers by doing a foor loop:

    num_starship <- numeric()
    for (i in 1:length(sw_people)){
      num_starship[i] <- length(sw_people[[i]]$starships)
    }
    num_starship

    ##  [1] 2 0 0 1 0 0 0 0 1 5 3 0 2 2 0 0 1 1 0 0 1 0 0 1 0 0 0 1 0 1 0 0 0 0 0
    ## [36] 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0
    ## [71] 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 3

or a sapply()

    sapply(sw_people, function(x) length(x$starships))

    ##  [1] 2 0 0 1 0 0 0 0 1 5 3 0 2 2 0 0 1 1 0 0 1 0 0 1 0 0 0 1 0 1 0 0 0 0 0
    ## [36] 0 1 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0
    ## [71] 0 0 0 0 0 0 1 0 0 0 0 0 0 1 0 0 3

Why should we use `map()`?
==========================

The main advantage of `map()` is the helpers which allow you to write
compact code for common special cases. It also has much greater
consistency between the functions, for example, the first argument to
all map functions is always the data while it's different between
`lapply()` and `mapply()`. Hadley also commented on the comparison of
purrr and lapply on this [Stack Overflow
post](https://stackoverflow.com/questions/45101045/why-use-purrrmap-instead-of-lapply/47123420#47123420)
and his [Twitter
post](https://twitter.com/hadleywickham/status/927199362668814336).

Let's try to use map for another example. Let's find the name of each
character's home world.

We will use the variable, planet\_lookup that is created using the
following code, but we can ignore the code details for now.

    planet_lookup <- map_chr(sw_planets, "name")  %>%  # planets 
      set_names(map_chr(sw_planets, "url")) #name as the URL
    head(planet_lookup)

    ## http://swapi.co/api/planets/2/ http://swapi.co/api/planets/3/ 
    ##                     "Alderaan"                     "Yavin IV" 
    ## http://swapi.co/api/planets/4/ http://swapi.co/api/planets/5/ 
    ##                         "Hoth"                      "Dagobah" 
    ## http://swapi.co/api/planets/6/ http://swapi.co/api/planets/7/ 
    ##                       "Bespin"                        "Endor"

planet\_lookup is a named character that the URLs are the names and the
home world planet names are the value. In our sw\_people data, the home
world of each character is stored in sw\_people\[\[i\]\]$homeworld as a
URL.

    sw_people[[1]]$homeworld

    ## [1] "http://swapi.co/api/planets/1/"

We can use the URL as an index to find the planet name of Luke from
planet\_lookup.

    planet_lookup[sw_people[[1]]$homeworld]

    ## http://swapi.co/api/planets/1/ 
    ##                     "Tatooine"

Your turn! Can you use `map()` to find the name of each character's home
world?

**Tips**: think about what do we need to change to find it for each
character

Other map functions
===================

As you can notice from our above example, the output from `map()` is
always a **list**.

Sometimes, we want to have the output in other simpler formats. There
are a few other functions in purrr that can help:

1.  **map\_lgl()** returns logical vector
2.  **map\_int()** returns integer vector
3.  **map\_dbl()** returns double vector (What is \[double
    vector\]?(<http://uc-r.github.io/integer_double/>))
4.  **map\_chr()** returns a character vector

Another function, `walk()` returns nothing at all, it's helpful in cases
where you:

1.  only want to print to screen
2.  plot to graphics device
3.  file maniuplation (saving, writing, moving, etc.)
4.  system calls

Your turn! Can you use map and the other typed functions to answer the
following question?

1.  How many starships has each character been in?
2.  What color is each character's hair?
3.  Is the character male?
4.  How heavy is each character?

To find out how heavy is each character, we want the **numeric** value
of "mass" of each character from the dataset.

If we use `map_dbl()`, it will not work, because "mass" is stored as a
**string** in the data

    map_dbl(sw_people, ~.x[["mass"]])

    ## Error: Can't coerce element 1 from a character to a double

    map(sw_people, ~ .x[["mass"]]) %>% head()

    ## [[1]]
    ## [1] "77"
    ## 
    ## [[2]]
    ## [1] "75"
    ## 
    ## [[3]]
    ## [1] "32"
    ## 
    ## [[4]]
    ## [1] "136"
    ## 
    ## [[5]]
    ## [1] "49"
    ## 
    ## [[6]]
    ## [1] "120"

The solution is to do it in two steps.

    map_chr(sw_people, ~ .x[["mass"]]) %>% # map the character
      readr::parse_number(na = "unknown") #convert character to numbers and turn "unknown" to NA

    ##  [1]   77.0   75.0   32.0  136.0   49.0  120.0   75.0   32.0   84.0   77.0
    ## [11]   84.0     NA  112.0   80.0   74.0 1358.0   77.0  110.0   17.0   75.0
    ## [21]   78.2  140.0  113.0   79.0   79.0   83.0     NA     NA   20.0   68.0
    ## [31]   89.0   90.0     NA   66.0   82.0     NA     NA     NA   40.0     NA
    ## [41]     NA   80.0     NA   55.0   45.0     NA   65.0   84.0   82.0   87.0
    ## [51]     NA   50.0     NA     NA   80.0     NA   85.0     NA     NA   80.0
    ## [61]   56.2   50.0     NA   80.0     NA   79.0   55.0  102.0   88.0     NA
    ## [71]     NA   15.0     NA   48.0     NA   57.0  159.0  136.0   79.0   48.0
    ## [81]   80.0     NA     NA     NA     NA     NA   45.0

.f can be a formula, a string or an integer
===========================================

    map_chr(sw_people, ~ .x[["hair_color"]]) # a formula
    # can be
    map_chr(sw_people, "hair_color") # a string

    char_starships <- map(sw_people, "starships")
    map_int(char_starships, length) # a function
    # equivalent to
    map_int(sw_people, ~length(.x[["starships"]])) # a formula

Your turn! Star Wars Challenges

1.  Which film (sw\_films) has the most characters?
2.  Which sw\_species has the most possible eye colors?
3.  Which sw\_planets do we know the least about (i.e., have the most
    "unknown" entries)?

What if I have more than 1 variable to iterate?
===============================================

map2()!
-------

Similar to `map()`, `map2()` helps with iteration problems with two
variables. map2(.x, .y, .f): for each element of .x and corresponding
element of .y, apply .f

    letter <- c("a", "b", "c")
    number <- c(1, 2, 3)
    map2(letter, number, paste)

    ## [[1]]
    ## [1] "a 1"
    ## 
    ## [[2]]
    ## [1] "b 2"
    ## 
    ## [[3]]
    ## [1] "c 3"

It can be helpful for plotting.

    gap_split_small <- gap_split[1:10]
    countries <- names(gap_split_small)

    # For each country create a ggplot of Life Expectancy through time 
    # with a title

    # For one country
    ggplot(gap_split_small[[1]], aes(year, lifeExp)) +
      geom_line() +
      labs(title = countries[[1]])

![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-1.png)

    # For all countries
    plots <- map2(gap_split_small, countries, 
      ~ ggplot(.x, aes(year, lifeExp)) + 
          geom_line() +
          labs(title = .y))

    plots[[1]]

![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-2.png)

    # Display all plots
    walk(plots, print) # this might take awhile

![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-3.png)![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-4.png)![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-5.png)![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-6.png)![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-7.png)![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-8.png)![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-9.png)![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-10.png)![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-11.png)![](purrr-RLadies-talk-presentation_files/figure-markdown_strict/unnamed-chunk-18-12.png)

    # Save all plots
    walk2(.x = plots, .y = countries, 
      ~ ggsave(filename = paste0(.y, ".pdf"), plot = .x))

    ## Saving 7 x 5 in image
    ## Saving 7 x 5 in image
    ## Saving 7 x 5 in image
    ## Saving 7 x 5 in image
    ## Saving 7 x 5 in image
    ## Saving 7 x 5 in image
    ## Saving 7 x 5 in image
    ## Saving 7 x 5 in image
    ## Saving 7 x 5 in image
    ## Saving 7 x 5 in image

    # Argh! I didn't want all those pictures in this directory,
    # remove them all
    file.remove(paste0(countries, ".pdf"))

    ##  [1] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE

Similar to `map()`, there are also other type functions for `map2()`:
`walk2()`, `map2_lgl()`, `map2_int()`, `map2_dbl()`, `map2_chr()` that
will give you outputs in other formats.

What if I have more than 2 variables to iterate?
================================================

pmap()!
-------

pmap(.l, .f, ...): for each element of each vector in .l, apply .f

What if I want to apply multiple functions on the same variable?
================================================================

invoke\_map()!
--------------

invoke\_map(.f, .x, ...): for each function in .f, apply to .x

Reference
=========

Most of the material references from Charlotte Wickham's [purrr
tutorial](https://github.com/cwickham/purrr-tutorial). There are also
some
[challenges](https://github.com/cwickham/purrr-tutorial/tree/master/challenges)
on the repo that you should try!
