---
layout: post
title: E-commerce Data Analysis Part Two - Customer Cohort Analysis 
permlink: /Customer_Cohort_Analysis
---


## Introduction: 
In E-commerce Data Series Part Two, we will mainly focus on customer cohort analysis. In part one, we already went through data cleaning process, so we would follow the same steps here.  There are many benefits to adopt cohort analysis. In my personal opinion, the main goal we are trying to achieve in Cohort Analysis is form a better understanding of how customer behavior have affect on business. 

In this Ecommerce case, we would like to know: 
1. How much revenue new customers bring to the business compared to the existing customers? 
2. What's the customer retention rate for different cohort group?   

Why these two question is so important to this business? 
Because it's more expensive to acquire a new customer than keeping a warm customer who pruchased items on the business. We would love to see our customer base expanding over time and re-curring revenue increasing.  

## Table Of Content: 
  * Data Wrangling
  * New vs. Existing Customer
  * Cohort Analysis
  

## Data Wrangling - Exploratory Data Analysis


```python
# Setting up the environment
import pandas as pd
import numpy as np
import datetime as dt
import matplotlib.pyplot as plt
from operator import attrgetter
import seaborn as sns
import matplotlib.colors as mcolors

# Import Data
data = pd.read_csv("OnlineRetail.csv",engine="python")
data.info()

# Change InvoiceDate data type
data['InvoiceDate'] = pd.to_datetime(data['InvoiceDate'])
# Convert to InvoiceDate hours to the same mid-night time. 
data['date']=data['InvoiceDate'].dt.normalize()

# Summerize the null value in dataframe. 
print(data.isnull().sum())
# Missing values percentage
missing_percentage = data.isnull().sum() / data.shape[0] * 100

```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 541909 entries, 0 to 541908
    Data columns (total 8 columns):
    InvoiceNo      541909 non-null object
    StockCode      541909 non-null object
    Description    540455 non-null object
    Quantity       541909 non-null int64
    InvoiceDate    541909 non-null object
    UnitPrice      541909 non-null float64
    CustomerID     406829 non-null float64
    Country        541909 non-null object
    dtypes: float64(2), int64(1), object(5)
    memory usage: 33.1+ MB
    InvoiceNo           0
    StockCode           0
    Description      1454
    Quantity            0
    InvoiceDate         0
    UnitPrice           0
    CustomerID     135080
    Country             0
    date                0
    dtype: int64



```python
data.loc[data.CustomerID.isnull(), ["UnitPrice", "Quantity"]].describe()
```




<div style="overflow-x:auto;">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>UnitPrice</th>
      <th>Quantity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>count</td>
      <td>135080.000000</td>
      <td>135080.000000</td>
    </tr>
    <tr>
      <td>mean</td>
      <td>8.076577</td>
      <td>1.995573</td>
    </tr>
    <tr>
      <td>std</td>
      <td>151.900816</td>
      <td>66.696153</td>
    </tr>
    <tr>
      <td>min</td>
      <td>-11062.060000</td>
      <td>-9600.000000</td>
    </tr>
    <tr>
      <td>25%</td>
      <td>1.630000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>50%</td>
      <td>3.290000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <td>75%</td>
      <td>5.450000</td>
      <td>3.000000</td>
    </tr>
    <tr>
      <td>max</td>
      <td>17836.460000</td>
      <td>5568.000000</td>
    </tr>
  </tbody>
</table>
</div>



It is fair to say 'UnitePirce' and 'Quantity' column shows extreme outliers. As we need to use historical purhcase price and quantity to make sale forcast in the future, these outliers can be disruptive.  Therefore, let us dorp of the null values. Besides dropping all the missing values in description column. How about the hidden values? like 'nan' instead of 'NaN'. In the next following steps we will drop any rows with 'nan' and '' strings in it. 

## Data Cleaning - Dropping Rows With Missing Values


```python
# Can we find any hidden Null values? "nan"-Strings? in Description
data.loc[data.Description.isnull()==False, "lowercase_descriptions"] =  data.loc[
    data.Description.isnull()==False,"Description"].apply(lambda l: l.lower())

data.lowercase_descriptions.dropna().apply(
    lambda l: np.where("nan" in l, True, False)).value_counts()
```




    False    539724
    True        731
    Name: lowercase_descriptions, dtype: int64




```python
# How about "" in Description?
data.lowercase_descriptions.dropna().apply(
    lambda l: np.where("" == l, True, False)).value_counts()
```




    False    540455
    Name: lowercase_descriptions, dtype: int64




```python
# Transform "nan" toward "NaN"
data.loc[data.lowercase_descriptions.isnull()==False, "lowercase_descriptions"] = data.loc[
    data.lowercase_descriptions.isnull()==False, "lowercase_descriptions"
].apply(lambda l: np.where("nan" in l, None, l))

# Verified all the 'nan' changed into 'Nan'
data.lowercase_descriptions.dropna().apply(lambda l: np.where("nan" in l, True, False)).value_counts()
```




    False    539724
    Name: lowercase_descriptions, dtype: int64




```python
#drop all the null values and hidden-null values
data.loc[(data.CustomerID.isnull()==False)&(data.lowercase_descriptions.isnull()==False)].info()

# data without any null values
data_clean = data.loc[(data.CustomerID.isnull()==False)&(data.lowercase_descriptions.isnull()==False)].copy()
data_clean.isnull().sum().sum()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 406223 entries, 0 to 541908
    Data columns (total 10 columns):
    InvoiceNo                 406223 non-null object
    StockCode                 406223 non-null object
    Description               406223 non-null object
    Quantity                  406223 non-null int64
    InvoiceDate               406223 non-null datetime64[ns]
    UnitPrice                 406223 non-null float64
    CustomerID                406223 non-null float64
    Country                   406223 non-null object
    date                      406223 non-null datetime64[ns]
    lowercase_descriptions    406223 non-null object
    dtypes: datetime64[ns](2), float64(2), int64(1), object(5)
    memory usage: 34.1+ MB





    0



## New Customer Vs Existing
In this Project, we defined the Existing customers as the group of records with lastest $FirstConversion$ later than the $FirstConversionYearMonth$. In other words, a existing customer should have purchase history with InvoiceDate later than their first purchase date. Since we only have part of the Dec data, a lower than average monthly revenue number from Dec 2011 was expected. 


```python
# Create Revenue Column
data_clean['Revenue'] = data_clean['UnitPrice'] * data_clean['Quantity']

# Create YearMonth Column 
data_clean['YearMonth'] = data_clean['InvoiceDate'].apply(lambda date: date.year*100+date.month)

# New Vs. Existing
first_conversion = data_clean.groupby('CustomerID').agg({'InvoiceDate': lambda x: x.min()}).reset_index()
first_conversion.columns = ['CustomerID','FirstConversion']

# Change the Strings in FirstConversion Column into 'YearMonth' format number. 
first_conversion['FirstConversionYearMonth'] = first_conversion['FirstConversion'].apply\
(lambda date: date.year*100 + date.month)

first_conversion.head()
```




<div style="overflow-x:auto;">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CustomerID</th>
      <th>FirstConversion</th>
      <th>FirstConversionYearMonth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>12346.0</td>
      <td>2011-01-18 10:01:00</td>
      <td>201101</td>
    </tr>
    <tr>
      <td>1</td>
      <td>12347.0</td>
      <td>2010-12-07 14:57:00</td>
      <td>201012</td>
    </tr>
    <tr>
      <td>2</td>
      <td>12348.0</td>
      <td>2010-12-16 19:09:00</td>
      <td>201012</td>
    </tr>
    <tr>
      <td>3</td>
      <td>12349.0</td>
      <td>2011-11-21 09:51:00</td>
      <td>201111</td>
    </tr>
    <tr>
      <td>4</td>
      <td>12350.0</td>
      <td>2011-02-02 16:01:00</td>
      <td>201102</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Merge first_conversion with data_clean
df_customer = pd.merge(data_clean, first_conversion, on='CustomerID')
df_customer.head()
```




<div style="overflow-x:auto;">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>InvoiceNo</th>
      <th>StockCode</th>
      <th>Description</th>
      <th>Quantity</th>
      <th>InvoiceDate</th>
      <th>UnitPrice</th>
      <th>CustomerID</th>
      <th>Country</th>
      <th>date</th>
      <th>lowercase_descriptions</th>
      <th>Revenue</th>
      <th>YearMonth</th>
      <th>FirstConversion</th>
      <th>FirstConversionYearMonth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>536365</td>
      <td>85123A</td>
      <td>WHITE HANGING HEART T-LIGHT HOLDER</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>2.55</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
      <td>white hanging heart t-light holder</td>
      <td>15.30</td>
      <td>201012</td>
      <td>2010-12-01 08:26:00</td>
      <td>201012</td>
    </tr>
    <tr>
      <td>1</td>
      <td>536365</td>
      <td>71053</td>
      <td>WHITE METAL LANTERN</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>3.39</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
      <td>white metal lantern</td>
      <td>20.34</td>
      <td>201012</td>
      <td>2010-12-01 08:26:00</td>
      <td>201012</td>
    </tr>
    <tr>
      <td>2</td>
      <td>536365</td>
      <td>84406B</td>
      <td>CREAM CUPID HEARTS COAT HANGER</td>
      <td>8</td>
      <td>2010-12-01 08:26:00</td>
      <td>2.75</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
      <td>cream cupid hearts coat hanger</td>
      <td>22.00</td>
      <td>201012</td>
      <td>2010-12-01 08:26:00</td>
      <td>201012</td>
    </tr>
    <tr>
      <td>3</td>
      <td>536365</td>
      <td>84029G</td>
      <td>KNITTED UNION FLAG HOT WATER BOTTLE</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>3.39</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
      <td>knitted union flag hot water bottle</td>
      <td>20.34</td>
      <td>201012</td>
      <td>2010-12-01 08:26:00</td>
      <td>201012</td>
    </tr>
    <tr>
      <td>4</td>
      <td>536365</td>
      <td>84029E</td>
      <td>RED WOOLLY HOTTIE WHITE HEART.</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>3.39</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
      <td>red woolly hottie white heart.</td>
      <td>20.34</td>
      <td>201012</td>
      <td>2010-12-01 08:26:00</td>
      <td>201012</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_customer['customer_type'] = 'New'
df_customer.loc[df_customer['YearMonth']>df_customer['FirstConversionYearMonth'],'customer_type']='Existing'
df_customer.loc[df_customer['customer_type']=='Existing']      

df_customer_revenue = df_customer.groupby(['customer_type','YearMonth']).agg({'Revenue':'sum'}).reset_index()


sns.catplot(x="YearMonth", y="Revenue", data=df_customer_revenue, kind="point", aspect=2.5, \
            palette="YlGnBu",hue= "customer_type")

plt.ylabel('Revenue')
plt.xlabel('')
sns.despine(left=True, bottom=True)
plt.title('Monthly Revenue (New vs Existing)');
```


![](/public/image/new.vs.existing.png)


## Cohort Analysis:

In this project, we define the cohort group as the customer who purchase on-line within the same months. To translate this idea into cohort analysis, this means we need to group people by their 'CustomerID' and 'InvoiceDate'. 


```python
# create two variables: month of order and cohort
data_clean['order_month'] = data_clean['InvoiceDate'].dt.to_period('M')
data_clean['cohort'] = data_clean.groupby('CustomerID')['InvoiceDate'] \
                       .transform('min') \
                       .dt.to_period('M')
data_clean
```




<div style="overflow-x:auto;">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>InvoiceNo</th>
      <th>StockCode</th>
      <th>Description</th>
      <th>Quantity</th>
      <th>InvoiceDate</th>
      <th>UnitPrice</th>
      <th>CustomerID</th>
      <th>Country</th>
      <th>date</th>
      <th>lowercase_descriptions</th>
      <th>Revenue</th>
      <th>YearMonth</th>
      <th>order_month</th>
      <th>cohort</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>536365</td>
      <td>85123A</td>
      <td>WHITE HANGING HEART T-LIGHT HOLDER</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>2.55</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
      <td>white hanging heart t-light holder</td>
      <td>15.30</td>
      <td>201012</td>
      <td>2010-12</td>
      <td>2010-12</td>
    </tr>
    <tr>
      <td>1</td>
      <td>536365</td>
      <td>71053</td>
      <td>WHITE METAL LANTERN</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>3.39</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
      <td>white metal lantern</td>
      <td>20.34</td>
      <td>201012</td>
      <td>2010-12</td>
      <td>2010-12</td>
    </tr>
    <tr>
      <td>2</td>
      <td>536365</td>
      <td>84406B</td>
      <td>CREAM CUPID HEARTS COAT HANGER</td>
      <td>8</td>
      <td>2010-12-01 08:26:00</td>
      <td>2.75</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
      <td>cream cupid hearts coat hanger</td>
      <td>22.00</td>
      <td>201012</td>
      <td>2010-12</td>
      <td>2010-12</td>
    </tr>
    <tr>
      <td>3</td>
      <td>536365</td>
      <td>84029G</td>
      <td>KNITTED UNION FLAG HOT WATER BOTTLE</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>3.39</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
      <td>knitted union flag hot water bottle</td>
      <td>20.34</td>
      <td>201012</td>
      <td>2010-12</td>
      <td>2010-12</td>
    </tr>
    <tr>
      <td>4</td>
      <td>536365</td>
      <td>84029E</td>
      <td>RED WOOLLY HOTTIE WHITE HEART.</td>
      <td>6</td>
      <td>2010-12-01 08:26:00</td>
      <td>3.39</td>
      <td>17850.0</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
      <td>red woolly hottie white heart.</td>
      <td>20.34</td>
      <td>201012</td>
      <td>2010-12</td>
      <td>2010-12</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>541904</td>
      <td>581587</td>
      <td>22613</td>
      <td>PACK OF 20 SPACEBOY NAPKINS</td>
      <td>12</td>
      <td>2011-12-09 12:50:00</td>
      <td>0.85</td>
      <td>12680.0</td>
      <td>France</td>
      <td>2011-12-09</td>
      <td>pack of 20 spaceboy napkins</td>
      <td>10.20</td>
      <td>201112</td>
      <td>2011-12</td>
      <td>2011-08</td>
    </tr>
    <tr>
      <td>541905</td>
      <td>581587</td>
      <td>22899</td>
      <td>CHILDREN'S APRON DOLLY GIRL</td>
      <td>6</td>
      <td>2011-12-09 12:50:00</td>
      <td>2.10</td>
      <td>12680.0</td>
      <td>France</td>
      <td>2011-12-09</td>
      <td>children's apron dolly girl</td>
      <td>12.60</td>
      <td>201112</td>
      <td>2011-12</td>
      <td>2011-08</td>
    </tr>
    <tr>
      <td>541906</td>
      <td>581587</td>
      <td>23254</td>
      <td>CHILDRENS CUTLERY DOLLY GIRL</td>
      <td>4</td>
      <td>2011-12-09 12:50:00</td>
      <td>4.15</td>
      <td>12680.0</td>
      <td>France</td>
      <td>2011-12-09</td>
      <td>childrens cutlery dolly girl</td>
      <td>16.60</td>
      <td>201112</td>
      <td>2011-12</td>
      <td>2011-08</td>
    </tr>
    <tr>
      <td>541907</td>
      <td>581587</td>
      <td>23255</td>
      <td>CHILDRENS CUTLERY CIRCUS PARADE</td>
      <td>4</td>
      <td>2011-12-09 12:50:00</td>
      <td>4.15</td>
      <td>12680.0</td>
      <td>France</td>
      <td>2011-12-09</td>
      <td>childrens cutlery circus parade</td>
      <td>16.60</td>
      <td>201112</td>
      <td>2011-12</td>
      <td>2011-08</td>
    </tr>
    <tr>
      <td>541908</td>
      <td>581587</td>
      <td>22138</td>
      <td>BAKING SET 9 PIECE RETROSPOT</td>
      <td>3</td>
      <td>2011-12-09 12:50:00</td>
      <td>4.95</td>
      <td>12680.0</td>
      <td>France</td>
      <td>2011-12-09</td>
      <td>baking set 9 piece retrospot</td>
      <td>14.85</td>
      <td>201112</td>
      <td>2011-12</td>
      <td>2011-08</td>
    </tr>
  </tbody>
</table>
<p>406223 rows × 14 columns</p>
</div>




```python
# add total number of customers for every time period. 
df_cohort = data_clean.groupby(['cohort', 'order_month']) \
              .agg(n_customers=('CustomerID', 'nunique')) \
              .reset_index(drop=False)

# add an indicator for periods (months since first purchase)
df_cohort['period_number'] = (df_cohort.order_month - df_cohort.cohort).\
                              apply(attrgetter('n'))
df_cohort.head(5)
```




<div style="overflow-x:auto;">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cohort</th>
      <th>order_month</th>
      <th>n_customers</th>
      <th>period_number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>2010-12</td>
      <td>2010-12</td>
      <td>948</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2010-12</td>
      <td>2011-01</td>
      <td>362</td>
      <td>1</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2010-12</td>
      <td>2011-02</td>
      <td>317</td>
      <td>2</td>
    </tr>
    <tr>
      <td>3</td>
      <td>2010-12</td>
      <td>2011-03</td>
      <td>367</td>
      <td>3</td>
    </tr>
    <tr>
      <td>4</td>
      <td>2010-12</td>
      <td>2011-04</td>
      <td>341</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
# pivot the data into a form of the matrix
cohort_pivot = df_cohort.pivot_table(index='cohort',
                                     columns = 'period_number',
                                     values = 'n_customers')
cohort_pivot
```




<div style="overflow-x:auto;">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>period_number</th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>11</th>
      <th>12</th>
    </tr>
    <tr>
      <th>cohort</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2010-12</td>
      <td>948.0</td>
      <td>362.0</td>
      <td>317.0</td>
      <td>367.0</td>
      <td>341.0</td>
      <td>376.0</td>
      <td>360.0</td>
      <td>336.0</td>
      <td>336.0</td>
      <td>374.0</td>
      <td>354.0</td>
      <td>474.0</td>
      <td>260.0</td>
    </tr>
    <tr>
      <td>2011-01</td>
      <td>421.0</td>
      <td>101.0</td>
      <td>119.0</td>
      <td>102.0</td>
      <td>138.0</td>
      <td>126.0</td>
      <td>110.0</td>
      <td>108.0</td>
      <td>131.0</td>
      <td>146.0</td>
      <td>155.0</td>
      <td>63.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-02</td>
      <td>380.0</td>
      <td>94.0</td>
      <td>73.0</td>
      <td>106.0</td>
      <td>102.0</td>
      <td>94.0</td>
      <td>97.0</td>
      <td>107.0</td>
      <td>98.0</td>
      <td>119.0</td>
      <td>34.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-03</td>
      <td>440.0</td>
      <td>84.0</td>
      <td>112.0</td>
      <td>96.0</td>
      <td>102.0</td>
      <td>78.0</td>
      <td>116.0</td>
      <td>105.0</td>
      <td>127.0</td>
      <td>39.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-04</td>
      <td>299.0</td>
      <td>68.0</td>
      <td>66.0</td>
      <td>63.0</td>
      <td>62.0</td>
      <td>71.0</td>
      <td>69.0</td>
      <td>78.0</td>
      <td>25.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-05</td>
      <td>279.0</td>
      <td>66.0</td>
      <td>48.0</td>
      <td>48.0</td>
      <td>60.0</td>
      <td>68.0</td>
      <td>74.0</td>
      <td>29.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-06</td>
      <td>235.0</td>
      <td>49.0</td>
      <td>44.0</td>
      <td>64.0</td>
      <td>58.0</td>
      <td>79.0</td>
      <td>24.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-07</td>
      <td>191.0</td>
      <td>40.0</td>
      <td>39.0</td>
      <td>44.0</td>
      <td>52.0</td>
      <td>22.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-08</td>
      <td>167.0</td>
      <td>42.0</td>
      <td>42.0</td>
      <td>42.0</td>
      <td>23.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-09</td>
      <td>298.0</td>
      <td>89.0</td>
      <td>97.0</td>
      <td>36.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-10</td>
      <td>352.0</td>
      <td>93.0</td>
      <td>46.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-11</td>
      <td>321.0</td>
      <td>43.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2011-12</td>
      <td>41.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# divide by the cohort size (month 0) to obtain retention as %
cohort_size = cohort_pivot.iloc[:,0]
retention_matrix = cohort_pivot.divide(cohort_size, axis = 0)
```


```python
# plot the rentention matrix
with sns.axes_style("white"):
    fig, ax = plt.subplots(1, 2, figsize=(12, 8), sharey=True, \
                           gridspec_kw={'width_ratios': [1, 11]})
    
    # retention matrix
    sns.heatmap(retention_matrix, 
                mask=retention_matrix.isnull(), 
                annot=True, 
                fmt='.0%', 
                cmap='RdYlGn', 
                ax=ax[1])
    ax[1].set_title('Monthly Cohorts: User Retention', fontsize=16)
    ax[1].set(xlabel='# of periods',
              ylabel='')

    
    # cohort size
    cohort_size_df = pd.DataFrame(cohort_size).rename(columns={0: 'cohort_size'})
    white_cmap = mcolors.ListedColormap(['white'])
    sns.heatmap(cohort_size_df, 
                annot=True, 
                cbar=False, 
                fmt='g', 
                cmap=white_cmap, 
                ax=ax[0])
    
    fig.tight_layout()
```


![](/public/image/cohort.png)

