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
'data.frame:	1070 obs. of  18 variables:'
 '$ Purchase      : Factor w/ 2 levels "CH","MM": 1 1 1 2 1 1 1 1 1 1 ...'
 '$ WeekofPurchase: num  237 239 245 227 228 230 232 234 235 238 ...'
 '$ StoreID       : num  1 1 1 1 7 7 7 7 7 7 ...'
 '$ PriceCH       : num  1.75 1.75 1.86 1.69 1.69 1.69 1.69 1.75 1.75 1.75 ...'
 '$ PriceMM       : num  1.99 1.99 2.09 1.69 1.69 1.99 1.99 1.99 1.99 1.99 ...'
 '$ DiscCH        : num  0 0 0.17 0 0 0 0 0 0 0 ...'
 '$ DiscMM        : num  0 0.3 0 0 0 0 0.4 0.4 0.4 0.4 ...'
 '$ SpecialCH     : num  0 0 0 0 0 0 1 1 0 0 ...'
 '$ SpecialMM     : num  0 1 0 0 0 1 1 0 0 0 ...'
 '$ LoyalCH       : num  0.5 0.6 0.68 0.4 0.957 ...'
 '$ SalePriceMM   : num  1.99 1.69 2.09 1.69 1.69 1.99 1.59 1.59 1.59 1.59 ...'
 '$ SalePriceCH   : num  1.75 1.75 1.69 1.69 1.69 1.69 1.69 1.75 1.75 1.75 ...'
 '$ PriceDiff     : num  0.24 -0.06 0.4 0 0 0.3 -0.1 -0.16 -0.16 -0.16 ...'
 '$ Store7        : Factor w/ 2 levels "No","Yes": 1 1 1 1 2 2 2 2 2 2 ...'
 '$ PctDiscMM     : num  0 0.151 0 0 0 ...'
 '$ PctDiscCH     : num  0 0 0.0914 0 0 ...'
 '$ ListPriceDiff : num  0.24 0.24 0.23 0 0 0.3 0.3 0.24 0.24 0.24 ...'
 '$ STORE         : num  1 1 1 1 0 0 0 0 0 0 ...'
 
 
