---
title: "Getting Started with R - Part 9: Lists and the [[]] operator"
excerpt.separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---
Lists allows us to keep a collection of various objects together
<!--more-->


I am posting this tutorial as I learn R. I will respond to feedback for errata in the comments.
{: .notice}


## What is a list?

A `list` object allows us to maintain a a list objects of various types. We can mix different values of different types in a 1 dimensional list structure similar to what you find in a `vecotor` but without the restrictions of holding a single type.

## Constructing a data frame

A list is constructed with `list()` and we pass in the values we want to keep in that list. We can pass matrices, vectors, simple values and even dataframes to be held in a list 

```R
city.name <- c( "Columbus", "Cleveland", "Cincinnati")
alist <- list( city.name, 12.4, TRUE, matrix(1:9, nrow=3))
alist
```
 returns the list and its content

 ```
[[1]]
[1] "Columbus"   "Cleveland"  "Cincinnati"

[[2]]
[1] 12.4

[[3]]
[1] TRUE

[[4]]
     [,1] [,2] [,3]
[1,]    1    4    7
[2,]    2    5    8
[3,]    3    6    9
 ```

## The `[[]]` indexer
The `[[]]` indexer is slightly different from the `[]` indexer in that it returns a single element and drops names. If we index our list using `[]`, you will notice that elements are returned with a `[[listindex]]`. For instance

```R
alist[1]
```
returns
```
[[1]]
[1] "Columbus"   "Cleveland"  "Cincinnati"
```
and

```R
alist[[1]]
```
returns
```
[1] "Columbus"   "Cleveland"  "Cincinnati"
```
The difference is not immediately obious until you try to use the result. Try these command

```R
alist[[1]][1]
alist[[1]][2]

alist[1][1]
alist[1][2]
```
returns
```
[1] "Columbus"

[1] "Cleveland"

[[1]]
[1] "Columbus"   "Cleveland"  "Cincinnati"

[[1]]
NULL
```
So what is going on here? If you query the classes of these types you can get more clarity

```R
class(alist[2])
class(alist[[2]])
```

Reveals what is going on behind the scenes

```
[1] "list"
[1] "numeric"
```
The list holds its elements inside a object of class list and our data is the first element in that object. Let us dig deeper into this by giving our list elements names

```R
alist.named <- list (cities=city.name, magic.number=12.4, TRUE, matrix.3x3=matrix(1:9, nrow=3))
alist.named
```
Shows the same list, but now we have named values

```
$cities
[1] "Columbus"   "Cleveland"  "Cincinnati"

$magic.number
[1] 12.4

[[3]]
[1] TRUE

$matrix.3x3
     [,1] [,2] [,3]
[1,]    1    4    7
[2,]    2    5    8
[3,]    3    6    9
```
This new naming allows us to also use the `$` operator to extract values like this: `alist.named$magic.number`. That is easy to read, yet shorter than typing `alist.named[["magic.number"]]`

If we re-run our earlier queries with `[]` and `[[]]` you will notice the one includes the name with the value, the other just the value at that index position

## Adding to a list

Adding to a list is simple. We use the `c()` combine function. 

```
alist.named.added <- c(alist.named, extra="Hello")
alist.named.added
```
You will see in the results that the new `alist.named.added$extra` value was added