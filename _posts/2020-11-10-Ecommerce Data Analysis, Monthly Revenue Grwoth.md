---
layout: post
title: E-commerce Data Analysis Part One - Data Wrangling & Monthly Revenue 
permlink: /Montly_Revenue_Analysis
---

## Introduction: 
In this E-commerce Data Series, we will analyze sales data from a UK online retialer. In part one, we will be mainly focus on the data wrangling and then move on to caculate the monthly revenue growth rate. 

## Table Of Content: 
  * Exploratory Data Analysis 
  * Data Cleaning
  * Monthly Revenue Growth 
  

## Prepare to Take-Off


```python
# Setting up the environment
import pandas as pd
import numpy as np
import datetime as dt
import matplotlib.pyplot as plt
from operator import attrgetter
import seaborn as sns
import matplotlib.colors as mcolors
```

## Exploratory Data Analysis


```python
# Import Data
data = pd.read_csv("OnlineRetail.csv",engine="python")
data.info()
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


It's better to change $InvoiceDate$ column in to $datatime64(ns)$ types for future data analysis on monthly revenue growth. In this project, we would ignore the hours and minutes within $InvoiceDate$ column, only the day,month,year will be stored in the dataframe. 


```python
# Change InvoiceDate data type
data['InvoiceDate'] = pd.to_datetime(data['InvoiceDate'])
# Convert to InvoiceDate hours to the same mid-night time. 
data['date']=data['InvoiceDate'].dt.normalize()
# Check Data Type
data.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 541909 entries, 0 to 541908
    Data columns (total 9 columns):
    InvoiceNo      541909 non-null object
    StockCode      541909 non-null object
    Description    540455 non-null object
    Quantity       541909 non-null int64
    InvoiceDate    541909 non-null datetime64[ns]
    UnitPrice      541909 non-null float64
    CustomerID     406829 non-null float64
    Country        541909 non-null object
    date           541909 non-null datetime64[ns]
    dtypes: datetime64[ns](2), float64(2), int64(1), object(4)
    memory usage: 37.2+ MB



```python
# Summerize the null value in dataframe. 
print(data.isnull().sum())
# Missing values percentage
missing_percentage = data.isnull().sum() / data.shape[0] * 100
missing_percentage
```

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





    InvoiceNo       0.000000
    StockCode       0.000000
    Description     0.268311
    Quantity        0.000000
    InvoiceDate     0.000000
    UnitPrice       0.000000
    CustomerID     24.926694
    Country         0.000000
    date            0.000000
    dtype: float64



Around 25% of rows wihtin $CustomerID$ column is missing values, this would impose a question mark on the implementation of digital tracking system for any business. We normally would look into more details but in this case, we do not have any visibilities on them. Let's continue to explore the missing values in $Description$ and $UnitPrice$ Column. 


```python
data[data.Description.isnull()].head()
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>622</td>
      <td>536414</td>
      <td>22139</td>
      <td>NaN</td>
      <td>56</td>
      <td>2010-12-01 11:52:00</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
    </tr>
    <tr>
      <td>1970</td>
      <td>536545</td>
      <td>21134</td>
      <td>NaN</td>
      <td>1</td>
      <td>2010-12-01 14:32:00</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
    </tr>
    <tr>
      <td>1971</td>
      <td>536546</td>
      <td>22145</td>
      <td>NaN</td>
      <td>1</td>
      <td>2010-12-01 14:33:00</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
    </tr>
    <tr>
      <td>1972</td>
      <td>536547</td>
      <td>37509</td>
      <td>NaN</td>
      <td>1</td>
      <td>2010-12-01 14:33:00</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
    </tr>
    <tr>
      <td>1987</td>
      <td>536549</td>
      <td>85226A</td>
      <td>NaN</td>
      <td>1</td>
      <td>2010-12-01 14:34:00</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
    </tr>
  </tbody>
</table>
</div>




```python
# How often a record without description is also missing unit price information and customerID
data[data.Description.isnull()].CustomerID.isnull().value_counts()
```




    True    1454
    Name: CustomerID, dtype: int64




```python
data[data.Description.isnull()].UnitPrice.isnull().value_counts()
```




    False    1454
    Name: UnitPrice, dtype: int64




```python
data[data.CustomerID.isnull()].head()
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>622</td>
      <td>536414</td>
      <td>22139</td>
      <td>NaN</td>
      <td>56</td>
      <td>2010-12-01 11:52:00</td>
      <td>0.00</td>
      <td>NaN</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
    </tr>
    <tr>
      <td>1443</td>
      <td>536544</td>
      <td>21773</td>
      <td>DECORATIVE ROSE BATHROOM BOTTLE</td>
      <td>1</td>
      <td>2010-12-01 14:32:00</td>
      <td>2.51</td>
      <td>NaN</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
    </tr>
    <tr>
      <td>1444</td>
      <td>536544</td>
      <td>21774</td>
      <td>DECORATIVE CATS BATHROOM BOTTLE</td>
      <td>2</td>
      <td>2010-12-01 14:32:00</td>
      <td>2.51</td>
      <td>NaN</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
    </tr>
    <tr>
      <td>1445</td>
      <td>536544</td>
      <td>21786</td>
      <td>POLKADOT RAIN HAT</td>
      <td>4</td>
      <td>2010-12-01 14:32:00</td>
      <td>0.85</td>
      <td>NaN</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
    </tr>
    <tr>
      <td>1446</td>
      <td>536544</td>
      <td>21787</td>
      <td>RAIN PONCHO RETROSPOT</td>
      <td>2</td>
      <td>2010-12-01 14:32:00</td>
      <td>1.66</td>
      <td>NaN</td>
      <td>United Kingdom</td>
      <td>2010-12-01</td>
    </tr>
  </tbody>
</table>
</div>




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
```


```python
# Verified all the 'nan' changed into 'Nan'
data.lowercase_descriptions.dropna().apply(lambda l: np.where("nan" in l, True, False)).value_counts()
```




    False    539724
    Name: lowercase_descriptions, dtype: int64




```python
#drop all the null values and hidden-null values
data.loc[(data.CustomerID.isnull()==False)&(data.lowercase_descriptions.isnull()==False)].info()
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



```python
# data without any null values
data_clean = data.loc[(data.CustomerID.isnull()==False)&(data.lowercase_descriptions.isnull()==False)].copy()
data_clean.isnull().sum().sum()
```




    0



## Monthly Revenue Growth


```python
# Create Revenue Column
data_clean['Revenue'] = data_clean['UnitPrice'] * data_clean['Quantity']
```


```python
# Create YearMonth Column 
data_clean['YearMonth'] = data_clean['InvoiceDate'].apply(lambda date: date.year*100+date.month)
data_clean.head(n=5)
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
    </tr>
  </tbody>
</table>
</div>




```python
#monthly data using pivot table. 
df_monthly = data_clean.copy()
df_monthly = pd.pivot_table(data_clean,index=['YearMonth'],values="Revenue",aggfunc=[np.sum])
print(df_monthly)
```

                       sum
                   Revenue
    YearMonth             
    201012      554314.170
    201101      474834.980
    201102      435859.050
    201103      578790.310
    201104      425132.101
    201105      647536.030
    201106      607671.010
    201107      573614.931
    201108      615532.500
    201109      930559.562
    201110      973348.740
    201111     1130196.310
    201112      342039.000



```python
# Groupby 'YearMonth' Column 
df_monthly_more = data_clean.groupby('YearMonth').agg({'CustomerID':lambda x: x.nunique(), 'Quantity': 'sum',\
                                                       'Revenue': 'sum',}).reset_index()
print(df_monthly_more)
```

        YearMonth  CustomerID  Quantity      Revenue
    0      201012       948.0    296153   554314.170
    1      201101       783.0    269219   474834.980
    2      201102       798.0    262381   435859.050
    3      201103      1020.0    343284   578790.310
    4      201104       899.0    278050   425132.101
    5      201105      1079.0    367411   647536.030
    6      201106      1051.0    356715   607671.010
    7      201107       993.0    363027   573614.931
    8      201108       980.0    386080   615532.500
    9      201109      1302.0    536963   930559.562
    10     201110      1425.0    568919   973348.740
    11     201111      1711.0    668568  1130196.310
    12     201112       685.0    203554   342039.000



```python
# Create MonthlyGrowth Column: 
df_monthly_more['MonthlyGrowth'] = df_monthly_more['Revenue'].pct_change()
import seaborn as sns
sns.set_style("whitegrid")
sns.catplot(x="YearMonth", y="Revenue", data=df_monthly_more, kind="point", aspect=2.5, color="#95a5a6")
plt.ylabel('Revenue')
plt.xlabel('')
sns.despine(left=True, bottom=True)

df_monthly_more.style.format({"Customers": "{:.0f}", \
                         "Quantity": "{:.0f}",\
                         "Revenue": "${:.0f}",\
                        "MonthlyGrowth": "{:.2f}"})\
                    .bar(subset=["Revenue",], color='lightgreen')\
                    .bar(subset=["CustomerID"], color='#FFA07A')
```




<style  type="text/css" >
    #T_1469cf96_2ac3_11eb_8279_acde48001122row0_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 25.6%, transparent 25.6%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row0_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 26.9%, transparent 26.9%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row1_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 9.6%, transparent 9.6%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row1_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 16.8%, transparent 16.8%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row2_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 11.0%, transparent 11.0%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row2_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 11.9%, transparent 11.9%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row3_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 32.7%, transparent 32.7%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row3_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 30.0%, transparent 30.0%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row4_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 20.9%, transparent 20.9%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row4_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 10.5%, transparent 10.5%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row5_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 38.4%, transparent 38.4%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row5_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 38.8%, transparent 38.8%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row6_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 35.7%, transparent 35.7%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row6_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 33.7%, transparent 33.7%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row7_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 30.0%, transparent 30.0%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row7_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 29.4%, transparent 29.4%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row8_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 28.8%, transparent 28.8%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row8_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 34.7%, transparent 34.7%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row9_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 60.1%, transparent 60.1%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row9_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 74.7%, transparent 74.7%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row10_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 72.1%, transparent 72.1%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row10_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 80.1%, transparent 80.1%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row11_col1 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,#FFA07A 100.0%, transparent 100.0%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row11_col3 {
            width:  10em;
             height:  80%;
            background:  linear-gradient(90deg,lightgreen 100.0%, transparent 100.0%);
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row12_col1 {
            width:  10em;
             height:  80%;
        }    #T_1469cf96_2ac3_11eb_8279_acde48001122row12_col3 {
            width:  10em;
             height:  80%;
        }</style><table id="T_1469cf96_2ac3_11eb_8279_acde48001122" ><thead>    <tr>        <th class="blank level0" ></th>        <th class="col_heading level0 col0" >YearMonth</th>        <th class="col_heading level0 col1" >CustomerID</th>        <th class="col_heading level0 col2" >Quantity</th>        <th class="col_heading level0 col3" >Revenue</th>        <th class="col_heading level0 col4" >MonthlyGrowth</th>    </tr></thead><tbody>
                <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row0" class="row_heading level0 row0" >0</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row0_col0" class="data row0 col0" >201012</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row0_col1" class="data row0 col1" >948</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row0_col2" class="data row0 col2" >296153</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row0_col3" class="data row0 col3" >$554314</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row0_col4" class="data row0 col4" >nan</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row1" class="row_heading level0 row1" >1</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row1_col0" class="data row1 col0" >201101</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row1_col1" class="data row1 col1" >783</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row1_col2" class="data row1 col2" >269219</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row1_col3" class="data row1 col3" >$474835</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row1_col4" class="data row1 col4" >-0.14</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row2" class="row_heading level0 row2" >2</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row2_col0" class="data row2 col0" >201102</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row2_col1" class="data row2 col1" >798</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row2_col2" class="data row2 col2" >262381</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row2_col3" class="data row2 col3" >$435859</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row2_col4" class="data row2 col4" >-0.08</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row3" class="row_heading level0 row3" >3</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row3_col0" class="data row3 col0" >201103</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row3_col1" class="data row3 col1" >1020</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row3_col2" class="data row3 col2" >343284</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row3_col3" class="data row3 col3" >$578790</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row3_col4" class="data row3 col4" >0.33</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row4" class="row_heading level0 row4" >4</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row4_col0" class="data row4 col0" >201104</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row4_col1" class="data row4 col1" >899</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row4_col2" class="data row4 col2" >278050</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row4_col3" class="data row4 col3" >$425132</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row4_col4" class="data row4 col4" >-0.27</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row5" class="row_heading level0 row5" >5</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row5_col0" class="data row5 col0" >201105</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row5_col1" class="data row5 col1" >1079</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row5_col2" class="data row5 col2" >367411</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row5_col3" class="data row5 col3" >$647536</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row5_col4" class="data row5 col4" >0.52</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row6" class="row_heading level0 row6" >6</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row6_col0" class="data row6 col0" >201106</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row6_col1" class="data row6 col1" >1051</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row6_col2" class="data row6 col2" >356715</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row6_col3" class="data row6 col3" >$607671</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row6_col4" class="data row6 col4" >-0.06</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row7" class="row_heading level0 row7" >7</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row7_col0" class="data row7 col0" >201107</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row7_col1" class="data row7 col1" >993</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row7_col2" class="data row7 col2" >363027</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row7_col3" class="data row7 col3" >$573615</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row7_col4" class="data row7 col4" >-0.06</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row8" class="row_heading level0 row8" >8</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row8_col0" class="data row8 col0" >201108</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row8_col1" class="data row8 col1" >980</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row8_col2" class="data row8 col2" >386080</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row8_col3" class="data row8 col3" >$615533</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row8_col4" class="data row8 col4" >0.07</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row9" class="row_heading level0 row9" >9</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row9_col0" class="data row9 col0" >201109</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row9_col1" class="data row9 col1" >1302</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row9_col2" class="data row9 col2" >536963</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row9_col3" class="data row9 col3" >$930560</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row9_col4" class="data row9 col4" >0.51</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row10" class="row_heading level0 row10" >10</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row10_col0" class="data row10 col0" >201110</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row10_col1" class="data row10 col1" >1425</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row10_col2" class="data row10 col2" >568919</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row10_col3" class="data row10 col3" >$973349</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row10_col4" class="data row10 col4" >0.05</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row11" class="row_heading level0 row11" >11</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row11_col0" class="data row11 col0" >201111</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row11_col1" class="data row11 col1" >1711</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row11_col2" class="data row11 col2" >668568</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row11_col3" class="data row11 col3" >$1130196</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row11_col4" class="data row11 col4" >0.16</td>
            </tr>
            <tr>
                        <th id="T_1469cf96_2ac3_11eb_8279_acde48001122level0_row12" class="row_heading level0 row12" >12</th>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row12_col0" class="data row12 col0" >201112</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row12_col1" class="data row12 col1" >685</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row12_col2" class="data row12 col2" >203554</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row12_col3" class="data row12 col3" >$342039</td>
                        <td id="T_1469cf96_2ac3_11eb_8279_acde48001122row12_col4" class="data row12 col4" >-0.70</td>
            </tr>
    </tbody></table>




![](/public/image/output_26_1.png)



## Conclusion: 
The monthly revenue fluctuates over time and we see a sharp decline in Dec 2011. Since we do not have full month data in December 2011, the monthly revenue in Dec is much lower than previous months.  From the line graph, we also saw a drop in the spring of 2011. These fluctuation may caused by credit card debts which were accumulated in previous Christmas season. Customers might also needs to file their tax during spring.  In general, the E-commerce revenue is booming for this retailer, revenue from 2010 to 2011 is going upwards and we saw a major breakthourgh for 51% month to month growth in Sep. 

The next question we need to ask: How many customer decide to come back and purchase from this retailer? 

In the next post, we will look into the customer cohort and retention rate. 
