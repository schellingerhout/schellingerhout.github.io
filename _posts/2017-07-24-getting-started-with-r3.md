---
title: "Getting Started with R - Part 3: Vector Operations"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---
Expanding on our [Vectors Basics]({{ site.baseurl }}{% post_url 2017-07-24-getting-started-with-r2 %}) we will continue with operations accross vectors.
<!--more-->


I am posting this tutorial as I learn R. I will respond to feedback for errata in the comments.
{: .notice}


## Vector Operations

We already looked at combining vectors with the `c()` function. What would have happened if we used `+` instead ?

```R
probability_of_rain_work <- c(0.8, 0.2, 0.05, 0.4, 0.65)
probability_of_rain_play <- c(0.1, 0.0)
probability_of_rain_work+probability_of_rain_play
```

Yields an odd result along with a warning

```
   Monday   Tuesday Wednesday  Thursday    Friday 
     0.90      0.20      0.15      0.40      0.75 
Warning message:
In probability_of_rain_play + probability_of_rain_work :
  longer object length is not a multiple of shorter object length
```
Looks like Saturday and Sundays values were added to Monday and Tuesday, and again to Wednesday and Thursday. Finally Saturday's value was added to Friday. **We did not want this** and that is why we used `c()` in our previous post.

We did learn something from the result and message above. The smaller vector was repeatedly applied to the larger vector, almost like a [rack and pinion gear](https://en.wikipedia.org/wiki/Rack_and_pinion) rolling. Per the error message above, one vector needs to be at least a multiple of the other to avoid a warning. This makes sense because otherwise we'd have an remainder of unapplied values.

Let us see what happens if one vector is a multiple of another, so lets add a 3 value vector to a 12 value vector:

``` R
cincinnati_rainfall <- c(2.87, 2.64, 3.82, 3.82, 4.72, 4.17, 3.86, 3.98, 3.11, 2.83, 3.31, 3.11)
names(cincinnati_rainfall) <- c("Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec")
quarter_irrigation <- c(1, 3, 0)
cincinnati_rainfall + quarter_irrigation
```
yields a result showing that the quater irrigation values are cycled over the larger vector and then added. 
```
 Jan  Feb  Mar  Apr  May  Jun  Jul  Aug  Sep  Oct  Nov  Dec 
3.87 5.64 3.82 4.82 7.72 4.17 4.86 6.98 3.11 3.83 6.31 3.11 
```

From this it should be obvious what happens when the vectors are exactly the same length.


Lets test that with the `-` operator

```R
weekday_names = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
predicted_precipitation <- c(0.0, 0.3, 0.01, 0.07, 0.0)
names(predicted_precipitation) = weekday_names
actual_precipitation <-  c(0.0, 0.27, 0.0, 0.08, 0.02)
names(actual_precipitation) = weekday_names
predicted_precipitation-actual_precipitation
```

Shows a nice element-wise difference vector

```
   Monday   Tuesday Wednesday  Thursday    Friday 
     0.00      0.03      0.01     -0.01     -0.02 
```

We are not limited to just mathematical operators we can also use logical operators like this ...
```R
actual_precipitation > 0.0
```
... which yields a result as if each element was compared against 0.0. In essence we applied the `>` operator against a vector of length 5 and a vector of length 1. Since 5 is a multiple of 1 the comparison was applied to each element.

```
   Monday   Tuesday Wednesday  Thursday    Friday 
    FALSE      TRUE     FALSE      TRUE      TRUE 
```

Using this same principle I can do the following just as easily!

```R
predicted_precipitation > actual_precipitation
```

In this case we again have an exact same number of elements so the operator applies as if comparing corresponding elements between the two arrays, similar to how we added elements between arrays

```
   Monday   Tuesday Wednesday  Thursday    Friday 
    FALSE      TRUE      TRUE     FALSE     FALSE 
```

Its easy to think of `[]` as a simple indexer, but it is really a much more powerful operator. For instance: Notice how `predicted_precipitation > actual_precipitation` 
returned a vector of the same length filled with logical values. We can use those as a selector, making this possible


```R
more_than_predicted <- predicted_precipitation > actual_precipitation 
predicted_precipitation[more_than_predicted]
```

yields

```
  Tuesday Wednesday 
     0.30      0.01 
```

I could also have used `predicted_precipitation[predicted_precipitation > actual_precipitation]`, but sometimes readability trumps short code.

From your new found knowledge, can you predict the result of the following?

```R
cincinnati_rainfall[c(T,F)]
```
Do you understand why you got the result? If not, please re-read the section above, or post in the comment section below.

I will cover other ways to use the `[]` Extract\Replace operator in the next post.

## Vector functions

Let me leave you with some of the many built in functions in R for vectors

```R
length(predicted_precipitation)
mean(predicted_precipitation)
median(predicted_precipitation)
max(predicted_precipitation)
min(predicted_precipitation)
```

These operate accross the vector and return a single value. I'll let you use the help system to read up on those if you are not familiar with the difference between `mean` and `median`

One very useful function for vectors is order

```R
actual_precipitation <-  c(0.0, 0.27, 0.0, 0.08, 0.02)
order(actual_precipitation)
```
Returns the indices in order as a vector. We can of course use that vector to index the vector itself

```R
actual_precipitation <-  c(0.0, 0.27, 0.0, 0.08, 0.02)
actual_precipitation[ order(actual_precipitation) ]
```