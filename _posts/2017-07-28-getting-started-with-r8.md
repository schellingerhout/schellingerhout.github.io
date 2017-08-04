---
title: "Getting Started with R - Part 8: Data frames - Construction, Subsetting and Structure"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---
Vectors and matrices are designed to hold single types. When we need to mix data types in a tabular form we can use a data frame.
<!--more-->


I am posting this tutorial as I learn R. I will respond to feedback for errata in the comments.
{: .notice}


## What is a data frame?

A dataframe allows us to mix different values in a tabular form similar to what you might find in a database table. In statiscal context the columns are "data points" and the rows are "observations". If we think of a data frame as a database table then the columns are our "fields" and the rows are our our "records". Each column can have a different data type.

## Constructing a data frame

Most often dataframes are imported from csv (comma separated values), but you can create data frames inside of R. Here is a data frame with some basic types
```R
city.name <- c( "Columbus", "Cleveland", "Cincinnati")
latitude <- c(39.98, 41.48, 39.14)
longitude <-  c(-82.99, -81.68, -84.51)
population <- c(860090, 385809, 298800)
nickname <- c( "The Arch City", "America's North Coast", "The Queen City")
on.state.border <- c(F, T, T)

oh.city.df <- data.frame(city.name, longitude, latitude, population, nickname, on.state.border)
oh.city.df
```
 returns this nice dataframe

```
   city.name longitude latitude population              nickname on.state.border
1   Columbus    -82.99    39.98     860090         The Arch City           FALSE
2  Cleveland    -81.68    41.48     385809 America's North Coast            TRUE
3 Cincinnati    -84.51    39.14     298800        The Queen City            TRUE
```

## Indexing a data frame
Subsetting and selecting works very much like was saw with matrices, with a few execeptions I'll cover in a moment

```R
oh.city.df[,1:3] # Select city names and location
oh.city.df[3,] # Select all observations for city 3 (Cincinnati)
```

However, when we switch to single indexing we are essentially indexing by column. You'll remember that matrices when indexed on a single dimension essentially would traverse the data row-wise to locate an indexed value.

```R
oh.city.df["nickname"] #output the city nicknames
oh.city.df[1] # returns column 1 the city.name
oh.city.df[1:3] #returns city names and location
```

Try an out-of-bounds index. Notice how it returns an error unlike our vector and matrix that returns an `NA`. Here you will receive an error saying "undefined columns selected". Essentially single dimension indexing [index] is the same as [,index]. 

We can also select our data as vectors from our dataset using the following syntax

```R
oh.city.df$nickname
oh.city.df$population
oh.city.df$on.state.border
```
Try an invalid column name? Did you get what you expected? In this case the `$` does not throw an invalid column name error, but returns an `NA`.

BTW, did you notice something about the nickname's vector output? If not run this: `is.factor(oh.city.df$nickname)`. It was converted to a factor automatically!

## Naming rows
Looking at our dataframe it seems that the numbering of the rows are a waste, we'd rather just have our city names insted of numbers in those positions.

We could constructor our dataframe by telling it that one of the columns are actually our row names.

```R
oh.city.df <- data.frame(city.name, longitude, latitude, population, nickname, on.state.border, row.names=1)
oh.city.df
```

gives us this more readable dataframe:

```
           longitude latitude population              nickname on.state.border
Columbus      -82.99    39.98     860090         The Arch City           FALSE
Cleveland     -81.68    41.48     385809 America's North Coast            TRUE
Cincinnati    -84.51    39.14     298800        The Queen City            TRUE
```


**Careful!** When selecting a vector as names for the data frame removes the column's data. If you coded using indexes your indexes may have shifted
{: .notice--warning}

```R
oh.city.df["nickname"] #output the city nicknames
oh.city.df[1] # returns column 1 the logitude
oh.city.df[1:3] #returns city location and population
```

This is one reason it may better to use name based indexing, to prevent inadvertent errors with numeric indexes

```R
oh.city.df["Cincinnati", "nickname"]
oh.city.df["Columbus", c("longitude", "latitude", "on.state.border")]
```
As with matrices we have the full logic of matches to our disposal

```R
oh.city.df[oh.city.df$on.state.border,  ]
```

Give use the cities on the state border. The reason this simple logic works is that the `oh.city.df[ oh.city.df$on.state.border` is a vector of type logical and it works as a selector similar to what we saw in my earlier post on vectors. Lets try some of the other fields

```R
oh.city.df[ oh.city.df$longitude< 40,  ]
```
Returns all the fields for the cities south of 40.

```R
oh.city.df[ (oh.city.df$population > 300000) & (oh.city.df$population < 600000)
            , "population", drop=F] 
```
Returns the city that has a population between 300,000 and 600,000. Notice the optional drop paramter, I pass that to prevent the result to be converted to a vector. Try it without that to see what happens

## Structure information
Often, data frames are much larger than this dataset and viewing the data can be difficult. Often we just want to see a basic summary of information. To assist and getting a general overview of the data frame you can use the `str()` structure function

```R
str(oh.city.df)
```
oh.city.df[1:3] #returns city location and population


shows us the structure of our dataframe
```
'data.frame':	3 obs. of  5 variables:
 $ longitude      : num  -83 -81.7 -84.5
 $ latitude       : num  40 41.5 39.1
 $ population     : num  860090 385809 298800
 $ nickname       : Factor w/ 3 levels "America's North Coast",..: 2 1 3
 $ on.state.border: logi  FALSE TRUE TRUE
```

Observations are our rows, variables are our columns. We can see all our columns with a sample of their data (our data sample was small so all data was shown except for nickname). Lets say we want to change nickname back to "characters" type and we want population to be an integer, how can we do that? First lets find the source of the problem. First we'll examine our original vectors

```R
class(population)
class(nickname)
```
population was numeric to begin, but nickname was converted from characters to a factor. So doing this

```R
population <- as.integer(population) #convert from numeric to integer
oh.city.df <- data.frame(city.name, longitude, latitude, population, nickname, 
                          on.state.border, row.names=1)

str(oh.city.df)
```
This Fixes the population, but `as.characters(nickname)` does not change our input at all so it calling that in the constructor will not help. To fix this we need to pass `stringsAsFactors=F`. 

**Beware!** Passing  `as.integer(population)` as column set adds a column of NULLS with exactly that name. This is why I had to convert population *before* passing it to the data frame constructor
{: .notice--warning}


## Ordering Data Frames
At the end of [Part 3: Vector Operations](2017-07-24-getting-started-with-r3.md) I briefly discussed using the `order()` function on a vector to get a vector of indices in order. We can apply what we have learned there to data frames as well

```R
oh.city.df[ order(oh.city.df$latitude), ]
```
Gives us our cities ordered by latitude. This works simply becase the input to the rows in the selector is the ordered indices. If its not clear run `order(oh.city.df$latitude)` and notice that it returns a vector of values in the order 3, 1, 2