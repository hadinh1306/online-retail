---
title: "EDA"
author: "Ha Dinh"
date: '2018-08-02'
output: 
  html_document:
    keep_md: yes
---

This document is a step-by-step instruction of how to use data wrangling in R to segment customers using **R**ecency-**F**requency-**M**onetary Value method. 

# RMF and Customer Segmentation

RFM stands for: 

- RECENCY (R): Days since last purchase
- FREQUENCY (F): Total number of purchases
- MONETARY VALUE (M): Total money this customer spent

So how does RFM relate to customer segmentation? In this project, I choose 6 customer segemts that can be easily identified through RFM, which are Best Customers, Loyal Customers, Big Spenders, Almost Lost, Lost, and Lost Cheap Customers. Let's describe our segments in terms of RFM. 

- Best Customers are those who buy most recently (**R**) and most often (**F**), and spend the most (**M**).
- Loyal Customers are those who buy most frequently (**F**).
- Big Spenders are those who spend the most (**M**).
- Almost Lost Customers are those who haven't purchased for some time (**R**), but purchased frequently (**F**) and spend the most (**M**).
- Lost Customers are those who haven't purchased for longer time than `Almost Lost` ones (**R**), but purchased frequently (**F**) and spend the most (**M**). 
- Lost Cheap Customers are those who last purchased long ago (**R**), purchased few (**F**), and spent little (**M**).

My goal is to calculate recency, frequency and monetary value for each customer. Then I compute a score for each customer based on his/her RMF results, and assign customer segment based on his/her score. 

# Get set up for the tutorial

Before getting into coding, if you don't have R in your laptop, please follow this [instruction](https://github.com/hadinh1306/workshop-materials/blob/master/Misc/Download-instruction.md) to install R and RStudio.

Below are *packages* you need to load. Each package contains functions that are used for calculations and data manipulation in this tutorial. If you don't have these packages in R, please delete the `#` in front of `install.packages()` and run the code line to install each package. It will take sometimes to finish the installment, especially for `dplyr`. 

**Tips:**
- To run a specific line of code, put the cursor to that line, and press `Command + R` (for Mac) or `Ctrl + R` (for Windows). 
- To run multiple lines of code, highlight those lines, and press `Command + R` (for Mac) or `Ctrl + R` (for Windows).
- To run a whole code chunk, the fastest way is to click the green play button on the top right of your chunk. 


```r
knitr::opts_chunk$set(echo = TRUE)

# Install new packages

# install.packages("readr")
# install.packages("dplyr")
# install.packages("lubridate")

# Load packages

library(readr)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

Like I mentioned in the project's [README](../README.md), the data I use in this tutorial is from [Online Retail](https://www.kaggle.com/sanjeet41/online-retail) dataset on Kaggle. You can download it directly from [here](https://www.kaggle.com/sanjeet41/online-retail/downloads/online-retail.zip/3).

Now we load the data and look at the top 6 rows of the dataset.


```r
# Load data
df <- read_csv("../data/raw/online_retail.csv")

# Look at the top 6 rows
head(df)
```

```
## # A tibble: 6 x 8
##   InvoiceNo StockCode Description           Quantity InvoiceDate UnitPrice
##   <chr>     <chr>     <chr>                    <int> <chr>           <dbl>
## 1 536365    85123A    WHITE HANGING HEART …        6 12/1/10 8:…      2.55
## 2 536365    71053     WHITE METAL LANTERN          6 12/1/10 8:…      3.39
## 3 536365    84406B    CREAM CUPID HEARTS C…        8 12/1/10 8:…      2.75
## 4 536365    84029G    KNITTED UNION FLAG H…        6 12/1/10 8:…      3.39
## 5 536365    84029E    RED WOOLLY HOTTIE WH…        6 12/1/10 8:…      3.39
## 6 536365    22752     SET 7 BABUSHKA NESTI…        2 12/1/10 8:…      7.65
## # ... with 2 more variables: CustomerID <int>, Country <chr>
```

We have all packages and the data we need, let's get started.

# Data Structure and Validation

For any analysis project, understanding data structure and validating data are important. These tasks help us to see if we need to transform any variables for later analysis, or if we have missing values and any outliers that might affect our analysis. 


```r
# View structure of data 
str(df)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	240007 obs. of  8 variables:
##  $ InvoiceNo  : chr  "536365" "536365" "536365" "536365" ...
##  $ StockCode  : chr  "85123A" "71053" "84406B" "84029G" ...
##  $ Description: chr  "WHITE HANGING HEART T-LIGHT HOLDER" "WHITE METAL LANTERN" "CREAM CUPID HEARTS COAT HANGER" "KNITTED UNION FLAG HOT WATER BOTTLE" ...
##  $ Quantity   : int  6 6 8 6 6 2 6 6 6 32 ...
##  $ InvoiceDate: chr  "12/1/10 8:26" "12/1/10 8:26" "12/1/10 8:26" "12/1/10 8:26" ...
##  $ UnitPrice  : num  2.55 3.39 2.75 3.39 3.39 7.65 4.25 1.85 1.85 1.69 ...
##  $ CustomerID : int  17850 17850 17850 17850 17850 17850 17850 17850 17850 13047 ...
##  $ Country    : chr  "United Kingdom" "United Kingdom" "United Kingdom" "United Kingdom" ...
##  - attr(*, "spec")=List of 2
##   ..$ cols   :List of 8
##   .. ..$ InvoiceNo  : list()
##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
##   .. ..$ StockCode  : list()
##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
##   .. ..$ Description: list()
##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
##   .. ..$ Quantity   : list()
##   .. .. ..- attr(*, "class")= chr  "collector_integer" "collector"
##   .. ..$ InvoiceDate: list()
##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
##   .. ..$ UnitPrice  : list()
##   .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
##   .. ..$ CustomerID : list()
##   .. .. ..- attr(*, "class")= chr  "collector_integer" "collector"
##   .. ..$ Country    : list()
##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
##   ..$ default: list()
##   .. ..- attr(*, "class")= chr  "collector_guess" "collector"
##   ..- attr(*, "class")= chr "col_spec"
```

```r
# View statistical summary of each variable
summary(df)
```

```
##   InvoiceNo          StockCode         Description       
##  Length:240007      Length:240007      Length:240007     
##  Class :character   Class :character   Class :character  
##  Mode  :character   Mode  :character   Mode  :character  
##                                                          
##                                                          
##                                                          
##                                                          
##     Quantity         InvoiceDate          UnitPrice          CustomerID   
##  Min.   :-74215.00   Length:240007      Min.   :    0.00   Min.   :12346  
##  1st Qu.:     1.00   Class :character   1st Qu.:    1.25   1st Qu.:13842  
##  Median :     3.00   Mode  :character   Median :    2.10   Median :15132  
##  Mean   :     9.28                      Mean   :    5.12   Mean   :15275  
##  3rd Qu.:    10.00                      3rd Qu.:    4.21   3rd Qu.:16814  
##  Max.   : 74215.00                      Max.   :38970.00   Max.   :18287  
##                                                            NA's   :67225  
##    Country         
##  Length:240007     
##  Class :character  
##  Mode  :character  
##                    
##                    
##                    
## 
```

Looking at both data structure and statistical summary,  

1. I confirm that `UnitPrice` is larger than 0. `Quantity` has negative numbers which indicate returns.

2. I'm curious how many countries are there in the data, and how many transactions each country had.

There are 38 countries present in this dataset, but 92% of observations belong to United Kingdom. Customer segmentation in different countries can be different. Thus, for this project, I'll focus on United Kingdom only. 


```r
df %>% 
  group_by(Country) %>% 
  summarise(n = n()) %>% 
  arrange(desc(n))
```

```
## # A tibble: 38 x 2
##    Country             n
##    <chr>           <int>
##  1 United Kingdom 220279
##  2 Germany          4208
##  3 France           3642
##  4 EIRE             3034
##  5 Netherlands      1142
##  6 Spain            1142
##  7 Belgium           933
##  8 Switzerland       708
##  9 Australia         642
## 10 Portugal          624
## # ... with 28 more rows
```

Let's filter data to get only observations from United Kingdom.


```r
ukdf <- df %>% 
  filter(Country == "United Kingdom")
```

3. I see that there are 66,659 missing values in `CustomerID`. Since the purpose of this project is to segment customers using RFM methods, we need `CustomerID` to identify each customer. We can't identify through `InvoiceNo` since each customer can purchase more than 1 time. Thus, I'll remove all observations with missing `CustomerID`. 


```r
summary(ukdf)
```

```
##   InvoiceNo          StockCode         Description       
##  Length:220279      Length:220279      Length:220279     
##  Class :character   Class :character   Class :character  
##  Mode  :character   Mode  :character   Mode  :character  
##                                                          
##                                                          
##                                                          
##                                                          
##     Quantity         InvoiceDate          UnitPrice          CustomerID   
##  Min.   :-74215.00   Length:220279      Min.   :    0.00   Min.   :12346  
##  1st Qu.:     1.00   Class :character   1st Qu.:    1.25   1st Qu.:14227  
##  Median :     3.00   Mode  :character   Median :    2.10   Median :15498  
##  Mean   :     8.24                      Mean   :    5.04   Mean   :15540  
##  3rd Qu.:     9.00                      3rd Qu.:    4.21   3rd Qu.:16942  
##  Max.   : 74215.00                      Max.   :38970.00   Max.   :18287  
##                                                            NA's   :66659  
##    Country         
##  Length:220279     
##  Class :character  
##  Mode  :character  
##                    
##                    
##                    
## 
```


```r
nonadf <- ukdf %>% 
  filter(!is.na(CustomerID))
```

2. I need to transform `InvoiceDate` from character to date form, removing hour and minute information.


```r
nonadf$InvoiceDate <- as_date(mdy_hm(nonadf$InvoiceDate))
```

Now I look at how long the data was recorded.


```r
paste("The first date recorded in this dataset is", min(nonadf$InvoiceDate)) 
```

```
## [1] "The first date recorded in this dataset is 2010-12-01"
```

```r
paste("The last date recorded in this dataset is", max(nonadf$InvoiceDate))
```

```
## [1] "The last date recorded in this dataset is 2011-06-26"
```

3. I see that there might be outliers in `Quantity` and `UnitPrice`, so I want to check their distributions.

Ploting ECDF (empirical distribution function) graphs for both `Quantity` and `UnitPrice`, I see that there are less than 5 outliers for each variable.


```r
plot(ecdf(nonadf$Quantity), main = "ECDF plot of Quantity", 
                            xlab = "Quantity (units)",
                            ylab = "Cumulative Probability")
```

![](dataprep_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
plot(ecdf(nonadf$UnitPrice), main = "ECDF plot of Unit Price",
                             xlab = "Unit Price (£)",
                             ylab = "Cumulative Probability")
```

![](dataprep_files/figure-html/unnamed-chunk-9-2.png)<!-- -->

I'm curious which customer(s) related to the 2 outliers in `Quantity`. It turns out that they belong to a single customer. He/she purchased a lot and returned them all.


```r
nonadf %>% 
  filter(Quantity > 50000 | Quantity < -50000)
```

```
## # A tibble: 2 x 8
##   InvoiceNo StockCode Description           Quantity InvoiceDate UnitPrice
##   <chr>     <chr>     <chr>                    <int> <date>          <dbl>
## 1 541431    23166     MEDIUM CERAMIC TOP S…    74215 2011-01-18       1.04
## 2 C541433   23166     MEDIUM CERAMIC TOP S…   -74215 2011-01-18       1.04
## # ... with 2 more variables: CustomerID <int>, Country <chr>
```

Let's check out `UnitPrice` outliers. Although from the ECDF plot, it appears that there are only 3 outliers, base on this table below, there are actually 4. Three of them belong to customer `16029`. He/she bought a `POSTAGE` worths 8142.75 pounds and returned it, and also return a `Manual` worths 6930 pounds on the same day. 


```r
nonadf %>% 
  filter(UnitPrice > 5000)
```

```
## # A tibble: 4 x 8
##   InvoiceNo StockCode Description Quantity InvoiceDate UnitPrice
##   <chr>     <chr>     <chr>          <int> <date>          <dbl>
## 1 C551685   POST      POSTAGE           -1 2011-05-03       8143
## 2 551697    POST      POSTAGE            1 2011-05-03       8143
## 3 C551699   M         Manual            -1 2011-05-03       6930
## 4 C556445   M         Manual            -1 2011-06-10      38970
## # ... with 2 more variables: CustomerID <int>, Country <chr>
```

For now, I decide to keep all outliers since the purpose of this project is not to build any statistical models based on current data, but to do data wrangling and compute RFM score for each customer and come up with his/her segment.  

# Data Preparation

## Recency

To calculate days since last purchase, I need to know what is the last day recorded in this data. The result is June 26th 2011. I will take June 27th 2011 as the present time to calculate recency. 


```r
max(nonadf$InvoiceDate)
```

```
## [1] "2011-06-26"
```


```r
present = as_date("2011-06-27")
```

Now I calculate the recency table including `CustomerID` and `recency` - days since last purchase. 

Note that for this calculation, there is an assumption: 

> There are many reasons that can lead to a return and there is no record of these reasons in the dataset. Thus, as long as customers make a purchase, we count their good intention toward our business and account this into recency, no matter if they return or not.

Also note that there are 2,689 customers in this dataset, so we should see at max 2,689 observations for our recency, frequency and monetary value tables. 


```r
n_distinct(nonadf$CustomerID)
```

```
## [1] 2689
```

Below, we only have 2,652 observations for recency table. This is because I filter to only have positive `Quantity` before calculate recency, avoiding counting return days. There are 37 customers who only returned in this dataset. Since they must have purchased before but that data is not in this dataset, I will filter out these 37 customers in my analysis. 


```r
(recency_df <- nonadf %>% 
   # Filter to only have positive Quantity
  filter(Quantity > 0) %>% 
   # Calculate recency for each customer
  group_by(CustomerID) %>% 
  summarise(recency = as.integer(present - max(InvoiceDate))))
```

```
## # A tibble: 2,652 x 2
##    CustomerID recency
##         <int>   <int>
##  1      12346     160
##  2      12747      33
##  3      12748       4
##  4      12749      48
##  5      12820     161
##  6      12821      49
##  7      12823      89
##  8      12826      13
##  9      12829     171
## 10      12830       6
## # ... with 2,642 more rows
```

## Frequency

To calculate the total number of purchases, I need to consider number of returns as well. 

`Total number of purchases = number of purchases - number of returns`

Let's calculate the number of purchases for each `CustomerID`.


```r
(frequency_df <- nonadf %>% 
  # create new column to detect return; 
  # if return then 0, else (if purchase) then 1
  mutate(return = ifelse(Quantity < 0, 0, 1)) %>% 
  # calculate number of purchases for each customer
  group_by(CustomerID) %>% 
  summarise(frequency = sum(return)))
```

```
## # A tibble: 2,689 x 2
##    CustomerID frequency
##         <int>     <dbl>
##  1      12346      1.00
##  2      12747     55.0 
##  3      12748   1579   
##  4      12749     43.0 
##  5      12820     11.0 
##  6      12821      6.00
##  7      12823      3.00
##  8      12826     56.0 
##  9      12829     11.0 
## 10      12830     13.0 
## # ... with 2,679 more rows
```

## Monetary value

To calculate the total money each customer spend, I take the sum of his/her revenue and loss. 

`Revenue/Loss = Quantity * Unit price`


```r
(money_df <- nonadf %>% 
  group_by(CustomerID) %>% 
  summarise(monetary_value = sum(Quantity*UnitPrice)))
```

```
## # A tibble: 2,689 x 2
##    CustomerID monetary_value
##         <int>          <dbl>
##  1      12346            0  
##  2      12747         2091  
##  3      12748        11028  
##  4      12749          782  
##  5      12820          170  
##  6      12821           92.7
##  7      12823          994  
##  8      12826          812  
##  9      12829          253  
## 10      12830         2222  
## # ... with 2,679 more rows
```

## RFM table 

Now I combine results from recency, frequency, and monetary value tables to a single RFM table. 


```r
# We don't need to state which variable(s) these tables are joining on
# because they have a common variable `CustomerID`
rfm_df <- inner_join(recency_df, frequency_df) %>% 
  inner_join(money_df)
```

```
## Joining, by = "CustomerID"
## Joining, by = "CustomerID"
```

```r
# View first 5 observations of RFM table
head(rfm_df, 5)
```

```
## # A tibble: 5 x 4
##   CustomerID recency frequency monetary_value
##        <int>   <int>     <dbl>          <dbl>
## 1      12346     160      1.00              0
## 2      12747      33     55.0            2091
## 3      12748       4   1579             11028
## 4      12749      48     43.0             782
## 5      12820     161     11.0             170
```

Customer `12346` made only 1 purchase 160 days ago, and returned his/her purchase. This customer is certainly not our best one. Let's see what he/she purchased and returned. 

It was a medium ceramic top storage jar of 1.04 pounds per unit. This seems like a cheap price. The customer purchased 74215 units but returned them all.


```r
nonadf %>% 
  filter(CustomerID == 12346)
```

```
## # A tibble: 2 x 8
##   InvoiceNo StockCode Description           Quantity InvoiceDate UnitPrice
##   <chr>     <chr>     <chr>                    <int> <date>          <dbl>
## 1 541431    23166     MEDIUM CERAMIC TOP S…    74215 2011-01-18       1.04
## 2 C541433   23166     MEDIUM CERAMIC TOP S…   -74215 2011-01-18       1.04
## # ... with 2 more variables: CustomerID <int>, Country <chr>
```

Like I mentioned in the beginning, I want to assign score for recency, frequency, and monetary value. These scores will help me to assign segment for each customer. I will assign 4 scores (1, 2, 3, 4) with 1 representing the best, and 4 representing the worst.

For recency, the best scenario is for the lowest recency, meaning a customer purchased something very recent.  

For both frequency and monetary value, the best scenario is for the highest frequency and value, meaning a customer purchased many time and spent a lot of money. 

One of the easiest way to assign score is based on quantiles. Here, I'll use 25%, 50%, and 75% quantiles. 


```r
rq <- quantile(rfm_df$recency, c(0.25, 0.5, 0.75))
fq <- quantile(rfm_df$frequency, c(0.25, 0.5, 0.75))
mq <- quantile(rfm_df$monetary_value, c(0.25, 0.5, 0.75))

rfm_df <- rfm_df %>% 
  # rq[[1]] takes the 25%, rq[[2]] takes the 50%, rq[[3]] takes the 75% quantile
  mutate(R = ifelse(recency <= rq[[1]], 1,
                                  ifelse(recency <= rq[[2]], 2,
                                         ifelse(recency <= rq[[3]], 3, 4)))) %>% 
  # Have to put F inside a quote 
  # because without a quote, it can be mistaken as the abbreviation of FALSE
  mutate('F' = ifelse(frequency <= fq[[1]], 4, 
                      ifelse(frequency <= fq[[2]], 3, 
                             ifelse(frequency <= fq[[3]], 2, 1)))) %>% 
  mutate(M = ifelse(monetary_value <= mq[[1]], 4, 
                      ifelse(monetary_value <= mq[[2]], 3, 
                             ifelse(monetary_value <= mq[[3]], 2, 1))))
```

Now that we have scores for recency, frequency, and monetary value, let's find scores for each segment. 

| Segment | Description | RFM score | 
|---------|-------------|-----------|
| Best Customers | Buy most recently and most often, and spend the most | 1, 1, 1 | 
| Loyal Customers | Buy most frequently | ?, 1, ? | 
| Big Spenders | Spend the most | ?, ?, 1 | 
| Almost Lost | Haven't purchased for some time, but purchased frequently and spend the most | 3, 1, 1 | 
| Lost Customers | Haven't purchased for longer time than `Almost Lost` ones, but purchased frequently and spend the most | 4, 1, 1 | 
| Lost Cheap Customers | Last purchased long ago, purchased few, and spent little | 4, 4, 4 | 

For Loyal Customers and Big Spenders, `?` can either be 1 to 4, as long as RFM combination is not similar to other segments. 

Let's asign segment for each customer. Customers who are not in my list of segments are assigned to `Others` group for now. 


```r
rfm_df <- rfm_df %>%
  mutate(segment = ifelse(R == 1 & F == 1 & M == 1, "Best Customer", 
                          ifelse(R == 3 & F == 1 & M == 1, "Almost Lost",
                                 ifelse(R == 4 & F == 1 & M == 1, "Lost Customers",
                                        ifelse(R == 4 & F == 4 & M == 4, "Lost Cheap Customers", 
                                               ifelse(M == 1, "Big Spenders", 
                                                      ifelse(F == 1, "Loyal Customers", "Others")))))))
```

# Export Data

Congratulations! You have assigned segment for each customer. We need to export data for visualizations. 

There are 2 files I want to export: 

1. A table which includes all customer transactions, along with customer segments. This table helps to visualize segment performance - how each customer segment contributes to our company's revenue.


```r
output_df <- rfm_df %>% 
  select(CustomerID, segment) %>% 
  inner_join(nonadf)
```

```
## Joining, by = "CustomerID"
```

```r
# Export data as csv file
write_csv(output_df, "../data/clean/segmented_data.csv")
```

2. A table which includes RFM data to investigate if the way we segment customers make sense. 


```r
write_csv(rfm_df, "../data/clean/rfm.csv")
```


