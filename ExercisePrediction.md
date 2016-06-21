# Exercise prediction
Adriano Mendo  
June 21, 2016  

# Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, the goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways.

# Reading and cleaning training data

Reading the training data:

```r
setwd("C:/Adri/Courses/The Data Science Specialization/Exercises/8 - Machine Learning")
df.raw = read.csv("pml-training.csv")
```

Cleaning the data by removing those variables with some NAs or empty values, and those that do not make sense for the prediction (factors and date/time):

```r
na_or_blank_count  = sapply(df.raw, function(x) sum(is.na(x) | x==""))
df = df.raw[,-which(na_or_blank_count > 0)]
df = df[,-c(1,2,3,4,5,6,7)]
```

# Splitting between training and testing

Data will be now splitted between training data (70 %) and testing data (30 %):

```r
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.2.5
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```
## Warning: package 'ggplot2' was built under R version 3.2.4
```

```r
set.seed(1)
inTrain = createDataPartition(y=df$classe,p=0.7,list=FALSE)
training = df[inTrain,]
testing = df[-inTrain,]
```

# Model selection and building

Random forest is one of the most suitable algorithms to make this kind of predictions:


```r
modFit = train(classe~., data=training, method="rf", ntree=500)
```

```
## Loading required package: randomForest
```

```
## Warning: package 'randomForest' was built under R version 3.2.5
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```r
modFit
```

```
## Random Forest 
## 
## 13737 samples
##    52 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Bootstrapped (25 reps) 
## Summary of sample sizes: 13737, 13737, 13737, 13737, 13737, 13737, ... 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa    
##    2    0.9875551  0.9842495
##   27    0.9879930  0.9848052
##   52    0.9775792  0.9716257
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 27.
```

# Prediction error

Using the testing data, let us calculate the prediction error:

```r
pred = predict(modFit,testing)
confusionMatrix(pred,testing$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1671    8    0    0    0
##          B    0 1128    5    0    1
##          C    2    2 1018    5    1
##          D    0    1    3  956    2
##          E    1    0    0    3 1078
## 
## Overall Statistics
##                                          
##                Accuracy : 0.9942         
##                  95% CI : (0.9919, 0.996)
##     No Information Rate : 0.2845         
##     P-Value [Acc > NIR] : < 2.2e-16      
##                                          
##                   Kappa : 0.9927         
##  Mcnemar's Test P-Value : NA             
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9982   0.9903   0.9922   0.9917   0.9963
## Specificity            0.9981   0.9987   0.9979   0.9988   0.9992
## Pos Pred Value         0.9952   0.9947   0.9903   0.9938   0.9963
## Neg Pred Value         0.9993   0.9977   0.9984   0.9984   0.9992
## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
## Detection Rate         0.2839   0.1917   0.1730   0.1624   0.1832
## Detection Prevalence   0.2853   0.1927   0.1747   0.1635   0.1839
## Balanced Accuracy      0.9982   0.9945   0.9951   0.9952   0.9977
```

# Testing the model

Loading the 20 cases where the fitted model will be applied:

```r
val = read.csv("pml-testing.csv")
```

Prediction for these cases are:

```r
pred = predict(modFit,val)
pred
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
