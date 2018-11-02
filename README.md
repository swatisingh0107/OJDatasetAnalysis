# Orange Juice Case Study

## Introduction
The data contains 1070 purchases where the customer either purchased Citrus Hill or Minute Maid Orange Juice. A number of characteristics of the customer and product are recorded.

| Attribute | Descritpion |
| ----------|-------------|
| Purchase  | A factor with levels CH and MM indicating whether the customer purchased Citrus Hill or Minute Maid Orange Juice|
| WeekofPurchase|Week of purchase |
| StoreID | Store ID |
|PriceCH | Price charged for CH |
|PriceMM |Price charged for MM|
|DiscCH|Discount offered for CH|
|DiscMM|Discount offered for MM|
|SpecialCH|Indicator of special on CH|
|SpecialMM|Indicator of special on MM|
|LoyalCH|Customer brand loyalty for CH|
|SalePriceMM|Sale price for MM|
|SalePriceCH|Sale price for CH|
|PriceDiff|Sale price of MM less sale price of CH|
|Store7|A factor with levels No and Yes indicating whether the sale is at Store 7|
|PctDiscMM|Percentage discount for MM|
|PctDiscCH|Percentage discount for CH|
|ListPriceDiff|List price of MM less list price of CH|
|STORE|Which of 5 possible stores the sale occured at|

## Objective
Assumption is that there is higher margin on the sale of MM. The store chain wants to increase their overall revenue and would like to know the following:
1. What variables influnece the sales of MM? Based on the highly significant variable , the store can devise strategies to improve the sales of MM.
2. Can we have a predictive model to predict the probability of customers buying MM?

To achieve this aim, we will first perform exploratory data analysis to prepare the data for modelling. Later in this project, we will perform various machine learning algorithms to identify features for modelling and compare two prediction models : Logistic regression vs Support Vector Machine.

## Exploratory Data Analysis
Let's load some R packages to assist with our analysis

```{r}
#Data Management and Statistics
library(ISLR)
library(skimr)
#Data Visualization
library(ggplot2) 
library(GGally) 
#machine Learning Methods Packages
library(caret) 
```
### Data Statistics
```{r}
str(OJ)
skim(OJ)
```
![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/str(OJ).PNG "OJ dataset Structure")

![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/summary(OJ).PNG "Dataset Summary")

From the summary analysis of the nature of the values in different columns, we can see that SpecialCH, SpecialMM,Store, StoreID are rather categorical than numerical

 ![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/Skim(OJ).PNG "Skim OJ dataset")
 
On skiming the data, we see that all the varibales have complete values. While analyzing the histograms, PctDiscMM and DiscMM show similar trends in data. Similary, SpecialCH and SpecialMM appear to be collinear. It is the same story of PriceCH and PctDiscCH.

```{r}
table(OJ$STORE,OJ$StoreID)
```
On cross classification of the Store and StorID, we built a a contingency table of the counts at each combination of factor levels.
  
  ||1|2|3|4|7|
  |:--:|:--:|:--:|:--:|:--:|:--:|
  |0|0|0|0|0|356|
  |1|157|0|0|0|0|
  |2|0|222|0|0|0|
  |3|0|0|196|0|0|
  |4|0|0|0|139|0|
  
Looking at the customer counts for Store and StoreID, one can see that Store and StoreID are essentially the same. It seems that the identifier for Store7 in Store column is 0. Both have the same number of count.
Also, looking at the count of Store7 for yes values, we can deduce that 714 is the count of stores from 1-4.

|No| Yes| 
|:--:|:--:|
|714| 356 |

[Dimension Reduction](https://www.analyticsvidhya.com/blog/2015/07/dimension-reduction-methods/) refers to the process of converting a set of data having vast dimensions into data with lesser dimensions ensuring that it conveys similar information concisely. Before we move on with our modelling exercise, we will try to reduce the dimensions by removing information that is collinear, redundant or duplicate.  Store and Store7 duplicates the information captured in Store. Also we will transform some of the numerical variables into categorical variables.


```{r}
OJ<-OJ[, !(colnames(OJ) %in% c("Store7","STORE"))]

#Convert SpecialCH, SpecialMM, StoreID into categorical variables
OJ$StoreID<-as.factor(OJ$StoreID)
OJ$SpecialCH<-as.factor(OJ$SpecialCH)
OJ$SpecialMM<-as.factor(OJ$SpecialMM)
OJ$Purchase<-as.factor(OJ$Purchase)

#Data distribution for categorical variables
prop.table(table(OJ$SpecialCH,OJ$SpecialMM)) 
```

   ||0|1|
  |:--:|:--|:--|
  |0| 0.69| 0.15|
  |1| 0.14| 0.00|
  
Looking at the proportion of variables SpecialCH and SpecialMM, we can see that the case for special offers is very sparse.
```{r}
prop.table(table(OJ$Purchase)) # CH-MM 61%-39% split
```
 |CH|        MM|
 |:--|:--|
|0.6102804| 0.3897196|

61% of the customers purchased CH whereas 39% of the customers purchased MM. MM has a higher margin and if sales of MM is increased, the revenue will also increase. 

```{r}
OJ$PurchaseMM<-ifelse(OJ$Purchase=="MM",1,0)
OJ$PurchaseMM<-factor(OJ$PurchaseMM,levels=c(0,1))

```
Since this project requires to improve the sales of MM, we will modify the Purchase variable to reflect 1 when MM was purchased and 0 when MM was not purchased. This new inference is stored as new variable 'PurchaseMM'.
All categorical attributes have now been checked for collinearity. We will now focus on the numeric columns to study trends and patterns and analyze if any of the numeric attributes have correlation.

## Feature Selection
### Check for multi- collinearity
The function ggpairs() produces a matrix of plots for visualizing the correlation between variables.

```{r}
numericCols<-OJ[c(1,2,4,5,6,7,10,11,12,13,14,15,16)]
plotNumeric = ggpairs(numericCols,
            aes(colour = Purchase, alpha=0.9),
            upper = list(continuous = wrap("cor", size = 2)),
            diag = list(continuous = "barDiag"),
            lower = list(continuous = "smooth"))
suppressMessages(print(plotNumeric))

```
![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/GGPlotPairs.png "GGPlotPairs")

Based on the process of elimination where correlation>0.7, we identified followin pairs of collinearity variables

|Variable|	Correlation|
|:--|:--:
|PriceCH	|Week of purchase|
|SalePriceMM	|DiscMM |
|SalePriceCH	|DiscCH|
|PriceDiff	|DiscMM, SalePriceMM|
|PctDiscMM	|DiscMM, SalePriceMM,PriceDiff|
|PctDiscCH	|DiscCH, SalePriceCH|

Considering the result from ggpairs, if we eliminate the collinear pairs we get

**Featureset1 <- StoreID+PriceCH+LoyalCH+PctDiscMM+PctDiscCH+ListPriceDiff**

Now how do we decide which variables to eliminate. The importance can be estimated using a ROC curve analysis conducted for each attribute.

### Rank features by importance
```{r}
#Prepare train and test data
split=0.75
set.seed(79056099)

train_index <- sample(1:nrow(OJ), split * nrow(OJ)) 
test_index <- setdiff(1:nrow(OJ), train_index)

train_data <- OJ[train_index, ]
test_data <- OJ[test_index,]

# prepare training scheme
control <- trainControl(method="repeatedcv", number=10, repeats=3)
# train the model
model <- train(PurchaseMM~.-Purchase, data=train_data, method="lvq", preProcess=c("center","scale"), trControl=control)
# estimate variable importance
importance <- varImp(model, scale=FALSE)
# summarize importance
print(importance)
# plot importance
plot(importance)

```
![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/Variable%20Importance.PNG "Variable Importance")

Based on the above results of estimating variable importance and multi collinear pairs from the ggpairs, we can deduce that the few variables are of importance and the rest can be removed from our modelling purposes.

"LoyalCH" "PriceDiff" "StoreID" "ListPriceDiff" "WeekOfPurchase" "SpecialMM" "PriceMM" "SpecialCH" "SalePriceCH"
This will form our second feature set 2

**Featureset2<- LoyalCH+PriceDiff+StoreID+ListPriceDiff+WeekofPurchase+SpecialMM+PriceMM+SpecialCH+SalePriceCH**

### Random Forest Recursive Feature Elimination 

We can also use RFE algorithm to configure to explore all possible subsets of the attributes. 

```{r}
#define the control using a random forest selection function
set.seed(4)
ctrl <- rfeControl(functions = rfFuncs,method = "repeatedcv",repeats = 5,verbose = FALSE)
# run the RFE algorithm
lmProfile <- rfe(train_data[,2:16], train_data[,17],
                 sizes = 2:16,
                 rfeControl = ctrl)
# list the chosen features
predictors(lmProfile)
```
 "LoyalCH"   "StoreID"   "PriceDiff"
 
 ```{r}
# plot the results
plot(lmProfile, type=c("g", "o"))
```
![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/RFE.PNG)

All attributes are selected in this example, although in the plot showing the accuracy of the different attribute subset sizes, we can see that just 3 attributes gives almost comparable results. Based on this result, we will form our third featureset

**Featureset3 <- LoyalCH+StoreID+PriceDiff**

## Logistic regression
Now we are ready for regression analysis. We chose regression analysis because it is appropriate regression analysis to conduct when the dependent variable is dichotomous (binary). Logistic regression is used to describe data and to explain the relationship between one dependent binary variable and one or more nominal, ordinal, interval or ratio-level independent variables.[Reference](http://www.statisticssolutions.com/what-is-logistic-regression/)

```{r}
model1 <- train(PurchaseMM~StoreID+PriceCH+LoyalCH+PctDiscMM+PctDiscCH+ListPriceDiff, data = train_data,
                       method = "glm",preProcess=c("center","scale"), family = "binomial",trControl=control)
summary(model1)
```
![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/Model1.PNG)

```{r}
model2 <- train(PurchaseMM~LoyalCH+PriceDiff+StoreID+ListPriceDiff+WeekofPurchase+SpecialMM+PriceMM+SpecialCH+SalePriceCH, data = train_data,
                       method = "glm",preProcess=c("center","scale"), family = "binomial",trControl=control)
summary(model2)
```
![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/Model2.PNG)

```{r}
model3 <- train(PurchaseMM~LoyalCH+StoreID+PriceDiff, data = train_data,
                       method = "glm",preProcess=c("center","scale"), family = "binomial",trControl=control)
summary(model3)
```
![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/model3.PNG)

On comparing the three models, the AIC score for model three is the lowest at 614.56
We use the exponential function to calculate the odds ratios for each preditor.

|(Intercept)|     LoyalCH|    StoreID2|    StoreID3|    StoreID4|    StoreID7|   PriceDiff|
|:--|:--|:--|:--|:--|:--|:--|
|0.4456994|   0.1387189|   1.0157003|   1.0687629|   0.8774379|   0.7225689|   0.4642658| 

Also based on the results above we can see **PriceDiff**, **StoreID1**, **LoyalCH** has significant effect on the sales of MM. 
StoreID 1 ,Loyalty to CH and Price Difference between MM and CH , all have negative influence on the sales of MM. Since CH sells cheaper than MM, any unit increase in the Price Difference will have a negative effect on the probability of the purchase of MM.

We will now calculate the prediction accuracy of our model.

```{r}
Predict <- predict(model3,newdata=test_data)
accuracy<-table(Predict,test_data$PurchaseMM)
sum(diag(accuracy))/sum(accuracy)
confusionMatrix(data=Predict,test_data$PurchaseMM)
```
![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/Confusion%20Matrix%20Statistics.PNG)
We have been able to reach an accuracy of 82% with a 95% confidence interval of 76.96% to 86.49%

## SVM
Support Vector Machine with Linear Kernel
```{r}
svm_Linear <- train(PurchaseMM~LoyalCH+StoreID+PriceDiff, data = train_data, method = "svmLinear",
                 trControl=control,
                 preProcess = c("center", "scale"),
                 tuneLength = 10)

PredictSVMLinear <- predict(svm_Linear,newdata=test_data)
accuracy<-table(PredictSVMLinear,test_data$PurchaseMM)
sum(diag(accuracy))/sum(accuracy)
confusionMatrix(data=PredictSVMLinear,test_data$PurchaseMM)
```
![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/SVMLinearAccuracy.PNG)

Support Vector Machine with Radial Kernel
```{r}
grid_radial <- expand.grid(sigma = c(0.01, 0.02, 0.025, 0.03, 0.04,
 0.05, 0.06, 0.07,0.08, 0.09, 0.1, 0.25, 0.5, 0.75,0.9),
 C = c(0.01, 0.05, 0.1, 0.25, 0.5, 0.75,
 1, 1.5, 2,5))
svm_Radial <- train(PurchaseMM~LoyalCH+StoreID+PriceDiff, data = train_data, method = "svmRadial",
                 trControl=control,
                 preProcess = c("center", "scale"),
                 tuneGrid = grid_radial,
                 tuneLength = 10)

PredictSVMRadial <- predict(svm_Radial,newdata=test_data)
accuracy<-table(PredictSVMRadial,test_data$PurchaseMM)
sum(diag(accuracy))/sum(accuracy)
confusionMatrix(data=PredictSVMRadial,test_data$PurchaseMM)
```
![alt text](https://github.com/swatisingh0107/OJDatasetAnalysis/blob/master/images/SVMRadialAccuracy.PNG)

Both SVM Linear and Radial return slightly better accuracy than the logistic regression model. However SVM Linear and SVM Radial return same accuracy. In order to select which SVM kernel is better we will compare the sensitivity of the confusion matrix for the two.
The sensitivity is defined as the proportion of positive results out of the number of samples which were actually positive. Since the model is expected to predict true MM customers, the sensitivity gives us the proportion of predicted positive MM customers out of total number of true MM customers. SVM Radial has the highest sensitivity of 89.16%






