---
title: "Getting Started with R - Part 5: Matrices - Captions"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---
Adding Captions to matrices make them more readable and allows us to retain information when we subset the matrix.
<!--more-->


I am posting this tutorial as I learn R. I will respond to feedback for errata in the comments.
{: .notice}


## Adding columns after construction

You should remember how we added names to our vector values. In that case we used the `names()` function. If you need a second look, then head back to [Part 2: Vector Basics]({{ site.baseurl }}{% post_url 2017-07-24-getting-started-with-r2 %}).

We have two similar functions for matrices: `rownames` and `colnames`. Let us retry our previous subset

 ```R
matrix_dim <- 10  
x_matrix <- matrix(ncol=matrix_dim, nrow=matrix_dim)
rownames(x_matrix) <- 1:matrix_dim
colnames(x_matrix) <- 1:matrix_dim
x_matrix[] <- " "
x_matrix[c(3:5, 9), c(3, 7:9)] <- "X"
x_matrix[, c(1, 10)] <- "|"
x_matrix
x_matrix[3:4, 9:10]
```
 
 This gives us a more understandable result:

```
    1   2   3   4   5   6   7   8   9   10 
1  "|" " " " " " " " " " " " " " " " " "|"
2  "|" " " " " " " " " " " " " " " " " "|"
3  "|" " " "X" " " " " " " "X" "X" "X" "|"
4  "|" " " "X" " " " " " " "X" "X" "X" "|"
5  "|" " " "X" " " " " " " "X" "X" "X" "|"
6  "|" " " " " " " " " " " " " " " " " "|"
7  "|" " " " " " " " " " " " " " " " " "|"
8  "|" " " " " " " " " " " " " " " " " "|"
9  "|" " " "X" " " " " " " "X" "X" "X" "|"
10 "|" " " " " " " " " " " " " " " " " "|"

  9   10 
3 "X" "|"
4 "X" "|"

```

Of course we are not limited to just numeric ranges for captions. Just like with our vectors we can also use characters.

## Adding captions during matrix construction

To add the captions during construction I can use the dimnames parameter. This paramter takes a list with two vectors: The row captions, then the column captions

```R
month_names <- c('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec')
calendar_matrix <- matrix(nrow=31, ncol=12, dimnames = list(1:31, month_names))

calendar_matrix[1:31, months_31days]

months_31days <- c(1,3,5,7,8,10,12)
months_30days <- c(4,6,9,11)
calendar_matrix[1:31, months_31days] <- ''
calendar_matrix[1:30, months_30days] <- ''
calendar_matrix[1:28, 'Feb'] <- ''
# Invalid calendar positions are now NA, valid ones are an empty string

oldw <- getOption("warn")
  # We have to turn off a warning because the days of the week do not even divide - there is a remainder of 1
  options(warn = -1) 
  calendar_matrix[!is.na(calendar_matrix)] <- c('Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun') # assuming Jan 1 fell on a Monday
options(warn = oldw)

calendar_matrix
```

If you know of a way I can simplify the selection, please comment below so I can learn and share with others. For those that are curious, here is the matrix output
```
   Jan   Feb   Mar   Apr   May   Jun   Jul   Aug   Sep   Oct   Nov   Dec  
1  "Mon" "Thu" "Thu" "Sun" "Tue" "Fri" "Sun" "Wed" "Sat" "Mon" "Thu" "Sat"
2  "Tue" "Fri" "Fri" "Mon" "Wed" "Sat" "Mon" "Thu" "Sun" "Tue" "Fri" "Sun"
3  "Wed" "Sat" "Sat" "Tue" "Thu" "Sun" "Tue" "Fri" "Mon" "Wed" "Sat" "Mon"
4  "Thu" "Sun" "Sun" "Wed" "Fri" "Mon" "Wed" "Sat" "Tue" "Thu" "Sun" "Tue"
5  "Fri" "Mon" "Mon" "Thu" "Sat" "Tue" "Thu" "Sun" "Wed" "Fri" "Mon" "Wed"
6  "Sat" "Tue" "Tue" "Fri" "Sun" "Wed" "Fri" "Mon" "Thu" "Sat" "Tue" "Thu"
7  "Sun" "Wed" "Wed" "Sat" "Mon" "Thu" "Sat" "Tue" "Fri" "Sun" "Wed" "Fri"
8  "Mon" "Thu" "Thu" "Sun" "Tue" "Fri" "Sun" "Wed" "Sat" "Mon" "Thu" "Sat"
9  "Tue" "Fri" "Fri" "Mon" "Wed" "Sat" "Mon" "Thu" "Sun" "Tue" "Fri" "Sun"
10 "Wed" "Sat" "Sat" "Tue" "Thu" "Sun" "Tue" "Fri" "Mon" "Wed" "Sat" "Mon"
11 "Thu" "Sun" "Sun" "Wed" "Fri" "Mon" "Wed" "Sat" "Tue" "Thu" "Sun" "Tue"
12 "Fri" "Mon" "Mon" "Thu" "Sat" "Tue" "Thu" "Sun" "Wed" "Fri" "Mon" "Wed"
13 "Sat" "Tue" "Tue" "Fri" "Sun" "Wed" "Fri" "Mon" "Thu" "Sat" "Tue" "Thu"
14 "Sun" "Wed" "Wed" "Sat" "Mon" "Thu" "Sat" "Tue" "Fri" "Sun" "Wed" "Fri"
15 "Mon" "Thu" "Thu" "Sun" "Tue" "Fri" "Sun" "Wed" "Sat" "Mon" "Thu" "Sat"
16 "Tue" "Fri" "Fri" "Mon" "Wed" "Sat" "Mon" "Thu" "Sun" "Tue" "Fri" "Sun"
17 "Wed" "Sat" "Sat" "Tue" "Thu" "Sun" "Tue" "Fri" "Mon" "Wed" "Sat" "Mon"
18 "Thu" "Sun" "Sun" "Wed" "Fri" "Mon" "Wed" "Sat" "Tue" "Thu" "Sun" "Tue"
19 "Fri" "Mon" "Mon" "Thu" "Sat" "Tue" "Thu" "Sun" "Wed" "Fri" "Mon" "Wed"
20 "Sat" "Tue" "Tue" "Fri" "Sun" "Wed" "Fri" "Mon" "Thu" "Sat" "Tue" "Thu"
21 "Sun" "Wed" "Wed" "Sat" "Mon" "Thu" "Sat" "Tue" "Fri" "Sun" "Wed" "Fri"
22 "Mon" "Thu" "Thu" "Sun" "Tue" "Fri" "Sun" "Wed" "Sat" "Mon" "Thu" "Sat"
23 "Tue" "Fri" "Fri" "Mon" "Wed" "Sat" "Mon" "Thu" "Sun" "Tue" "Fri" "Sun"
24 "Wed" "Sat" "Sat" "Tue" "Thu" "Sun" "Tue" "Fri" "Mon" "Wed" "Sat" "Mon"
25 "Thu" "Sun" "Sun" "Wed" "Fri" "Mon" "Wed" "Sat" "Tue" "Thu" "Sun" "Tue"
26 "Fri" "Mon" "Mon" "Thu" "Sat" "Tue" "Thu" "Sun" "Wed" "Fri" "Mon" "Wed"
27 "Sat" "Tue" "Tue" "Fri" "Sun" "Wed" "Fri" "Mon" "Thu" "Sat" "Tue" "Thu"
28 "Sun" "Wed" "Wed" "Sat" "Mon" "Thu" "Sat" "Tue" "Fri" "Sun" "Wed" "Fri"
29 "Mon" NA    "Thu" "Sun" "Tue" "Fri" "Sun" "Wed" "Sat" "Mon" "Thu" "Sat"
30 "Tue" NA    "Fri" "Mon" "Wed" "Sat" "Mon" "Thu" "Sun" "Tue" "Fri" "Sun"
31 "Wed" NA    "Sat" NA    "Thu" NA    "Tue" "Fri" NA    "Wed" NA    "Mon"
```
