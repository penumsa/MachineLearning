---
title: "Prediction Assignment Writeup"
author: "Sastry Penumarthy"
date: "Sunday, January 25, 2015"
output: html_document
---
DOCUMENT AND DATA SOURCE -
Ugulino, W.; Cardador, D.; Vega, K.; Velloso, E.; Milidiu, R.; Fuks, H. Wearable Computing: Accelerometers' Data Classification of Body Postures and Movements. Proceedings of 21st Brazilian Symposium on Artificial Intelligence. Advances in Artificial Intelligence - SBIA 2012. In: Lecture Notes in Computer Science. , pp. 52-61. Curitiba, PR: Springer Berlin / Heidelberg, 2012. ISBN 978-3-642-34458-9. DOI: 10.1007/978-3-642-34459-6_6.
Cited by 2 (Google Scholar)

Read more: http://groupware.les.inf.puc-rio.br/har#ixzz3PsMvtesA
---
Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset). 


Data 

The training data for this project are available here: 

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here: 

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

The data for this project come from this source: http://groupware.les.inf.puc-rio.br/har.
---
****Load the training and test data sets
```{r, echo=TRUE}
library(corrplot)
library(caret)
setwd("C:\\Penumarthy\\SastryFolder\\Coursera\\Data Science\\Machine Learning")

pml_training <- read.csv("pml-training.csv", header=TRUE, na.strings = c("NA", ""))
dim(pml_training)
pml_testing <- read.csv("pml-testing.csv", header=TRUE, na.strings = c("NA", ""))
dim(pml_testing)
```

Paring down the data by removing Columns with missing values - this is a pre-processing step to prepare the data for further processing. This is done by eliminating columns that have null values and also columns that are "time_stamp, user_name and new_window"
```{r, echo=TRUE}
pml_train_less_cols <- pml_training[, (colSums(is.na(pml_training)) == 0)]
pml_testing <- pml_testing[, (colSums(is.na(pml_training)) == 0)]

removed_cols <- grepl("X|user_name|timestamp|new_window", colnames(pml_train_less_cols))
pml_train_less_cols <- pml_train_less_cols[, !removed_cols]
pml_testing_final <- pml_testing[, !removed_cols]
colnames(pml_train_less_cols)
```
---
The following process steps are done on the data -
a. Split the training set into 2 - 70% into Training, and 30% of the data into Testing
b. Fitting a model
```{r, echo=TRUE}
inTrain <- createDataPartition(y = pml_train_less_cols$classe, p = 0.7, list = FALSE)
train_data <- pml_train_less_cols[inTrain, ]
dim(train_data)
test_data <- pml_train_less_cols[-inTrain, ]
colnames(test_data)
```
The data set is now 54 columns wide. Next, the algorithm looks at the correlations of the variables and taking a weighted context of the predictors of "classe" to preserve the most information

```{r, echo=TRUE}
Correl <- cor(train_data[, -54])
corrplot(Correl, order = "FPC", method = "color", type = "lower", tl.cex = 0.8, 
    tl.col = rgb(0, 0, 0))
```

Fitting a "PCA" model and identifying the key predictors
```{r, echo=TRUE}
preProc <- preProcess(train_data[, -54], method = "pca", thresh = 0.99)
trainPC <- predict(preProc, train_data[, -54])
valid_testPC <- predict(preProc, test_data[, -54])

modelFit <- train(train_data$classe ~ ., method = "rf", data = trainPC, trControl = trainControl(method = "cv", number = 4), importance = TRUE)


varImpPlot(modelFit$finalModel, sort = TRUE, type = 1, pch = 19, col = 1, cex = 1, 
    main = "Importance of the Individual Principal Components")
```

Cross-Validation
```{r, echo=TRUE}
predictValid <- predict(modelFit, valid_testPC)
conMatrix <- confusionMatrix(test_data$classe, predictValid)
conMatrix$table
```

Determining the model Accuracy and the out-of-sample error values
```{r, echo=TRUE}
accuracy <- postResample(test_data$classe, predictValid)
model_accuracy <- accuracy[[1]]
model_accuracy
out_of_sample_error <- 1 - model_accuracy
out_of_sample_error
```

Predicting using the test data set
```{r, echo=TRUE}
testPred <- predict(preProc, pml_testing_final[, -54])
Predictions <- predict(modelFit, testPred)
predictions
```
