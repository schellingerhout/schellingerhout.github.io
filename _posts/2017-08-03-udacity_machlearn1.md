---
title: "Notes on Udemy Class: R, ggplot, and Simple Linear Regression"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - ggplot
  - Udemy
---
Working through a free course at Udemy [R, ggplot, and Simple Linear Regression](https://www.udemy.com/machlearn1/learn/v4/overview). My notes will follow
<!--more-->

I am posting this tutorial as I learn R. I will respond to feedback for errata in the comments.
{: .notice}

I will not address any issues that we have covered in the getting started section or the [Kaggle Titanic tutorial follow-along]({{ site.baseurl }}{% post_url 2017-07-31-datacamp-kaggle-tutorial %}). 

To follow along with this blog post you can head to Udemy [R, ggplot, and Simple Linear Regression](https://www.udemy.com/machlearn1/learn/v4/overview). Register and and follow along with the free class.

## Getting Started
### Introduction
Dr. Charles Redmond has a series of R videos at Udemy. In the introduction he gives a motivation that he wants to bring data science under other disciplines. His goal is understanding linear regression. I am already familiar with linear regression, but a refresher is good.

This course is provided for free and I don't want to be too harsh, but I will point out shortcomings and errors in the tutorial. Anyone that makes a course and provides it for free needs to be commended, even if the course or material is not perfect. This is why I always ask for feedback. I will use the feedback and adjust and fix my material as needed

### Installing R and R-Studio
Some basics of R and the interface R-Studio. Nothing new for now. Getting R at http://wwww.r-project.org (I usually go to https://cran.r-project.org/ directly). He recommends the base distribution. Also recommends R-Studio at http://www.rstudio.com. 

### A Tour of R-Studio
Talking about the panels in R-Studio and how to use the "restore-down" button on the console to review an empty script panel. I've always just clicked new, didn't even know there was a ready text panel.

He likes to use the console to do his interactive work and the top panel for scripting. He mentions that he will rarely use the "Environment" panel. I actually use it quite a bit to quickly inspect values. Its quite handy to use it to click a dataframe and view the content in a nice spreadsheet-like table. 

He also mentions the Ctrl-L hotkey for clearing the console. Expanding the menus at the top will reveal other shortcuts. I accidentaly found Ctrl-L trying to delete a line in a script file. Many text editors - including Notepad++ - uses that for clearing a single line. 

### Vectors 
I already covered vectors in [Getting Started with R - Part 2: Vector Basics]({{ site.baseurl }}{% post_url 2017-07-24-getting-started-with-r2 %}) and [Getting Started with R - Part 3: Vector Operations]({{ site.baseurl }}{post_url 2017-07-24-getting-started-with-r3 %}). Notice that he typed `X<-3` in the console, this is **not** acceptable formatting because it can be confused for "Is x less than minus 3?".

He shows how to use the combine function to create a vector using `c()` and basic indexing with the `[]`. His showing of slicing with `:` operator hides the real dynamic of the the `:` operator. This operator is really a range generator :- essentially creating a series and that the series is actually then the vector for indexing. I covered this in [Getting Started with R - Part 4: Vector Extracting, Replacing and Excluding]({ site.baseurl }}{% post_url 2017-07-24-getting-started-with-r4 %}). For example: `-2:-6` is really just `c(-2, -3, -4, -5, -6)`. Since indices with a `-` are simply excluded.

This lesson glosses over important concepts of the `[]` Extract\Replace operator. Please understand how it works early on to save you headaches later

### Data Frames
He skips lists and matrices and move to data frames. Nothing much new here. Immediatly jumps into indexing by name with the `$` operator. 

#### Style
A note on the lecturer starting a data frame with a capital letter. Dataframes are identifiers and therefore I think it is odd to use a different style over any other identifier.

There is no universal R Style guide and even Google's own style guide to some extent does not play well with much of the CRAN Repository. In case you wonder; Google's style guide prefers `.` between lowercase words in an identifier like `my.vector.of.temps` over camelCase (`myVectorOfTemps`) and advises against underscores `my_vector_of_temps`. It also recommends function names be in PascalCase like this `MyFunction`. Other styleguides (like Hadley Wickham's) recommend undescores between words in identifiers the way you see me use it, but he advises against dots in function names. However, the CRAN is littered with lots of libraries that use `.`. 

I don't think the Google styleguide is not a good choice, I am confused that they did not go with Wicham's style. But even a bad choice can help a team work in a consistent style. So your team will need to decide on a style. I think Wickham's style guide is superior, but even I cherry pick what I want from it. 

There are a few things that are almost universal required for basic R style:
* Spaces either sides of binary operators (with the exception of the `:` operator),
* spaces after commas. Also, 
* you want to add an space between anything that is not a function (keyword, unary operator, identifier) and an opening paren like this `if (a_logical)`, because otherwise it looks like a function.

## Working with ggplot
### installing ggplot2
He shows how to install a package using the "Tools, Install Packages..." menu and then typing the name. I usually just type something like `install.packages("ggplot2")` in the console. Why? Because when you work on most projects the package is often included in the script (sometimes the `install.packages()` command is just commented out). So usually copy and paste or uncomment the line in the script and be done.

He mentions that once the cursor stops you know the package is installed. There is an easier way to see if the package is installed. Look to the right panel where it shows "Files, Plots, **Packages**, Help, viewer". Simply see if the package is listed under the Packages tab.

He does mention the `library()` function we saw in the [Kaggle Titanic tutorial follow-along]({{ site.baseurl }}{% post_url 2017-07-31-datacamp-kaggle-tutorial %}). In this case its a trivial `library(ggplot2)`.

Just a note on CRAN. I've had trouble getting packages to install. Sometimes the servers are just too busy, and often you just have to try at a later time. I don't know of any trustworth repositories outside of CRAN. 

Another way to get ggplot2 along with the rest of a family of packages call the "tidyverse"; you can simply install using `install.packages('tidyverse')`. The tidyverse is a bit more than you need for this tutorial, but is very handy for data science in general

### Plotting a point with ggplot
Yes, the package is `ggplot2` and the function is `ggplot`. 

The `ggplot()` constructs the inital plot object often given a default dataset and aesthetics. The function is almost always followed by the `+` operator to add components to the plot. The sample created here was just a simple skeletal plot to which a geometry point layer was added.

```R
# code like this: spaces on either side of infix operators (=, <-, +) and spaces after commas
ggplot() + geom_point(data = dat, mapping = aes(x = x, y = y), color = "blue")  
```
You'll notice that he is supplying the aesthetics opject without a parameter name. Its actually the second parameter so if he wanted to omit the name he should have had this format, otherwise specifify the name "mapping"

```R
ggplot() + geom_point( aes(x = x, y = y), data = dat, color = "blue") 
```

The `ggplot` constructor has three basic forms. 
* `ggplot(data, mapping)`, useful if all layers will use the same data and aesthetics
* `ggplot(data)`, when layers share data, but vary in aesthetics; and
* `ggplot()`, where layers control data and aesthetics

So the following will yield the exact same result

```R
ggplot(dat, aes(x = x, y = y)) + geom_point(size = 5, color = "blue")
```

The data is the first parameter in ggplot, and mapping is the second so I can supply `dat` and `aes`, without variable names.  Just note that the two parameters are reversed in the `geom_point` and `geom_line` functions. So most samples in this Udemy class are technically wrong since the aesthetics object is passed without a parameter name ever time as the second parameter into these functions. There may be an error handling mechanism that correctly assings the aesthetics option to the "mapping" variable, but technically these calls should have produced erros.

His explanation of assigning x to x and y to y is a bit brief. Here is deal: the `=` operator assign at current context and lower, while `<-` assigns in the global context. Here the `x` on the left is in defined inside the context of the function and the `x` on the right is on the global context. In this case the x on the right will reference `dat$x`. Since the console context cannot see what is assigned and cannot even see the `x` on the left this is a perfectly fine assignment. This type of assignment is most commonly used to pass parameters by name.

Just a note. Since the first two parameters of the `aes` objects are so commonly used it is common to drop the parameter names. Pretty much everyone knows they are x and y.

### Controlling axis properties

In this section he creates another empty ggplot and adds the geometry points to the plot like this

```R
ggplot() + geom_point(data = dat, mapping = aes(x = x, y = y), size =  5, color = "blue" )
```
If you read my notes on the previous section you'll know that if he wanted to ommit the `mapping =` then the parameter should have been the first one in `geom_point`. The documentation has more on this

```R
ggplot() + geom_point(data = dat, mapping = aes(x = x, y = y), size =  5, color = "blue" )
# or
ggplot() + geom_point(aes(x = x, y = y), data = dat, size =  5, color = "blue" )
# or simplified
ggplot() + geom_point(aes(x, y), dat, size =  5, color = "blue" )
# or prefereably having the data and aesthethics at the highest level possible
ggplot(data = dat, mapping = aes(x, y)) + geom_point(size =  5, color = "blue" )
```

He mentions that he has already loaded ggplot2 into the session. An easy way to do that is to go to the packages tab on the right and check the box next to ggplot. That will automatically execute `library("ggplot2")`

### `scale_*_continuous` and other continuous scales

`scale_x_continuous` and `scale_y_continuous` (`scale_*_continuous`). Are the default linear continuous scales, there are other continous scales :
* `scale_*_log10()`
* `scale_x_reverse()`
* `scale_x_sqrt()`
Beside allowing for the limits, the scales allow labels and positioning of the scales on the plot, among other features.

One important note. If you plot say a line with `geom_line` and one of the points is outside the scale then the line will not be drawn.

#### `seq()`

Remember the `:` operator? Well `3:17` is equivalent to `seq(3, 17)`. The `seq()` function allows more control, including step size or data sized based on a repeat pattern. In this case `seq(0, 15, 1)` is the same as `seq(0, 15)`, which is the same as `0:15`

### More with color and shape

No need to highlight before ctrl-enter. It executes code line by line. It understands continued lines of code and will excute them as one and move the cursor one position down so repeatedly pressing ctrl-enter is like stepping through as it executes.

For reference: 
* The [color chart](http://www.stat.columbia.edu/~tzheng/files/Rcolor.pdf) and 
* the [shape chart](http://sape.inf.usi.ch/quick-reference/ggplot2/shape) he showed

### Graphing lines with ggplot

In this case he uses `geom_line`. If he created the `ggplot` object earlier with the data and aesthetics he could have subsituted the points for lines with the by switching the function. This is a big reason the first form of `ggplot(data, mapping)` is generally prefered if data and aesthetics are common between all layers

### More with lines

If you remember my [post on vector operations]({{ site.baseurl }}{% post_url 2017-07-24-getting-started-with-r3 %}) (and the illustration of the rack-and-pinion gear) you know how a smaller vector gets applied in multiples over the larger vector. It should be no surprise that you can assign the vector y based on math of the applied to another.

## Sampling form populations
### Normal populations
Explains that samples typically near the mean with equal probability of a sample to be on either size of the mean. Explains that standard deviation is how wide the range is around the mean.

`rnorm` has the following signature `rnorm(n, mean = 0, sd = 1)` where `n` is the number of observations.

### Plotting a vertical sample

You can also create the dataframe in a single statement like this

```R
dat <- data.frame(x = 1, y = rnorm(100, 50, 10))
```

No need to use rep since the larger vector "y" will dictate the size and since x = 1 is a vector of single element it is applied accross the entire dataframe.

`mean` is actually the name of a built in function so its a bad identifier name. 

Another way to output this data would be as follows:

```R
dat_mean <- data.frame(x = 1, y = 50)

dat <- data.frame(x = 1, y = rnorm(100, 50, 10))

ggplot(data = dat, mapping = aes(x, y) ) +
    geom_point() + # inherits data and aesthetics
    geom_point(data = dat_mean, size = 7, color = "red") # overrides data, but uses inherited aesthetics
```

Notice how we specify the aesthetics only once, since the dat_mean frame has the same same aesthetics. 

```R
ggplot( mapping = aes(x, y) ) + # aesthetics are common
  geom_point(data = dat) +
  geom_point(data = dat_mean, size = 7, color = "red")
```
We have to specify the parameter name for the aesthetics (mapping), because it is the second param after data in the ggplot function. The x and y parameters in the aesthetics function are so commonly used they are often not named. Also we could have passed 1 as x in the aesthetics, without repeating in our data frame

```R
dat_mean <- data.frame(y = 50)

dat <- data.frame(y = rnorm(100, 50, 10))

ggplot(data = dat, mapping = aes(1, y)) +
    geom_point() + #Inherits the data and aesthetics
    geom_point(data = dat_mean, size = 7, color = "red") #overrides the data and inherits aesthetics
```

### Plotting multiple vertical samples
 
A shorthand for the vector he is creating here would be

```R
rep(c(1, 9, 15), 1, each = 100)
```

This will repeat the vector 1, 9, 15 one time, but each element repeated 100 times. If you know of other short ways to do this please comment below.

Another way to plot the  data without adding the x values into the data frame is to split into separate point layers like this with different x specified in the mapping  (aesthetics).

```R
dat <- data.frame(y = c(rnorm(100, 50, 10),
                        rnorm(100, 30, 10),
                        rnorm(100, 78, 10)))

dat_means <- data.frame(x = c(1, 9, 15), y = c(50, 30, 78))

ggplot() +
  geom_point(mapping = aes(1, y), data = dat[1:100, , drop = F]) +
  geom_point(mapping = aes(9, y), data = dat[101:200, , drop = F]) +
  geom_point(mapping = aes(15, y), data = dat[201:300, , drop = F]) +
  geom_point(mapping = aes(x, y), data = dat_means,  size = 7, color = "red")
```

I know there must be ways to facet the data in other ways using ggplot, but I'm still learning and may update later

### Samples along a line
You already saw how to do the simple repeats above or how to avoid using them at all using the aesthetics. 

### sapply
`sapply()` is the more friendly version of `lapply()` a  utility for applying an function over a list or a vector. `sapply()` returns a vector or matrix where possible.

In many case there is no need to specify `function(x) call_fun(x)`. You can simply supply `call_fun`

for instance. He could have done this:

```R
x <- c(2, 4, 9, 15)
sapply(x, sqrt)
```
As long as the elements in the supplied data can be passed to the function as `FUN(x[[index]], ...)` most of the time that would be fine, but as the documentation points out `is.numeric` (and potentially other `is.*` functions), may give you different results if you expected the indexing to be simple `[]` indexing instead of `[[]]` (indexing with tag removal)

A note on infix operators for instance `x ^ 2`, this can be rewritten as `` `^`(x, 2)`` so in that case 

```R
x <- c(2, 4, 9, 15)
sapply(x, `^`, 2) 
```

Note that I just added the second parameter to `^` as an additional parameter. It will automatically be placed by index in the function after the indexed value in the input. The nice thing about this is that the operator written as a parameter in the form ``(x, `^`, 2)``, looks surprisingly like the normal infix notation `x ^ 2`. 

Now, in the case where he calculates rnorm we are a bit out of luck as far as I know. I think in that case we have to use the `function(x) rnorm(1, x, 10)` format. If you know of an elegant alternative then please comment below

### Cloud of points
The point of this lesson is near the end, where he tries to show if we base a population off a line and that we will use regression to determine the line. He is essentially engineering a "random" set based off a line. I assume he will show that linear regression will fit a line to the data and match the "source" line.

He creates a normal population and plots it along a line, then takes one sample offset in y to add to the plot. 

Once you grasp the step by steps of the this lesson here is a shorter way to create the pointcloud along the line. I've elminated the temporary vectors

```R
data_scatter <- data.frame(x = rnorm(100, 10, 5))
data_scatter$y <- sapply(data_scatter$x, function(x) rnorm(1, 3 * x + 1, 10))

trend_line <- data.frame(x = c(-5, 25))
trend_line$y <- 3 * trend_line$x + 1

ggplot(data = data_scatter, mapping = aes(x, y) )+
  geom_point() + # inherits data and aesthetics
  geom_line(data = trend_line) + # inherits aesthetics
  scale_x_continuous(limits = c(-10, 30)) +
  scale_y_continuous(limits = c(-20, 80)) 

```

Note that the standard deviation in x and y are different. also note that if you place the endpoints of the trend line outside the scale limits the line is not drawn.

### Father and son heights
 I prefer `str()` over `head()`. It still shows a few values for each variable (column), but in a transposed output.

### Equation of a line
 Simple refresher on getting a line using the slope-point method

### Residual visualizations
One way to create the two data frames is just to use `assign` to make clone of the dataframe and add the extra column. 

```R
assign("dat", father.son)
colnames(dat) <- c("x", "y")
dat$group <- 1:nrow(father.son)

assign("means", dat)
means$y <- means$x + 3 
```

In this lesson he explains that residuals are the deltas in Y between measured and trend line. Notes on squaring residuals: It emphasizes large differences and ensures a positive metric

### Sum of squared residuals
Pretty straightforward explanation of the sum of squares.

### Least Squars Line
`lm` or Fit Linear Model function takes a formula as its first parameter. We saw formulas in  [Kaggle Titanic tutorial follow-along]({{ site.baseurl }}{% post_url 2017-07-31-datacamp-kaggle-tutorial %}) where we used them as parameters to the tree and random forest generator functions.

In this case we have `y ~ model` where we are attempting to predict y based off a model of other driving fields separated by `+`. We saw that in the Kaggle Titanic Turional. Note that there are advanced operators such as `*` for factor crossing, but that is an advanced topic. In this case we have a simple linear form so we are tracking the dependenct of y on x so the model is just `x` so the formula is simply `y ~ x`. 

If you'd like to fully qualify the formula parameter simply add `formula=` in front of the formula definition

Once you get the slope and intercept there is no need to recreate the means dataframe. You can simply update the y's without dissecting and reassembling the dataframe

```R
means$y <- slope * means$x + intercept
```

Doing this preserves the group data as well.

### Predicition
Not much here except that the regressed line does a best (least squares best) approximation of population regressed to a linear function.

### Readin in Excel Files

Saving as csv. Pretty much what you should know from the [Kaggle Titanic tutorial follow-along]({{ site.baseurl }}{% post_url 2017-07-31-datacamp-kaggle-tutorial %}). R-Studio has the ability to bring in Excel files directly if you select that from the "Import Dataset" menu (using the `readxl` package). Also, note that csv is also included under the "Import Dataset". In that case the `read_csv` package will be used to read the csv, that package has a slightly different syntax than the built in `read.csv`

### Conclusion
This course is just OK. I already knew how linear regression worked, but maybe you found value in this course