# Predicting Behaviour of Weight Lifting Exercises

========================================================

## Abstract

Research on activity recognition has traditionally focused on discriminating between different activities, i.e. to predict which activity was performed at a specific point in time.The quality of executing an activity, the how well has
only received little attention so far, even though it potentially provides useful information for a large variety of applications. In this work we are going to study about weight lifting patterns of individuals & based on this pattern we are going to predict execution mistakes. 

## Approach 

The required training & testing data sets are available in respective links [TRAINING DATA SET](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv) & [TEST DATA SET](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv). Next we need to clean our data set & then splitting taring data set into three sets:-

- Training 70%
- Testing 20%
- Validation 10%

To predict the outcome of particular excise pattern we are going to Random Forest as our algorithm. It is one of the faster & more accurate prediction algorithm.

Finally we need to compare predicted result with actual & already available % of accuracy

### Define Error Rate:-

Based on the paper "Qualitative Activity Recognition of Weight Lifting Exercises"  overall recognition performance was of 98.03%. Our goal is to predict the outcome as per our model & compare performance of the same. 

### Getting & Cleaning of Data:- 

Training data set should be loaded & its dimensions


```r
x <- read.csv("pml-training.csv")  ## Reading Training data set
dim(x)
```

```
## [1] 19622   160
```


we have 160 variable then we need to check & remove variable which are not required for our analysis. One by removing Near Zero Values & columns which are having maximum Na's.




```
## Loading required package: lattice
## Loading required package: ggplot2
```



```r
y <- nearZeroVar(x)
z <- x[, -y]  ## Removing Near zero values
v <- z[, colSums(is.na(z)) == 0]  ## Removing columns which has NA's
w <- v[, -(1:6)]
dim(w)  ## final cleaned data set 
```

```
## [1] 19622    53
```


### Data Slicing:- 

As indicated in our approach the final cleaned data set shall be sliced further to carry out further analysis as below. 


```r

wt <- createDataPartition(y = w$classe, p = 0.7, list = FALSE)
train <- w[wt, ]  ## spliting of tidy data set into training 
temp <- w[-wt, ]
tempt <- createDataPartition(y = temp$classe, p = 0.67, list = FALSE)  ## spliting of tidy data set into testing & validation 
test <- temp[tempt, ]  ## testing set
valid <- temp[-tempt, ]  ## validation set
```


### Modelling:- 

Using random forest technique data set shall be trained as below. 

```r
suppressWarnings(library(randomForest))
```

```
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```r
rf <- randomForest(classe ~ ., train, ntree = 60, norm.votes = FALSE)
```


Using cross validation we can predict the result of train data set with test set 

```r
v1 <- confusionMatrix(predict(rf, newdata = test), test$classe)
```

```
## Warning: package 'e1071' was built under R version 3.1.1
```

```r
v1$table
```

```
##           Reference
## Prediction    A    B    C    D    E
##          A 1121    1    0    0    0
##          B    1  762    3    0    0
##          C    0    1  684    5    2
##          D    0    0    1  639    2
##          E    0    0    0    2  721
```

```r
v1$overall
```

```
##       Accuracy          Kappa  AccuracyLower  AccuracyUpper   AccuracyNull 
##         0.9954         0.9942         0.9928         0.9973         0.2844 
## AccuracyPValue  McnemarPValue 
##         0.0000            NaN
```


Validation with new data set


```r
v2 <- confusionMatrix(predict(rf, newdata = valid), valid$classe)
v2$table
```

```
##           Reference
## Prediction   A   B   C   D   E
##          A 552   5   0   0   0
##          B   0 369   5   0   0
##          C   0   1 333   7   2
##          D   0   0   0 310   0
##          E   0   0   0   1 355
```

```r
v2$overall
```

```
##       Accuracy          Kappa  AccuracyLower  AccuracyUpper   AccuracyNull 
##         0.9892         0.9863         0.9835         0.9933         0.2845 
## AccuracyPValue  McnemarPValue 
##         0.0000            NaN
```


Our out of sample accurancy will be 98.92% estimated against cross validation as against expected accuracy of 99.54%.


### Final Model predicion for testing data set


```r
testing <- read.csv("pml-testing.csv")
prediction <- predict(rf, newdata = testing)
prediction
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```



#### Appendix:- 

Plot of error rate from our model


```r

plot(rf)
legend("topright", lty = 1, col = 1:6, colnames(rf$err.rate), cex = 0.6, fill = 1:6)
```

![plot of chunk . plot1](figure/__plot1.png) 


