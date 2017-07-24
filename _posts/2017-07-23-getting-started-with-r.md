---
title: "Getting Started with R - Part 1: The Console and Variables"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---

Here is a very fast moving tutorial in getting started in R. Hold on to your hat
<!--more-->

I am posting this tutorial as I learn R. I will add comments section soon and will respond to feedback for errata.
{: .notice}

## Installation

Install [R-Studio](https://www.rstudio.com/). You will most likely work in it and it will direct you to get the correct version of R for your platform. If you want to install R by itself and you can get R from [CRAN](https://cran.r-project.org/). The tutorial will use R-Studio.

## Mathematical Operators
In R-Studio you can directly enter new commands in the console. Try some typing in the console. Specifically, try a few basic mathematical operations
``` R
3 + 5
3 - 5
3 * 5
3 ^ 5
13 %% 5
```

By looking at the results the first few are very obviously addition, subtraction, multiplication and exponentiation, the last one `%%` may not be that obious. That is the modulo operator (returns the remainder after division)

**Self Study**: Find out what `**` and `%/%` does.
{: .notice--info}

When you entered the commands you may have noticed that the results looked a bit odd. For instance

``` R
3 + 5
```

yields

``` R
[1] 8
```

The 8 makes sense, but what is up with the `[1]`?

Since R is mostly concerned with multi-dimensional data it is telling you that a one dimensional construct was returned and the value at its first index had a value of 8.

**Watch out!** Most programming languages have zero based indexing, R has index 1 as the first.
{: .notice--warning}

## Variables

Variables are assigned using the `<-` operator as in `variable <- value`
Can you assign `42` to the variable `answer`.

``` R
answer <- 42
```

Try typing the command above in the console. Notice the console immediately returns to the `>` prompt and does not return a value. You may have seen a value show up in the environment pane, don't worry about that for now.

Type the name of the variable by itself

``` R
answer
```
This time the console returns

``` R
[1] 42
````
Try typing this

``` R
Answer
```
What happened?

**Watch out!** R variables are case sensitive.
{: .notice--warning}

Variables can be used wherever values are used. Try these

``` R
answer + 5
answer - 5
answer * 5
answer ^ 5
answer %% 5
```

Using a variable with an operator does not modify its value. At least not in with well written libraries.

You can change the value of a variable. Try this

``` R
answer <- 42
answer <- 24
answer
```

Our value of answer is now `24`. When a variable is used on the right side of an assigment expression it evaluated and then the value is returned. For instance, this allow us to modify variables using equations with their own values.

``` R
answer <- 42
answer <- answer / 2
answer
```

This also means that variables can be combined in expressions.

``` R
width <- 5
height <- 6
area <- height  * width
area
```

## Data Types

We have already seen integers. Here are the other simple types:
* Floating point numbers (called "numerics" in R) have a decimal point (as in `99.9`). 
* Booleans are denoted by `TRUE` and `FALSE` (or `T` and `F` if you are lazy) and are called "logical" in R. 
* Strings are expressed inside quotation marks as in `"Why did they change all the type names?"` 

Indeed even strings are called "characters" in R.

Can you try to create variables of each kind and return their values? You can verify the type of a variable by calling `class(variable_name)` to get the type of the variable. 

Here is an example that should be enough to help you do the boolean (I mean logical), and floating point (I mean numeric) types.

``` R
my_string_err_characters <- "Jasper"
class(my_string_err_characters)
```

## Comments

Comments are done with a `#` character. Everything following to the end of the line is ignored. 

``` R
# Calculate rectangular area
width <- 5 
height <- 6 # both measurements in inches
area <- height * width
area
```

You will most likely not use comments in the console, but you will probably use them in an R script.  
