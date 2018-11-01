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
This project is aimed to analyze the following:
1. What variables influnece the sale of MM?
2. Can we have a predictive model to predict the probability of customers buying MM?

To achieve this aim, we will first perform exploratory data analysis to prepare the data for modelling. Later in the project, we will perform various machine learning algorithms to identify features for modelling and compare two prediction models : Logistic regression vs Support Vector Machine.

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

