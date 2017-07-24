---
title: "Getting Started with R - Part 2: Vectors"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---
Expanding on the basics of the simple data classes we now start with the first higher dimension construct: vectors
<!--more-->


I am posting this tutorial as I learn R. I will add comments section soon and will respond to feedback for errata.
{: .notice}


## Combining values

Vectors are created by combining values under a single structure using the `c()` function. Think `c` for combine.
We can create vectors of our simple types: `numeric`, `logical`, `characters` and `integer`

``` R
combined_numeric <- c(12.3, 34.5, 67.8)
combined_logical <- c(TRUE, FALSE, TRUE, TRUE)
combined_integer <- c(1, 2, 4)

combined_numeric
combined_logical
combined_integer
```

Try a variable of combined `characters` type on your own.

## Coercing

.What happens if we mix types? Go ahead and try.

``` R
combined_mixed <- c(12.3, FALSE, 4)
combined_mixed
```

`FALSE` was converted to `0.0` and `4` to `4.0`.  This is called coercing and it can get alot worse than this sample. What happens if we try to combine those with a `characters` element

``` R
combined_basic_classes <- c(12.3, FALSE, 4, "Arggh")
.combined_basic_classes
```

**Careful!** When combining different classes into a vector. Values are coerced to a type that can represent them all, often the `numeric` or `characters` type
{: .notice}

## Basic Indexing

We have already touched on the fact that R uses 1-based indexing. This is different from most other programming languages that use 0-based indexing

``` R
combined_numeric <- c(12.3, 34.5, 67.8)
combined_logical <- c(TRUE, FALSE, TRUE, TRUE)
combined_integer <- c(1, 2, 4)

combined_numeric[1]
combined_logical[2]
combined_integer[3]
```

The `[]` syntax is called the extract\replace operator. But for now think of it as a way to index values, we'll dig into its advanced properties later. 

As you can see the first, second and third values of each of the vectors were returned. What if we pass an index outside the range such as `combined_integer[4]`. Try it. 

``` R
combined_integer <- c(1, 2, 4)
combined_integer[4]
```

Did you get what you expected? Most programming languages would throw an "Index out of bounds" exception. R returns a polite `NA` value. Simply it means "Not Available" (the value is missing). `NA` is somewhat analogous to a null value, in that it shows the absence of a value. But, as you will soon see, `NULL` is also present in the R language

Try the following 

``` R
combined_integer <- c(1, 2, 4)
combined_integer[0]
```

Bet you were expecting an `NA` value. This is an odd case. The online help states "An index of NULL is treated as integer(0)". So in essence it was the same as calling `combined_integer[NULL]`. 

## Help

Before I continue with more advanced indexing, a quick note on the help system. To get help on anything simply prefix `?` to the function For instance, to get help on the combine function we can use:

``` R
?c
```
or in some cases we need to quote the item for which we want help, like this:

``` R
?"["
```
