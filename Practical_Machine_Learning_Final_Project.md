Practical Machine Learning Final Project
================
Chidinma Egbukichi
March 6, 2016

The Assignment
--------------

> From the instructor:
> --------------------

> One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways.
>
> The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. You may use any of the other variables to predict with.

Cleaning the Data
-----------------

    training <- read.csv("pml-training.csv")
    testing <- read.csv("pml-testing.csv")
    View(training)

Looking at the training data, the first 7 variables will not be useful for prediction so they are dropped from the training set.

    subset_training <- subset(training, select = -c(X:num_window))

Further, there are many variables that are mainly populated with NA orblank values. Those will be removed from the training set as well.

    subset_training[subset_training==""] <- NA
    training_NAs <- is.na(subset_training)
    bad <- which(colSums(training_NAs) > 19000)
    subset_training <- subset_training[,-bad]

Lastly, check for row completion.

    complete <- complete.cases(subset_training)
    subset_training <- subset_training[complete,]

After cleaning there are 53 predictors and 19,622 observations

``` r
dim(subset_training)
```

    ## [1] 19622    53

Classification Method and Cross Validation
------------------------------------------

I used the boosting method of classification because, as discussed in lecture, it is one of the top-performing algorithms for prediction (along with random forest). Boosting combines many weak predictors, as is likely the case with data, to create stronger composite predictors.

I did not use random forest because of the increased danger of overfitting.

Because this a classification problem (categorical responses), regression methods are unsuitable.

For cross validation, I used the trControl function in caret and selected a k-folds cross validation technique. I chose to use 10 folds because there were a large enough number of observations to account for the bias and variance trade-off.

    library("caret")
    fitControl <- trainControl(method="cv",number=10)
    model_gbm <- train(classe~.,data=subset_training,trControl=fitControl,method="gbm",verbose=FALSE)

Model and Expected Out of Sample Error
--------------------------------------

    ## Stochastic Gradient Boosting 
    ## 
    ## 19622 samples
    ##    52 predictor
    ##     5 classes: 'A', 'B', 'C', 'D', 'E' 
    ## 
    ## No pre-processing
    ## Resampling: Cross-Validated (10 fold) 
    ## Summary of sample sizes: 17659, 17659, 17659, 17660, 17661, 17661, ... 
    ## Resampling results across tuning parameters:
    ## 
    ##   interaction.depth  n.trees  Accuracy   Kappa      Accuracy SD
    ##   1                   50      0.7508929  0.6842063  0.011990965
    ##   1                  100      0.8213237  0.7738058  0.010443345
    ##   1                  150      0.8530232  0.8139915  0.009773470
    ##   2                   50      0.8556736  0.8171090  0.010540405
    ##   2                  100      0.9061257  0.8811959  0.007010221
    ##   2                  150      0.9322197  0.9142339  0.004688926
    ##   3                   50      0.8964950  0.8689858  0.009796465
    ##   3                  100      0.9448579  0.9302276  0.005373246
    ##   3                  150      0.9640199  0.9544778  0.003356879
    ##   Kappa SD   
    ##   0.015302678
    ##   0.013164843
    ##   0.012337389
    ##   0.013323923
    ##   0.008857918
    ##   0.005920655
    ##   0.012370001
    ##   0.006785005
    ##   0.004246694
    ## 
    ## Tuning parameter 'shrinkage' was held constant at a value of 0.1
    ## 
    ## Tuning parameter 'n.minobsinnode' was held constant at a value of 10
    ## Accuracy was used to select the optimal model using  the largest value.
    ## The final values used for the model were n.trees = 150,
    ##  interaction.depth = 3, shrinkage = 0.1 and n.minobsinnode = 10.

The best sample had a 96.4% accuracy on the training set with 150 trees and an interaction depth of 3. As always, I expect the in sample error to be lower than the out of sample error. I predict 5% out of sample error in the test set.

Predictions
-----------

    predict_gbm <- predict(model_gbm, new_testing)
