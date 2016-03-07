Practical Machine Learning Final Project
================
Chidinma Egbukichi
March 6, 2016

Cleaning the Data
-----------------

The object types in the training and test data are different. So I dropped the logical types (NA) columns in the test set from both datasets before I began fitting the data in the training set.

    bad <- is.na(head(testing,n=1))
    new_testing <- testing[,!bad]
    new_training <- training[,!bad]

Classification Method and Cross Validation
------------------------------------------

I used the boosting method of classification because, as discussed in lecture, it is one of the top-performing algorithms for prediction (along with random forest). I did not use random forest because of the danger of overfitting. Because this a classification problem (categorical responses), regression methods are unsuitable.

For cross validation, I used the trControl function in caret and selected a k-folds cross validation technique. I chose to use 10 folds because there were a large enough number of observations to account for the bias and variance trade-off.

    fitControl <- trainControl(method="cv",number=10)
    model_gbm <- train(classe~.,data=new_training,trControl=fitControl,method="gbm",verbose=FALSE)

Model and Predictions
---------------------

The model used 19,622 samples, 59 predictors for a response with 5 classes

    ## Stochastic Gradient Boosting 
    ## 
    ## 19622 samples
    ##    59 predictor
    ##     5 classes: 'A', 'B', 'C', 'D', 'E' 
    ## 
    ## No pre-processing
    ## Resampling: Cross-Validated (10 fold) 
    ## Summary of sample sizes: 17660, 17659, 17660, 17661, 17660, 17659, ... 
    ## Resampling results across tuning parameters:
    ## 
    ##   interaction.depth  n.trees  Accuracy   Kappa      Accuracy SD 
    ##   1                   50      0.9997453  0.9996778  0.0004950863
    ##   1                  100      0.9997453  0.9996778  0.0004950863
    ##   1                  150      0.9996943  0.9996134  0.0004921746
    ##   2                   50      0.9997453  0.9996778  0.0004950863
    ##   2                  100      0.9996943  0.9996134  0.0004921626
    ##   2                  150      0.9996943  0.9996134  0.0004921626
    ##   3                   50      0.9997453  0.9996778  0.0004950863
    ##   3                  100      0.9996943  0.9996134  0.0004921626
    ##   3                  150      0.9996943  0.9996134  0.0004921626
    ##   Kappa SD    
    ##   0.0006261492
    ##   0.0006261492
    ##   0.0006224679
    ##   0.0006261492
    ##   0.0006224518
    ##   0.0006224518
    ##   0.0006261492
    ##   0.0006224518
    ##   0.0006224518
    ## 
    ## Tuning parameter 'shrinkage' was held constant at a value of 0.1
    ## 
    ## Tuning parameter 'n.minobsinnode' was held constant at a value of 10
    ## Accuracy was used to select the optimal model using  the largest value.
    ## The final values used for the model were n.trees = 50, interaction.depth
    ##  = 1, shrinkage = 0.1 and n.minobsinnode = 10.

    predict_gbm <- predict(model_gbm, new_testing)
    summary(predict_gbm)

    ##  A  B  C  D  E 
    ## 20  0  0  0  0

Expected Out of Sample Error
----------------------------

I would, as always, expect the out of sample error to be lower than the in sample error. Increasing number of categories also decreases the expected out of sample error. The model predicted every observation in the test set to be done correctly, class A.
