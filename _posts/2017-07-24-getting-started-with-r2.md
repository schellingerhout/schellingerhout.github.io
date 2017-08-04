---
title: "Getting Started with R - Part 2: Vector Basics"
excerpt.separator: "<!--more-->"
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

```R
combined.numeric <- c(12.3, 34.5, 67.8)
combined.logical <- c(TRUE, FALSE, TRUE, TRUE)
combined.integer <- c(1, 2, 4)

combined.numeric
combined.logical
combined.integer
```

Try a variable of combined `characters` type on your own. Also try to find out the class of the vectors you created. Was it what you expected? As you can see it returned the class of the first element.

## Coercing

What happens if we mix types? Go ahead and try.

```R
combined.mixed <- c(12.3, FALSE, 4)
combined.mixed
```

`FALSE` was converted to `0.0` and `4` to `4.0`.  This is called coercing and it can get alot worse than this sample. What happens if we try to combine those with a `characters` element?

```R
combined.basic.classes <- c(12.3, FALSE, 4, "Arggh")
combined.basic.classes
```

Call `class` against `combined.mixed` and `combined.basic.classes`. Do you understand the result that is returned? Again it returns the class of the first element, in this case the coerced type.

**Careful!** When combining different classes into a vector. Values are coerced to a type that can represent them all, often the `numeric` or `characters` type
{: .notice--warning}

## Basic Indexing

We have already touched on the fact that R uses 1-based indexing. This is different from most other programming languages that use 0-based indexing

```R
combined.numeric <- c(12.3, 34.5, 67.8)
combined.logical <- c(TRUE, FALSE, TRUE, TRUE)
combined.integer <- c(1, 2, 4)

combined.numeric[1]
combined.logical[2]
combined.integer[3]
```

The `[]` syntax is used to denote the extract\replace operator. For now think of it as a way to index values, we'll dig into its advanced properties later.

As you can see the first, second and third values of each of the vectors were returned. What if we pass an index outside the range such as `combined.integer[4]`. Try it. 

```R
combined.integer <- c(1, 2, 4)
combined.integer[4]
```

Did you get what you expected? Most programming languages would throw an "Index out of bounds" exception. R returns a polite `NA` value. Simply it means "Not Available" (the value is missing). `NA` is somewhat analogous to a null value, in that it shows the absence of a value. In the next post it will hopefully become more clear why `NA` is returned. Also, as you will soon see, `NULL` is also present in the R language

Try the following 

```R
combined.integer <- c(1, 2, 4)
combined.integer[0]
```

Bet you were expecting an `NA` value. This is an odd case. The online help states "Rows with an index 0 are ignored" and "An index of NULL is treated as integer(0)". So it was essentially the same as calling `combined.integer[NULL]`. 

## Help

Before I continue with more advanced vector value extracting, a quick note on the help system. To get help on anything simply prefix `?` to the function For instance, to get help on the combine function we can use:

```R
?c
```
or in some cases we need to quote the item for which we want help, like this:

```R
?"["
```

## Naming and combining vectors

Sometimes working with vectors is easier if we can name the indices. To do that we can use the `names` function

```R
probability.of.rain <- c(0.8, 0.2, 0.05, 0.4, 0.65)
names(probability.of.rain) = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
probability.of.rain
```
Now we see a nice output showing the labels over each indexed value

```  
   Monday   Tuesday Wednesday  Thursday    Friday 
     0.80      0.20      0.05      0.40      0.65 
```

Cool, say we have another two vectors like this 

```R
probability.of.rain.work <- c(0.8, 0.2, 0.05, 0.4, 0.65)
names(probability.of.rain.work) = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
probability.of.rain.play <- c(0.1, 0.0)
names(probability.of.rain.play) = c("Saturday", "Sunday")
```
How can I combine the two vectors? Combine? Combine? yes, combine as in `c()`

```R
c(probability.of.rain.play, probability.of.rain.work)
```
yields this nice result

```
 Saturday    Sunday    Monday   Tuesday Wednesday  Thursday    Friday 
     0.10      0.00      0.80      0.20      0.05      0.40      0.65 
```

Of course, as before, we can assign the result to a new variable as well. Use `?c`, notice that the signature can accept a parameter `use.names` that has a default value `TRUE`. This is why our index names were preserved
