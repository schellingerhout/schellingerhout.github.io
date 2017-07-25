---
title: "Getting Started with R - Part 2: Vector Basics"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---



Expanding on the basics of the simple data classes in [Part 1: The Console and Variables]({{ site.baseurl }}{% post_url 2017-07-23-getting-started-with-r %}), we now start with the first higher dimension construct: vectors
<!--more-->


I am posting this tutorial as I learn R. I will respond to feedback for errata in the comments.
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
{: .notice--warning}

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

## Naming and combining vectors

Sometimes working with vectors is easier if we can name the indices. To do that we can use the `names` function

``` R
probability_of_rain <- c(0.8, 0.2, 0.05, 0.4, 0.65)
names(probability_of_rain) = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
probability_of_rain
```
Now we see a nice output showing the labels over each indexed value

```  
   Monday   Tuesday Wednesday  Thursday    Friday 
     0.80      0.20      0.05      0.40      0.65 
```

Cool, say we have another two vectors like this 

``` R
probability_of_rain_work <- c(0.8, 0.2, 0.05, 0.4, 0.65)
names(probability_of_rain_work) = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
probability_of_rain_play <- c(0.1, 0.0)
names(probability_of_rain_play) = c("Saturday", "Sunday")
```
How can I combine the two vectors? Combine? Combine? yes, combine as in `c()`

```R
c(probability_of_rain_play, probability_of_rain_work)
```
yields this nice result

```
 Saturday    Sunday    Monday   Tuesday Wednesday  Thursday    Friday 
     0.10      0.00      0.80      0.20      0.05      0.40      0.65 
```

Of course as before we can assign the result to a new variable as well. use `?c`, notice that the signature can accept a parameter `use.names` that has a default value `TRUE`. This is why our index names were preserved
