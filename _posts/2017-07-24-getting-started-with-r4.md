---
title: "Getting Started with R - Part 4: Vector Extracting"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---
Reaching beyond the basic indexing of Vectors we started in our previous post [Vectors Operations]({{ site.baseurl }}{% post_url 2017-07-24-getting-started-with-r3 %}) we will continue selecting, subsetting vectors beyond simple indexing.
<!--more-->


I am posting this tutorial as I learn R. I will respond to feedback for errata in the comments.
{: .notice}


## The Extract\Replace Operator
Some recap from earlier lessons

### Extract via index

In Part 2 of the tutorial we saw that we can use the `[]` operator similar to indexer in tradtional languages by providing a 1-based index. Like this:
```R
combined_integer <- c(1, 2, 4)
combined_integer[3]
```

But as we saw in Part 3, we need to think beyond our old ideas of indexing. We can just as well pass a vector to the `[]` operator. For instance
```R
combined_integer[c(1,3)]
```
will return the first and third values.

Often we want to select contiguous values. There is a quick constructor to help with that in the `:` operator. Try this
```R
1:10
```
That was essentially the same as calling `c(1,2,3,4,5,6,7,8,9,10)`

```R
cincinnati_rainfall <- c(2.87, 2.64, 3.82, 3.82, 4.72, 4.17, 3.86, 3.98, 3.11, 2.83, 3.31, 3.11)
names(cincinnati_rainfall) <- c("Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec")
cincinnati_rainfall[3:8]
```
Gives us a nice concise way to get our rainfall from March to August. Surely more convenient than typing `cincinnati_rainfall[c(3,4,5,6,7,8)]`

### Extracting via logical
In Part 3 we also learned that we can pass in a boolean vector to select. Just a recap:

We created two vectors
```R
weekday_names = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
predicted_precipitation <- c(0.0, 0.3, 0.01, 0.07, 0.0)
names(predicted_precipitation) = weekday_names
actual_precipitation <-  c(0.0, 0.27, 0.0, 0.08, 0.02)
names(actual_precipitation) = weekday_names
```
which we could then subset (extract) like this.
```R
more_than_predicted <- predicted_precipitation > actual_precipitation 
predicted_precipitation[more_than_predicted]
```

The logical values in the `more_than_predicted` vector acted as slector that was applied in a one-to-one fashion and where the selector value was `TRUE` the value at the applied index was returned

### Extracting by name
We game our indices names, so can we use them with the extract operator? Of course we can

```R
actual_precipitation["Wednesday"]
predicted_precipitation["Friday"]
```

This is certainly very nice and readable.

**Self Study**: Use a vector of characters to select Wednesday and Friday's actual precipitation. 
{: .notice--info}

