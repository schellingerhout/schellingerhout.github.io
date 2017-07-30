---
title: "Getting Started with R - Part 7: Factors - Levels and Labels"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---
Factors are like vectors but with values classed into levels. When vectors have a limited number of repeated values they fit the bill.
<!--more-->


I am posting this tutorial as I learn R. I will respond to feedback for errata in the comments.
{: .notice}


## How to create a factor

If we have a vector of values where the values can be only one of a few values it becomes a good candidate for a factor. I essence we will convert a vector into a new kind of vector we call a factor that internally has integer values and labels associated with each integer value. 

```R
repeat_vector <- c('I', 'often', 'repeat', 'repeat', 'myself', 'I', 'often', 'repeat', 'repeat') # Jack Prelutsky
repeat_factor <- factor(repeat_vector)
repeat_factor
```
yields
```
[1] I      often  repeat repeat myself I      often  repeat repeat
Levels: I myself often repeat
```
In this case there were four levels for the vector strings: `I`, `myself`, `often` and `repeat`. The levels were simply created by taking the unique strings in the vector, and then sorting them. We could have also provided `levels` during the factor's construction. A little later I will show you how to change the ordinal positions of the levels. To see the levels of a factor you can simply call `levels(afactor)` to get the list of factors in order.

**Notice!** The `levels` are used to class the values into factors. We can also attach `labels` to the factor, if you do this and query the `levels` of the factor you will see the values you passed in `labels`. During construction `levels` are used for classing, `labels` are for naming the levels in the resulting factor *differently* than from the levels used for mapping during construction. 
{: .notice--info}

Let us take a look at the attributes of the factor object

```R
attributes(repeat_factor)
```
yields

```
$levels
[1] "I"      "myself" "often"  "repeat"

$class
[1] "factor"
```
Let us use `labels` to change our factor, and I will also pass the associated `levels` for classing the input.

```R
repeat_factor_labeled <- factor(repeat_vector, levels = c("I", "often", "repeat", "myself"), 
                                               labels = c("Jack", "frequently", "repeats", "himself" ) )
repeat_factor_labeled
```
returns this
```
[1] Jack       frequently repeats    repeats    himself    Jack       frequently
[8] repeats    repeats   
Levels: Jack frequently repeats himself
```

You may have noticed that when I listed the levels I swapped `"repeat"` and `"myself"` for the previously sorted order (look at how I ordered them in the `levels=`). You can see that the levels in the new factor are also listed in the new order, but with the new names. Run `attributes(repeat_factor_labeled)` and you will see that the difference.

## Changing levels

You can set the levels labels after constructing a factor. This would be similar to passing in the `labels` parameter. We can pass a full new vector or just labels the labels of the levels selectively. Let us just change factor label 1 from "Jack" to "Mr. Prelutsky".

```R
levels(repeat_factor_labeled)[1] <- "Mr. Prelutsky"
repeat_factor_labeled
```


