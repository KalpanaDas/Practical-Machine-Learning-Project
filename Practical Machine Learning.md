---
title: "Practical Machine Learning"
author: "Kalpana Das"
date: "7/31/2020"
output: html_document
---
## Background
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

## Data
The training data for this project are available here: https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv.

The test data are available here: https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv.

The data for this project come from this source: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har.

If you use the document you create for this class for any purpose please cite them as they have been very generous in allowing their data to be used for this kind of assignment.

## Reproducibility
An overall pseudo-random number generator seed is set at 1234 for all code. To reproduce the results below, the same seed should be used. Different packages were downloaded and installed, such as caret and randomForest. These should also be installed in order to reproduce the results below (please see code below for ways and syntax to do so).

## How the model was built?
Our outcome variable is classe, a factor variable with 5 levels. For this data set, participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in 5 different fashions:

1. Exactly according to the specification (Class A)

2. Throwing the elbows to the front (Class B)

3. Lifting the dumbbell only halfway (Class C)

4. Lowering the dumbbell only halfway (Class D)

5. Throwing the hips to the front (Class E)

Class A corresponds to the specified execution of the exercise, the other classes to common mistakes. Prediction evaluations will be based on maximizing the accuracy and minimizing the out-of-sample error. All other available variables after cleaning will be used for prediction. Two models will be tested using decision tree and random forest algorithms. The model with the highest accuracy will be chosen as our final model.

## Cross-validation
This will be done by subsampling our training data set randomly without replacement into 2 subsamples: subTraining data (75% of the original Training data set) and subTesting data (25%). Our models will be fitted on the subTraining data set, and tested on the subTesting data. Once the most accurate model is choosen, it will be tested on the original Testing data set.

## Expected out-of-sample error
The expected out-of-sample error corresponds to (1 - accuracy) in the cross-validation data. Accuracy is the proportion of correct classified observation over the total sample in the subTesting data set. Expected accuracy is the expected accuracy in the out-of-sample data set (i.e. original testing data set). Thus, the expected value of the out-of-sample error corresponds to the expected number of missclassified observations/total observations in the Test data set, which is 1-accuracy found from the cross-validation data set.

Our outcome variable “classe” is an unordered factor variable. Thus, we can choose our error type as 1-accuracy. We have a large sample size, N = 19622, in the Training data set. This allows us to divide our Training sample into subTraining and subTesting and cross-validation. Features with all missing values and irrelevant ones will be discarded. All other features will be kept as relevant variables. Decision tree and random forest algorithms are known for their ability to detecting features important for classification.

## Packages, Libraries and Seed

```{r}
library(caret)
library(randomForest)
library(rpart) 
library(rpart.plot)
library(RColorBrewer)
library(rattle)
library(e1071)
set.seed(1234)
```

## Getting and cleaning data

```{r}
# The training data set can be found on the following URL:
trainUrl <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
# The testing data set can be found on the following URL:
testUrl <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
# Load data to memory.
training <- read.csv(url(trainUrl), na.strings=c("NA","#DIV/0!",""))
testing <- read.csv(url(testUrl), na.strings=c("NA","#DIV/0!",""))
```

## Partioning the training set into two
We partion training data set into two data sets, 60% for myTraining, 40% for myTesting.

```{r}
inTrain <- createDataPartition(y=training$classe, p=0.6, list=FALSE)
myTraining <- training[inTrain, ] 
myTesting <- training[-inTrain, ]
dim(myTraining)
dim(myTesting)
```

## Cleaning the data
The following transformations were used to clean the data.

```{r}
# Transformation 1: Cleaning NearZeroVariance Variables. Run this code to view possible NZV Variables:
myDataNZV <- nearZeroVar(myTraining, saveMetrics=TRUE)
# Another subset of NZV variables.
myNZVvars <- names(myTraining) %in% c("new_window", "kurtosis_roll_belt", "kurtosis_picth_belt",
"kurtosis_yaw_belt", "skewness_roll_belt", "skewness_roll_belt.1", "skewness_yaw_belt",
"max_yaw_belt", "min_yaw_belt", "amplitude_yaw_belt", "avg_roll_arm", "stddev_roll_arm",
"var_roll_arm", "avg_pitch_arm", "stddev_pitch_arm", "var_pitch_arm", "avg_yaw_arm",
"stddev_yaw_arm", "var_yaw_arm", "kurtosis_roll_arm", "kurtosis_picth_arm",
"kurtosis_yaw_arm", "skewness_roll_arm", "skewness_pitch_arm", "skewness_yaw_arm",
"max_roll_arm", "min_roll_arm", "min_pitch_arm", "amplitude_roll_arm", "amplitude_pitch_arm",
"kurtosis_roll_dumbbell", "kurtosis_picth_dumbbell", "kurtosis_yaw_dumbbell", "skewness_roll_dumbbell",
"skewness_pitch_dumbbell", "skewness_yaw_dumbbell", "max_yaw_dumbbell", "min_yaw_dumbbell",
"amplitude_yaw_dumbbell", "kurtosis_roll_forearm", "kurtosis_picth_forearm", "kurtosis_yaw_forearm",
"skewness_roll_forearm", "skewness_pitch_forearm", "skewness_yaw_forearm", "max_roll_forearm",
"max_yaw_forearm", "min_roll_forearm", "min_yaw_forearm", "amplitude_roll_forearm",
"amplitude_yaw_forearm", "avg_roll_forearm", "stddev_roll_forearm", "var_roll_forearm",
"avg_pitch_forearm", "stddev_pitch_forearm", "var_pitch_forearm", "avg_yaw_forearm",
"stddev_yaw_forearm", "var_yaw_forearm")

myTraining <- myTraining[!myNZVvars]
dim(myTraining)
```

```{r}
# Transformation 2: Killing first column of Dataset - Removing first ID variable so that it does not interfer with ML Algorithms:
myTraining <- myTraining[c(-1)]

# Transformation 3: Cleaning Variables with too many NAs. Removing variables that have more than a 60% threshold of NA’s:
trainingV3 <- myTraining ## creating another subset to iterate in loop
for(i in 1:length(myTraining)) { ## for every column in the training dataset
        if( sum( is.na( myTraining[, i] ) ) /nrow(myTraining) >= .6 ) { ## if n?? NAs > 60% of total observations
        for(j in 1:length(trainingV3)) {
            if( length( grep(names(myTraining[i]), names(trainingV3)[j]) ) ==1)  { ## if the columns are the same:
                trainingV3 <- trainingV3[ , -j]}}}} ## remove that column 

# To check the new N?? of observations
dim(trainingV3)
# Setting back to our set:
myTraining <- trainingV3
rm(trainingV3)
```

Now let us do the exact same 3 transformations for myTesting and testing data sets.

```{r}
clean1 <- colnames(myTraining)
clean2 <- colnames(myTraining[, -58]) #already with classe column removed
myTesting <- myTesting[clean1]
testing <- testing[clean2]
# To check the new N?? of observations
dim(myTesting)
# To check the new N?? of observations
dim(testing)
```

To ensure proper functioning of Decision Trees and especially RandomForest Algorithm with the Test data set (data set provided), we need to coerce the data into the same type.

```{r}
for (i in 1:length(testing) ){
        for(j in 1:length(myTraining)){
        if( length( grep(names(myTraining[i]), names(testing)[j]) ) ==1){
            class(testing[j]) <- class(myTraining[i])}}}
# To make sure Coertion really worked, simple smart ass technique:
testing <- rbind(myTraining[2, -58] , testing) ## note row 2 does not mean anything, this will be removed right.. now:
testing <- testing[-1,]
```

## Using ML algorithms for prediction

### Decision Tree

```{r}
modFitA1 <- rpart(classe ~ ., data=myTraining, method="class")
# To view the decision tree with fancy
fancyRpartPlot(modFitA1)
```

### Predicting

```{r}
predictionsA1 <- predict(modFitA1, myTesting, type = "class")
# Using confusion Matrix to test results
confusionMatrix(predictionsA1, myTesting$classe)
```

### Random Forests

```{r}
modFitB1 <- randomForest(classe ~. , data=myTraining)
# Predicting in-sample error
predictionsB1 <- predict(modFitB1, myTesting, type = "class")
# Using confusion Matrix to test results
confusionMatrix(predictionsB1, myTesting$classe)
```

## Generating Answer files 

```{r}
# Finally, using the provided Test Set out-of-sample error, for Random Forests we use the following formula, which yields a much better prediction in sample
predictionsB2 <- predict(modFitB1, testing, type = "class")
# Function to generate files with predictions to submit for assignment

pml_write_files = function(x){
  n = length(x)
  for(i in 1:n){
    filename = paste0("problem_id_",i,".txt")
    write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)}}
pml_write_files(predictionsB2)
```

## Predicted answers for 20 different test cases
This tells which classe the problem IDs belong to:

Classe A: 2, 4, 5, 9, 10, 14, 17

Classe B: 1, 3, 8, 11, 13, 18, 19, 20

Classe C: 12

Classe D: 7

Classe E: 6, 15, 16
