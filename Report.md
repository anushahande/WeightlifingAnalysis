Weightlifting Analysis
=======================
Practical Machine Learning - Prediction Assignment Writeup

Summary
--------
This document is the final report of the Peer Assessment project from the Practical Machine Learning course, which is a part of the Data Science Specialization. It was written and coded in RStudio, using its knitr functions and published in the html format. The purpose of this analysis is to predict the manner in which the six participants performed the exercises described below and to answer the questions of the associated course quiz. The machine learning algorithm, which uses the classe variable in the training set, is applied to the 20 test cases available in the test data. The predictions are submitted to the Course Project Prediction Quiz for grading.

Introduction
-------------
Devices such as Jawbone Up, Nike FuelBand, and Fitbit can enable collecting a large amount of data about someone’s physical activity. These devices are used by the enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. However, even though these enthusiasts regularly quantify how much of a particular activity they do, they rarely quantify how well they do it. In this project, the goal is to use data from accelerometers on the belt, forearm, arm, and dumbell of six participants. They were asked to perform barbell lifts correctly and incorrectly in five different ways.

More information is available from the following website: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

Source of Data
--------------
The data for this project can be found on the following website:

http://groupware.les.inf.puc-rio.br/har.

The training data for this project:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data for this project:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

The full reference is as follows:

Velloso, E.; Bulling, A.; Gellersen, H.; Ugulino, W.; Fuks, H. “Qualitative Activity Recognition of Weight Lifting Exercises. Proceedings of 4th International Conference in Cooperation with SIGCHI (Augmented Human ’13)”. Stuttgart, Germany: ACM SIGCHI, 2013.

Data Loading and Cleaning
-------------------------

Load the required R packages and set a seed.
``` r
library(lattice)
library(ggplot2)
library(caret)
library(rpart)
library(rpart.plot)
library(corrplot)
library(rattle)
library(randomForest)
library(RColorBrewer)
``` 
``` r
set.seed(1813)
``` 
Load the training and test datasets.
``` r
url_train <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
url_quiz  <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
``` 
``` r
data_train <- read.csv(url(url_train), strip.white = TRUE, na.strings = c("NA",""))
data_quiz  <- read.csv(url(url_quiz),  strip.white = TRUE, na.strings = c("NA",""))
``` 
``` r
dim(data_train)
``` 
``` r
## [1] 19622   160
``` 
``` r
dim(data_quiz)
``` 
``` r
## [1]  20 160
```
Create two partitions (75 % and 25 %) within the original training dataset.
``` r
in_train  <- createDataPartition(data_train$classe, p=0.75, list=FALSE)
train_set <- data_train[ in_train, ]
test_set  <- data_train[-in_train, ]
``` 
``` r
dim(train_set)
``` 
``` r
## [1] 14718   160
``` 
``` r
dim(test_set)
``` 
``` r
## [1] 4904  160
``` 
The two datasets (train_set and test_set) have a large number of NA values as well as near-zero-variance (NZV) variables. Both will be removed together with their ID variables.
``` r
nzv_var <- nearZeroVar(train_set)
``` 
``` r
train_set <- train_set[ , -nzv_var]
test_set  <- test_set [ , -nzv_var]
``` 
``` r
dim(train_set)
``` 
``` r
## [1] 14718   121
``` 
``` r
dim(test_set)
``` 
``` r
## [1] 4904  121
``` 
Remove variables that are mostly NA.
``` r
na_var <- sapply(train_set, function(x) mean(is.na(x))) > 0.95
train_set <- train_set[ , na_var == FALSE]
test_set  <- test_set [ , na_var == FALSE]
``` 
``` r
dim(train_set)
``` 
``` r
## [1] 14718    59
``` 
``` r
dim(test_set)
``` 
``` r
## [1] 4904   59
``` 
Since columns 1 to 5 are identification variables only, they will be removed.
``` r
train_set <- train_set[ , -(1:5)]
test_set  <- test_set [ , -(1:5)]
``` 
``` r
dim(train_set)
``` 
``` r
## [1] 14718    54
``` 
``` r
dim(test_set)
``` 
``` r
## [1] 4904   54
```

Correlation Analysis
--------------------
 
``` r
corr_matrix <- cor(train_set[ , -54])
corrplot(corr_matrix, order = "FPC", method = "circle", type = "lower",
         tl.cex = 0.6, tl.col = rgb(0, 0, 0))
``` 
![](/1.png)
If two variables are highly correlated their colors are either dark blue (for a positive correlation) or dark red (for a negative corraltions)

Prediction Models
-----------------
### Decision Tree Model

``` r
set.seed(1813)
fit_decision_tree <- rpart(classe ~ ., data = train_set, method="class")
fancyRpartPlot(fit_decision_tree)
``` 
![](/2.png)

Predictions of the decision tree model on test_set.
``` r
predict_decision_tree <- predict(fit_decision_tree, newdata = test_set, type="class")
conf_matrix_decision_tree <- confusionMatrix(predict_decision_tree, test_set$classe)
conf_matrix_decision_tree
``` 
``` r
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1248  173   48   42   39
##          B   49  589   29   92   40
##          C    3   54  667  107   49
##          D   76   89   51  494   98
##          E   19   44   60   69  675
## 
## Overall Statistics
##                                           
##                Accuracy : 0.749           
##                  95% CI : (0.7366, 0.7611)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.6814          
##  Mcnemar's Test P-Value : < 2.2e-16       
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.8946   0.6207   0.7801   0.6144   0.7492
## Specificity            0.9139   0.9469   0.9474   0.9234   0.9520
## Pos Pred Value         0.8052   0.7372   0.7580   0.6114   0.7785
## Neg Pred Value         0.9562   0.9123   0.9533   0.9243   0.9440
## Prevalence             0.2845   0.1935   0.1743   0.1639   0.1837
## Detection Rate         0.2545   0.1201   0.1360   0.1007   0.1376
## Detection Prevalence   0.3161   0.1629   0.1794   0.1648   0.1768
## Balanced Accuracy      0.9043   0.7838   0.8638   0.7689   0.8506
```
The predictive accuracy of the decision tree model is relatively low at 74.9 %.

### Random Forest Model
``` r
set.seed(1813)
ctrl_RF <- trainControl(method = "repeatedcv", number = 5, repeats = 2)
fit_RF  <- train(classe ~ ., data = train_set, method = "rf",
                  trControl = ctrl_RF, verbose = FALSE)
fit_RF$finalModel
``` 
``` r
## 
## Call:
##  randomForest(x = x, y = y, mtry = param$mtry, verbose = FALSE) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 27
## 
##         OOB estimate of  error rate: 0.22%
## Confusion matrix:
##      A    B    C    D    E  class.error
## A 4184    1    0    0    0 0.0002389486
## B    5 2839    3    1    0 0.0031601124
## C    0    4 2563    0    0 0.0015582392
## D    0    0   10 2401    1 0.0045605307
## E    0    1    0    7 2698 0.0029563932
``` 
Predictions of the Random Forest model on test_set.
``` r
predict_RF <- predict(fit_RF, newdata = test_set)
conf_matrix_RF <- confusionMatrix(predict_RF, test_set$classe)
conf_matrix_RF
``` 
``` r
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1394    1    0    0    0
##          B    0  946    2    0    0
##          C    0    2  853    3    0
##          D    0    0    0  801    1
##          E    1    0    0    0  900
## 
## Overall Statistics
##                                          
##                Accuracy : 0.998          
##                  95% CI : (0.9963, 0.999)
##     No Information Rate : 0.2845         
##     P-Value [Acc > NIR] : < 2.2e-16      
##                                          
##                   Kappa : 0.9974         
##  Mcnemar's Test P-Value : NA             
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9993   0.9968   0.9977   0.9963   0.9989
## Specificity            0.9997   0.9995   0.9988   0.9998   0.9998
## Pos Pred Value         0.9993   0.9979   0.9942   0.9988   0.9989
## Neg Pred Value         0.9997   0.9992   0.9995   0.9993   0.9998
## Prevalence             0.2845   0.1935   0.1743   0.1639   0.1837
## Detection Rate         0.2843   0.1929   0.1739   0.1633   0.1835
## Detection Prevalence   0.2845   0.1933   0.1750   0.1635   0.1837
## Balanced Accuracy      0.9995   0.9982   0.9982   0.9980   0.9993
``` 
The predictive accuracy of the Random Forest model is excellent at 99.8 %.

Applying the Best Predictive Model to the Test Data
---------------------------------------------------

The predictive accuracy of the three models evaluated is as follows:

Decision Tree Model: 74.90 %

Random Forest Model: 99.80 %

The Random Forest model is selected and applied to make predictions on the 20 data points from the original testing dataset (data_quiz).
``` r
predict_quiz <- predict(fit_RF, newdata = data_quiz)
predict_quiz
``` 
``` r
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
``` 








