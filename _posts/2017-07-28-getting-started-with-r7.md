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

**Notice!** The `levels` are used to class the values into factors. We can also pass `labels` to the factor constructor, if you do this and query the `levels` of the resulting factor you will see the values you passed in `labels`. During construction `levels` are used for classing, `labels` are for naming the levels in the resulting factor *differently* than from the levels used for mapping during construction. 
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

## Ordered factors

We may have factors where the ordinal positition of the values are not important. In the example above the factors for the words above do not have a specific order to them. We can change that by adding the `order=TRUE` optional parameter to the constructor. First let us find a factor where we have order

```R
days_of_week = c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday")
days_of_week_factor <- factor(days_of_week, order=TRUE, levels=days_of_week)
days_of_week_factor
```
shows us that levels have order, by displaying  `<`  between them
```
[1] Sunday    Monday    Tuesday   Wednesday Thursday  Friday    Saturday 
Levels: Sunday < Monday < Tuesday < Wednesday < Thursday < Friday < Saturday
```
Since my vector was already unique and in the correct order I could use it for my levels. If I didn't my levels would just be ordered alphabetically. If I need to take a subset of a factor I can do the same as with vectors

```R
tuesday <- days_of_week_factor[3] 
tuesday
days_of_week_factor[c(2:7, 1)] # days of the week starting with Monday instead of Sunday
```

Ordered vectors allow us to do inequalities in expressions
```R
sunday <- days_of_week_factor[1]
days_of_week_factor[days_of_week_factor > sunday]
```


## Factors and matrices

All is nice in our factor space, but let us try and use factors in our matrix

```R
#sunday <- days_of_week_factor[1] 

month_names <- c('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec')
calendar_matrix <- matrix(nrow=31, ncol=12, dimnames = list(1:31, month_names))
months_31days <- c(1,3,5,7,8,10,12)
months_30days <- c(4,6,9,11)
calendar_matrix[1:31, months_31days] <- sunday
calendar_matrix[1:30, months_30days] <- sunday
calendar_matrix[1:28, 'Feb'] <- sunday
# Invalid calendar positions are now NA, valid ones are are the factor value of Sunday

oldw <- getOption("warn")
# We have to turn off a warning because the days of the week do not even divide - there is a remainder of 1
options(warn = -1) 
calendar_matrix[!is.na(calendar_matrix)] <- days_of_week_factor[c(2:7, 1)] # assuming Jan 1 fell on a Monday
options(warn = oldw)

calendar_matrix
```

Did you get what you expected? You see that the days of the week are populated by the ordinal values. This gives us a little peak of what is really going on inside of the factor: We have the integer values that simply index their names. Matrices are really not factors and giving them factors makes them store ordinal values. We will learn of another structure called a dataframe later that will make some of these complex scenarios simpler

What if we wanted to display our matrix with the labels of the factor? That is a bit tricky so here is the step by step breakdown. We can get our level names of our factor like this: `levels(days_of_week_factor)` and we can index the values

```R
levels(days_of_week_factor)[2] #Gives us "Monday"
levels(days_of_week_factor)[calendar_matrix] #Gives use the strings of the calendar_matrix, but as a vector
```

So we can get the calendar's values in 1-dimensional form so how can we get it back as a matrix. Simply we assign it to matrix with the same shape.

```R
matrix(levels(days_of_week_factor)[calendar_matrix], nrow=31, ncol=12, dimnames = list(1:31, month_names))
```
We could have also made a copy of our other calendar matrix and assign, like this

```R
calendar_matrix_names <- calendar_matrix
calendar_matrix_names[]  <- levels(days_of_week_factor)[calendar_matrix]
calendar_matrix_names
```