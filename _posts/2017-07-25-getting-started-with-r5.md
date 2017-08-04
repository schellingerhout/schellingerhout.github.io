---
title: "Getting Started with R - Part 5: Matrices - Creating, Filling and Subsetting"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---
Often we have the need to arrange data in a structure with rows and columns, or we need to represent a collection of vectors. R has a built-in matrix type that allows us to create a basic structure of rows and columns
<!--more-->


I am posting this tutorial as I learn R. I will respond to feedback for errata in the comments.
{: .notice}


## Constructing a matrix with the `matrix` function

Before we start, lets look at the basics in the online help by executing `?matrix`. Doing that reveals the signature of the function
```R
matrix(data = NA, nrow = 1, ncol = 1, byrow = FALSE,
       dimnames = NULL)
```
Reading further in the help you'll see we can supply a vector for the data, or use a type that can be coerced to a vector. We can specify the number of rows and columns and how the vector should be used to fill the matrix, and we can supply dimension names (which I'll cover a bit later). For now let us create a matrix

If we simply call `matrix()` we will get all the default parameters as specified with the `=` operator in definition of the help file. The `=` is very similar to the `<-` operator in that it is an assignment operator, but it can only be used in specific scope. So our simple `matrix()` call will create a 1 by 1 matrix with an `NA` value in it.

```R
matrix()
```
yields
```
     [,1]
[1,]   NA
```
 What on earth are those commas? Let us create a matrix with 1 row and 3 columns. You can see I can assign select variable like this `variable.name=value`, without the need to list them all in order.
 
**Note:** Here we use the `=` assignment operator to assign to a parameter, its scope in this case is limited to the call. We cannot get access to the value at the top level, by calling `ncol` after the the matrix function.
{: .notice--info}
 
 ```R
vectorlike.matrix <- matrix(ncol=3)
vectorlike.matrix
```
yields
```
     [,1] [,2] [,3]
[1,]   NA   NA   NA
```

Now we have some more clarity. Matrix values are indexed by rows then columns in this fashion `[row,column]`. To make this more clear let us add another row. To help us I need to introduce some new functions

## `rbind()` and `cbind()`

Simply put these add rows or columns to a matrix, or can combine matrices. lets add combine the `vectorlike.matrix` above with itself to create a two by three matrix.

```R
a.proper.matrix <- rbind(vectorlike.matrix, vectorlike.matrix)
a.proper.matrix
```
yields what we expect:

```
     [,1] [,2] [,3]
[1,]   NA   NA   NA
[2,]   NA   NA   NA
```
`rbind` and `cbind` are so similar that the online help shares a single entry for them. I'll let you explore `cbind` by yourself.

## Filling our matrix

When we created our matrix we didn't specify data at create time. We could have done so using a vector that would be unpacked row-wise or column-wise depending on the `byrow` parameter. The `byrow` parameter is FALSE if not specified so let us see what that looks like. Also, if we provide data we only need to specify either the column or row count. No need to do extra work

```R
two.by.three <- matrix(1:6, nrow=2)
two.by.three
```

We can see that the data was filled by column as expected
```
     [,1] [,2] [,3]
[1,]    1    3    5
[2,]    2    4    6
```

## Basic indexing
 The `[]` operator on a matrix looks suspiciously like the one we had for vectors so lets try it out. Try indexing from 1 to 6 on our `two.by.three` and you will see that we get our values by row, without specifying two coordinates. For instance `two.by.three[4]` yields 4. In case you are wondering, this has nothing to do with the `byrow` option during contruction. Single indexing is by row, regardless of how the data was unpacked. This is great news, there are many cases where the two coordinate indexing of matrices just get in the way. But, most of the time its still easier to index by row and column. 

 ```R
 two.by.three[2, 3]
 ```
returns the value 6 as expected. But what if we want to work with rows or columns and treat them as vectors? Our output above describing the matrix already revealed how this is done : simply ommit the dimension you want to include completely, for instance `[,2]` gives me all rows, and only column two. This is similar to what we saw in our previous lesson when we tried to set the entire vector's values. We wanted to set the entire vector's values without changing the vector, so we used empty brackets `[]` to select all values to be updated. We can do the same here

Say we want to fill our matrix, we can use the `rep()` function to repeat a value...
 ```R
 zero.matrix <- matrix(rep(0.0, 12), nrow=3) 
 ```
 ...or, we could have set all the values later with like this

 ```R
 fill.me.up <- matrix(nrow=3, ncol=4)
 fill.me.up[] <- 0
 fill.me.up
 ```

It would have been perfectly legal to use `fill.me.up[,] <- 0`. We can extend this idea to setting entire rows or columns as vectors:

```R
 fill.me.up[, 1] <- 1
 fill.me.up
 ```
 will set the entire first column as 1.

 We can also set a specific cell with basic indexing as you'd expect
 ```R
 fill.me.up[2, 1] <- 2
 fill.me.up
 ```

So far I've mainly shown the replace side of the extract \ replace `[]` operator. You can extract columns, rows, and cells similar to our replace operation. The only difference is that instead of setting values we yield them or assign them to other variables.

 ```R
 row.one <- fill.me.up[1, ]
 column.two <- fill.me.up[, 2]
 cell.3.2 <- fill.me.up[3, 2]
 fill.me.up[2, 2]
 ```

## Using the extract \ replace operator beyond just indexing
In [Part 4: Vector Extracting, Replacing and Excluding]({{ site.baseurl }}{% post_url 2017-07-24-getting-started-with-r4 %}), we saw that we can use combined integer indexes, for example `myvector[c(1,3)]` would select the first and third element. We can do the same here. To prevent repeated typing of the same value We can use the `rep()` function.

 ```R
three.by.three.identity.matrix <- matrix(rep(0.0, 9), nrow=3)
three.by.three.identity.matrix[c(1, 5, 9)] <-  1.0
three.by.three.identity.matrix
 ```
Here you can see an excelent example of how the single indexing of a matrix helps us... lets generalize that matrix a bit. I will use the `seq()` function to generate a sequence of numbers with a step.

  ```R
matrix.dim <- 10  
num.cells <- matrix.dim ^ 2
identity.matrix <- matrix(rep(0.0, num.cells), nrow=matrix.dim)
identity.matrix[seq(1, num.cells, matrix.dim + 1)] <- 1.0
identity.matrix
 ```

We are not restricted to using only single dimension syntax. We can apply index vector to the row or column side of the `,` inside the matrix `[]` operator. For instance

 ```R
matrix.dim <- 10  
x.matrix <- matrix(ncol=matrix.dim, nrow=matrix.dim)
x.matrix[] <- " "
x.matrix[c(3:5, 9), c(3, 7:9)] <- "X"
x.matrix[, c(1, 10)] <- "|"
x.matrix
```
 Gives us this output

 ```
     [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
 [1,] "|"  " "  " "  " "  " "  " "  " "  " "  " "  "|"  
 [2,] "|"  " "  " "  " "  " "  " "  " "  " "  " "  "|"  
 [3,] "|"  " "  "X"  " "  " "  " "  "X"  "X"  "X"  "|"  
 [4,] "|"  " "  "X"  " "  " "  " "  "X"  "X"  "X"  "|"  
 [5,] "|"  " "  "X"  " "  " "  " "  "X"  "X"  "X"  "|"  
 [6,] "|"  " "  " "  " "  " "  " "  " "  " "  " "  "|"  
 [7,] "|"  " "  " "  " "  " "  " "  " "  " "  " "  "|"  
 [8,] "|"  " "  " "  " "  " "  " "  " "  " "  " "  "|"  
 [9,] "|"  " "  "X"  " "  " "  " "  "X"  "X"  "X"  "|"  
[10,] "|"  " "  " "  " "  " "  " "  " "  " "  " "  "|"  
 ```

 Extracting a subset from a matrix is again similar to replacing values. We either yield the value or assign it. as we saw above.

 ```R
 double.x.pipe <- x.matrix[3:4, 9:10]
 double.x.pipe
 ```
shows that `double.x.pipe` is a nice sub-matrix

```
     [,1] [,2]
[1,] "X"  "|" 
[2,] "X"  "|" 
```

But, look at the rows and columns. The indices are based on their ordinal positions and not on the original source. We will address this next.