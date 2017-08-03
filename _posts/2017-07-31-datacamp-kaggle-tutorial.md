---
title: "Notes on Datacamp's kaggle tutorial"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
---
Kaggle has a tutorial competition on the survival of the passengers of the Titanic. Datacamp has a handy tutorial on using R to tackle the problem. Here are my notes working through the tutorial
<!--more-->

I am posting this tutorial as I learn R. I will respond to feedback for errata in the comments.
{: .notice}

I will not address any issues that we have covered in the getting started section. Follow along at [Datacamp's open course on machine learning and the sinking of the Titanic](https://www.datacamp.com/community/open-courses/kaggle-tutorial-on-machine-learing-the-sinking-of-the-titanic)


## How it works
Basic information on navigating the tutorial. Basically instructions are on the left, the script file you submit is at the top right and the console where you can experiment is at the bottom right. Using `?` in front of a topic brings up help in a new tab at the top right as a separate tab page to the script file.

## Set sail
### read.csv()
`read.csv()` is essentially just a call to `read.table()` with some defaults provided to make it better suited for comma separated value files. `read.table` and its variants `read.csv()`, `read.csv2()`, `read.delim()` and `read.delim2()` all are used for reading files and producing data frames. In case you are wondering, the variants ending in 2 are for files that have ',' as a decimal separator. As a developer this horrifies me, it would have been better simply to override the default `dec` parameter in `read.csv` with `dec = ","`.

### url()
The input to `read.table()` and by extension `read.csv()` is a file that can be provided simply by specifying a file name or via a "connection", which I understand to be analogous or perhaps equivalent to a text stream. In this sample the connection used is by using the `url()` function. The connections topic has more on this topic but the `read.table()` function takes care of opening, reading and closing the connector, we simply hav eto construct it and pass it as a parameter

`read.csv()` seems to operate fine when passing a url directly as a file location like this `read.csv(url_to_a_csv)`, I assume there is some parser magic happening that recognizes that the path provided is a URL. But to be explicit we can always call it like this: `read.csv(url(url_to_a_csv))` 

### print()
I have not shown you print, mostly because just calling a value will print its values automatically in the console. For your information `print()` is a more explict way to oputput data. As far as this tutorial at Datacamp, it doesn't seem to matter if you just list the variables in the script or actually call print on them to pass the exercise

## Understanding your data
I've already commented on the `str()` or structure function. In this case where we have a larger dataset you can now see how much more useful it is that with my simple three row dataset. To recap : Database table analog: observations are like records, variables are fields. Below the observation you can see the dataframe transposed so we can see each variable's name, type and a sample of some of its data. You'll also see that strings were converted to factors automatically

## Rose vs Jack, or Female vs Male
### table()
The `table()` function is used for cross tabulation. In this case a single variable was passed and the count of each instance is reported. If we had passed more variables we would have counts based on combinations.

### prop.table()
The `prop.table()` function is used to get the proportional relations. Viewing the help reveals that this is essentially a simplified call to `sweep(x, margin, margin.table(x, margin), "/") f`. The `margin.table()` call gives us the sum of x over the margin (1 for rows). The `sweep()` function produces a summary over a vector. So we can see that the since we didn't provide a margin index the sum of each instance by row was divided by the total of all instances. And so `prop.table()` ends up being just a simple way to convert a cross tabulated table into basic proportions

## Does age play a role?
### Adding new variables
Here we see something I neglected to cover in my dataframe post. Simply assigning to a field that does not exist will create the field before assigning it.  If you've followed along with my getting started tutorial then the 
```R
train$Child[train$Age < 18] <- 1
```
line should make perfect sense. Each of the `$` operators return a vector for the field, applying the `<` operator with 18 applies it over the entire vector and results in a vector of booleans that are used to index the vector of the `Child` column. Not sure why they did not use `logicial` in this case, but I assume it make the dataframe more flexible if `Child` needs to be more than just a binary value.

### table() and prop.table() on two columns
In this case we can see the crosstabulation of `Child` and `Survived`. Now you can see `prop.table()` return the proportions independently by row

## Making your first prediction
You should by now know how to do everthing required by this page:
    * Adding a variable (column) through initialization 
    * Setting one field's values based on a logical condition of another 
Both these can be done, but just refering back to how we set the `Child` column

## Intro to decision trees
A decision tree is a simple recursive sub-divider over a dataset. It greedily evaluates which criteria will class the data into the target categories and repeats the process recursively. 

### library and install.packages
In the distribution I installed `rpart` (recursive partitioning) was included and was listed as a system library. Not all libraries may be available local on your machine and so you can't just load them with `library(package_name)` before you install them by using `install.packages('package_name')`. If you don't have a package installed and use `library(package_name)` then you will get an error message that there is no package with the name you provided.

## Creating your first decision tree
### rpart.plot library with plot.rpart() and text.rpart() functions
The `plot()` and `text()` function called here are actually the overloads that takes an `rpart` object and plots it, and adds text to a plot. Just for interest sake the plot generated is called a "dendograph" because it is a graph of a tree.

# rattle and RColorBrewer libraries
The `rattle` package contains the `fancyRpartPlot()` creats the nice looking colorized dendograph. Not sure why the `RColorBrewer` package was listed here, its certainly not needed for the graph and including it does not seem to change the appearance of the graph. I'm not sure why my graph is colorized without the package, but I assume at some point it was needed to create the nice blue and green hues.

## Interpreting your decision tree
Nothing much to comment here, you can easily see the nodes that affect the outcome the most

## Predict and submit to Kaggle
### predict.rpart()
The `predict()` function here is actually the one from rpart `predict.rpart()`. Since we already set up our `rpart()` object to make decisions on the `Survived` field we don't have to specify it. We simply have to class our data over the tree to make predicitons. In this case we class our `test` data over the tree we created using `train` data and we get a vector of `1` and `0` similar to how `train` was classed during hte making of the decision tree.

### Creating the dataframe
The `data.frame` construction function should be pretty easy to understand. We simply pass the PassengerID vector of the `test` data along with the `my_prediction`. Most csv's have column names as the first row, by Kaggle specified in their instructions to exclude it. 

### write.csv()
Hhere `write.csv()` is the counterpart to `read.csv()`. As with the read, this is just a special version of the more general `write.table` and as before we can write to a file or to a connector (stream like interface object).

## Overfitting, the iceberg of decision trees
The `control` parameter passed into `rpart()` is a `rpart.control()` object that controls aspects of the fitting algorithm in this case `cp` means the "complexity parameter". What the heck does that mean?  From the help file:

> Any split that does not decrease the overall lack of fit by a factor of cp is not attempted. For instance, with anova splitting, this means that the overall R-squared must increase by cp at each step. The main role of this parameter is to save computing time by pruning off splits that are obviously not worthwhile. Essentially,the user informs the program that any split which does not improve the fit by cp will likely be pruned off by cross-validation, and that hence the program need not pursue it.

Times like these I wish I studied more statistics. At any rate, as far I can figure out the R-squared value approaches 1 as the tree grows, and that each decision node constributes to this factor - an indicator of difference between the model and training set. 

We do not want to overfit our data as you will see when you finish the course so the tree fragmentation stops early and prevents fine segmentation that reduces the "fuzziness" of the tree. By setting the `cp` parameter to 0 he essentially making the tree make a decision on every single factor or value and creating a very fragmented tree. The `minsplit` factor means that the minimum  number of values at a node to trigger a split is 2, meaning every single value will have its own bucket!

Even though we don't change `cp` the `minsplit` acted as a safety net allowing a minimum of 50 elements per node to be split. 

## Re-engineering our Titanic data set
This page touches on re-engineering which is essentially injecting human evaluation and generation as extra data. This can be classing, grouping, combining, filling missing data and cleanup of bad data. In this test an extra dummy column was addded that is the person, their parents and their siblings counted as a new variable family size. However we see that with our greedy decision tree it had no impact.

## Passenger Title and survival rate
Running `levels(train_new$Title)` shows there are 11 titles and its no surprise that "Dr, Mr, Rev, Sir" are a primary classer, similar to gender. However some of the other titles biased towards survival may also be male. I know from looking at the original Kaggle set that there were many more titles including "Don", "Master" (and it's Dutch variant "Jonkheer"), both abbreviation versions of mademoiselle (mme and mlle), as well as "The Countess". What is not show here is that these were parsed from the `Name` field, grouped and cleaned up and then added as an extra column.

## What is a random forest?
The description here about random forests are clear enough: the problem with overfitting is solved by random decision trees that are allowed to be highly fragmented and then allowing those trees to partake in an election. There are still some shortfalls here. Our greedy tree had a good starting point, but if we have a large number of trees with poor starting points in our data our results could be poor also. There is a balance here in the parameters, their variation and the factors that are given precedence in the trees of the forest.

### factor
I've previously discussed how when we created our dataframe that the string data was automatically converted to factors in this case the `Embarked` field was already a factor and changing the two `NA` values does not change it, so there was no real need to call `factor()` here. Now if the default during import was set to keep `characters` instead of converting them to factors during construction, then this would have been a way to factor them after the fact. 

### "anova" 
ANOVA: Analysis of variance. Seems to be a way to partition a dataset by evaluating the distribution and creating classes. So it seems that here age is predicted as a continuous factor dependent on the other variables. This is quite different from the previous division method that was labeled "class"

## A Random Forest analysis in R
The `randomforest` package provides the `randomForest()` that is very similar to `rpart` 

Let us compare a tree and a forest created to class a predicted variable 
```R
forest <- randomForest(forumula=as.factor(predicted_variable) ~ variable1 + variable2 + variable3, data = train)
tree <- rpart(predicted_variable ~  variable1 + variable2 + variable3, method = "class")
```
In this case we also have to add set the number of trees with `ntree=1000` (which defaults to 500) and `importance=TRUE`. Not 100% clear on the importance variable, but it aparently records the importance of each variable in the reduction of the error. I assume it is used when we call `predict`, but I don't yet know how it affects that function

The `predict` function applies a dataset to the forest which votes on the probability of the final result. 

## Important Variables
Here at least we can see what varaibles are most important. The plot here `varImpPlot(my_forest)` shows importance by two measures "Mean Decrease Accuracy" : which I understand to mean that if we omit this variable what is the decrease in the mean prediction, and "Mean Decrease Gini" which is a measure of the purity of each node. As we split the node should become more pure, this decrease in Gini-impurity can also be used as a measure of importance

Thats the end of the Tutorial and now you can go off to Kaggle and try the Titanic competition yourself