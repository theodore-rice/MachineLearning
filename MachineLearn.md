---
title: "Activity Monitoring"
author: "T Rice"
date: "September 22, 2015"
output: html_document
---

The goal of this project is to predict whether an activity was done correctly or incorrectly.  As one might expect, there is one correct way to do an exercise, but several incorrect ways.  The goal is to collect measurement data from participants doing several exercises both correctly and incorrectly and then later to predict whether the activity was done correctly or not from the measurements alone.

First we read in the data.  The data was downloaded an loaded into my working directory.  This data frame is the training data set.  Later I will apply my algorithm to the testing set.  We also load necessary packages.

```{r}
library(caret)
library(dplyr)
train<-read.csv("pml-training.csv",na.strings=c("NA","#DIV/0!"))

```
Many of the columns in the data frame are summary columns, like mean, average, kurtosis.  These are dependent on the non-summary columns and can be excluded.
```{r}
myvars<-c("roll_belt","pitch_belt","yaw_belt","total_accel_belt","roll_arm","pitch_arm","yaw_arm","total_accel_arm","roll_dumbbell","pitch_dumbbell","yaw_dumbbell","roll_forearm","pitch_forearm","yaw_forearm","classe")
PrunedTrain<-train[,myvars]
```
Some of the variables still have low variance, so I will remove those.
```{r}
nsv<-nearZeroVar(train)
  paredTrain<-PrunedTrain[,-nsv]
```
Now that the data frame is pre-processed, I will create   
training sets and testing sets to test the algorithm before applying it to the (final) testing set.
```{r}
inTrain<-createDataPartition(y=paredTrain$classe,p=.7,list=FALSE)
TrainingSet<-paredTrain[inTrain,]
TestingSet<-paredTrain[-inTrain,]
```
I want to create the model using random forests using cross-validation.  Since ultimately this is a decision-tree type problem, random forests seem most appropriate.
(Note the model is number 3 because in the process of working on this project, I created two earlier models that were not as accuate.  Since the code is all saved, it is easier to preserve the 3 than to try to change every symbol.)

```{r}
ctrl<-trainControl(method="cv")
modFit3<-train(classe~.,method="rf",data=TrainingSet,trControl=ctrl)
```
We can estimate the accuracy of this model by
```{r}
modFit3$results
```
This model appears to be very accurate.  Let's try it on the test set we created
```{r}
testPred<-predict(modFit3,TestingSet)
M<-table(testPred,TestingSet$classe)
print(M)
```
We can see what percent of the test entries are correct:
```{r}
sum(diag(M))/sum(M)
```
This model is very accurate with less than 1.5% error.
