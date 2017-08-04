---
title: "Getting Started with R - Part 7: Factors - Levels and Labels"
excerpt.separator: "<!--more-->"
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

If we have a vector of values where the values can be only one of a few values it becomes a good candidate for a factor. In essence we will convert a vector into a new kind of vector we call a factor that internally has integer values and labels associated with each integer value. 

```R
repeat.vector <- c('I', 'often', 'repeat', 'repeat', 'myself', 'I', 'often', 'repeat', 'repeat') # Jack Prelutsky
repeat.factor <- factor(repeat.vector)
repeat.factor
```
yields
```
[1] I      often  repeat repeat myself I      often  repeat repeat
Levels: I myself often repeat
```
In this case there were four levels for the vector strings: `I`, `myself`, `often` and `repeat`. The levels were simply created by taking the unique strings in the vector, and then sorting them. We could have also provided `levels` during the factor's construction. A little later I will show you how to change the ordinal positions of the levels. To see the levels of a factor you can simply call `levels(afactor)` to get the list of factors in order.

**Notice!** The `levels` are used to class the values into factors. We can also pass `labels` to the factor constructor, if you do this and query the `levels` of the resulting factor you will see the values you passed in `labels`. During construction `levels` are used for classing, `labels` are for naming the levels in the resulting factor *differently* than from the levels used for mapping during construction. 
{: .notice--info}

Let us take a look at the attributes of the factor object

```R
attributes(repeat.factor)
```
yields

```
$levels
[1] "I"      "myself" "often"  "repeat"

$class
[1] "factor"
```
I'll get to these attributes in a moment, but first let us use `labels` to change the level names in our new factor. We will also pass the associated `levels` for classing the input. Again, please do not confuse these two parameters. Levels are matched against input to categorize, it will also be used to name the levels except if we pass `labels`. Once the vector is constructed with `lablels` the levels names passed in constructor are the names of my levels. 

```R
repeat.factor.labeled <- factor(repeat.vector, levels = c("I", "often", "repeat", "myself"), 
                                               labels = c("Jack", "frequently", "repeats", "himself" ) )
repeat.factor.labeled
```
returns this
```
[1] Jack       frequently repeats    repeats    himself    Jack       frequently
[8] repeats    repeats   
Levels: Jack frequently repeats himself
```

You may have noticed that when I listed the levels I swapped `"repeat"` and `"myself"` for the previously sorted order (look at how I ordered them in the `levels=`). You can see that the levels in the new factor are also listed in the new order, but with the new names. Run `attributes(repeat.factor.labeled)` and you will see that the difference. Compare the result of the `attributes` function to when you ran it before against `repeat.factor`.

## Changing levels

You can set the levels labels after constructing a factor. This would be similar to passing in the `labels` parameter. We can pass a full new vector or just labels the labels of the levels selectively. Let us just change factor label 1 from "Jack" to "Mr. Prelutsky".

```R
levels(repeat.factor.labeled)[1] <- "Mr. Prelutsky"
repeat.factor.labeled
```

There are some advanced actions you can take to combine levels in a factor using the `levels()` function.  See [Cleaning up factor levels (collapsing multiple levels/labels)](https://stackoverflow.com/questions/19410108/cleaning-up-factor-levels-collapsing-multiple-levels-labels) for some advanced factor level cleanup.

## Ordered factors

We may have factors where the ordinal positition of the values are not important. In the example above the factors for the words above do not have a specific order to them. We can change that by adding the `order=TRUE` optional parameter to the constructor. First let us find a factor where we have order

```R
days.of.week = c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday")
days.of.week.factor <- factor(days.of.week, order=TRUE, levels=days.of.week)
days.of.week.factor
```
shows us that levels have order, by displaying  `<`  between them
```
[1] Sunday    Monday    Tuesday   Wednesday Thursday  Friday    Saturday 
Levels: Sunday < Monday < Tuesday < Wednesday < Thursday < Friday < Saturday
```
Since my vector was already unique and in the correct order I could use it for my levels. If I didn't my levels would just be ordered alphabetically. If I need to take a subset of a factor I can do the same as with vectors

```R
tuesday <- days.of.week.factor[3] 
tuesday
days.of.week.factor[c(2:7, 1)] # days of the week starting with Monday instead of Sunday
```

Ordered vectors allow us to do inequalities in expressions
```R
days.of.week.factor[3] > days.of.week.factor[1]
sunday <- days.of.week.factor[1]
days.of.week.factor[days.of.week.factor > sunday]
```

Try the same code above but omit the `order=TRUE`, you'll get a vector of `NA`s. To understand why compare two days `days.of.week.factor[2] > days.of.week.factor[1]`. There is no order so `>` has no meaning leading to an `NA` result

## Factors and matrices

All is nice in our factor space, but let us try and use factors in our matrix

```R
#sunday <- days.of.week.factor[1] 

month.names <- c('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec')
calendar.matrix <- matrix(nrow=31, ncol=12, dimnames = list(1:31, month.names))
months.31days <- c(1,3,5,7,8,10,12)
months.30days <- c(4,6,9,11)
calendar.matrix[1:31, months.31days] <- sunday
calendar.matrix[1:30, months.30days] <- sunday
calendar.matrix[1:28, 'Feb'] <- sunday
# Invalid calendar positions are now NA, valid ones are are the factor value of Sunday

oldw <- getOption("warn")
# We have to turn off a warning because the days of the week do not even divide - there is a remainder of 1
options(warn = -1) 
calendar.matrix[!is.na(calendar.matrix)] <- days.of.week.factor[c(2:7, 1)] # assuming Jan 1 fell on a Monday
options(warn = oldw)

calendar.matrix
```

Did you get what you expected? You see that the days of the week are populated by the ordinal values. This gives us a little peak of what is really going on inside of the factor: We have the integer values that simply index their names. Matrices are really not factors and giving them factors makes them store ordinal values. We will learn of another structure called a dataframe later that will make some of these complex scenarios simpler

What if we wanted to display our matrix with the labels of the factor? That is a bit tricky so here is the step by step breakdown. We can get our level names of our factor like this: `levels(days.of.week.factor)` and we can index the values

```R
levels(days.of.week.factor)[2] #Gives us "Monday"
levels(days.of.week.factor)[calendar.matrix] #Gives use the strings of the calendar.matrix, but as a vector
```

So we can get the calendar's values in 1-dimensional form so how can we get it back as a matrix. Simply we assign it to matrix with the same shape.

```R
matrix(levels(days.of.week.factor)[calendar.matrix], nrow=31, ncol=12, dimnames = list(1:31, month.names))
```
We could have also made a copy of our other calendar matrix and assign, like this

```R
calendar.matrix.names <- calendar.matrix
calendar.matrix.names[]  <- levels(days.of.week.factor)[calendar.matrix]
calendar.matrix.names
```