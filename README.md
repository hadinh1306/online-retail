# Customer segmentation using RFM

## **Project Objectives**

There are many techniques to segment customers. There are machine learning methods such as [*K-Nearest Neighbor*](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm) (supervised learning), or [*K-Means clustering*](https://en.wikipedia.org/wiki/K-means_clustering) (unsupervised learning). Another method that is widely used in Database Marketing and Direct Marketing, but does not relate to Machine Learning/Artificial Intelligence is [*RFM*](https://en.wikipedia.org/wiki/RFM_(customer_value)), standing for Recency - Frequency - Monetary Value. In this project, I want to show step by step how data wrangling in R can help marketers to do customer segmentation using *RFM* method. 

Many marketers nowadays use Tableau to visualize data due to its interactive, user-friendly interface and no coding background requirement. Visualizations on Tableau can be turned to informative reports for different stakeholders with different backgrounds. Thus, after getting the customer segments data, I will build Tableau dashboards to visualize these segments' value to business. 

## **Data Source**

In this project, I work with the [*Online Retail*](https://www.kaggle.com/sanjeet41/online-retail) data.

There are 240,007 observations of 8 variables as follow:

| Variable name | Description | Data type | 
|---------------|-------------|-----------|
| InvoiceNo     | Invoice number, with 12,468 unique values | `character` | 
| StockCode     | Stock code, with 3,645 unique values | `character` | 
| Description   | Product name, with 3,606 unique values | `character` | 
| Quantity      | Number or product(s) ordered          | `integer` | 
| InvoiceDate   | Date and time that an order was placed | `character` | 
| UnitPrice     | Price per unit of the product ordered | `double` |
| CustomerID    | Unique identifier for each customer, with 2,975 unique values | `character` |
| Country       | Country that the order was placed, with 38 unique values | `character` |

## **Project Deliveries**

1. Step by step [instruction](doc/dataprep.md) for data wrangling
2. Tableau Dashboard (TBD)

## **Project Directory**

```
|- data
|  -- raw                     Includes raw data
|  -- clean                   Includes clean data
|- doc                        Includes step-by-step instruction and other documentary 
|- src                        Includes source codes for this project
|- LICENSE                    MIT License
|- online-retail.Rproj        R project
|- README.md                  Project Overview
```



