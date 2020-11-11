---
layout: post
title: Cleaning Ebay Car Sales Data
permlink: /data-cleaning
---

## Introduction: 
We will work with a dataset of used cars from eBay Kleinanzeigen, A classified section of German eBay website. The dataset was originally scraped and uploaded to Kaggle. The aim of this project is to clean the data and analyze the included used car listing. Part one of this project focus on the data cleaning. Part Two emphasizes on the data analysis. 


Here is the **data dictionary** provided with data is as follows: 

* **dateCrawled** - When this ad was first crawled. All field-values are taken from this date.
* **name** - Name of the car.
* **seller** - Whether the seller is private or a dealer.
* **offerType** - The type of listing
* **price** - The price on the ad to sell the car.
* **abtest** - Whether the listing is included in an A/B test.
* **vehicleType** - The vehicle Type.
* **yearOfRegistration** - The year in which the car was first registered.
* **gearbox** - The transmission type.
* **powerPS** - The power of the car in PS.
* **model** - The car model name.
* **kilometer** - How many kilometers the car has driven.
* **monthOfRegistration** - The month in which the car was first registered.
* **fuelType** - What type of fuel the car uses.
* **brand** - The brand of the car.
* **notRepairedDamage** - If the car has a damage which is not yet repaired.
* **dateCreated** - The date on which the eBay listing was created.
* **nrOfPictures** - The number of pictures in the ad.
* **postalCode** - The postal code for the location of the vehicle.
* **lastSeenOnline** - When the crawler saw this ad last online.



## Part One: Data Cleaning

### Step 1 Import Pandas and autos.csv file name


```python
# Import Pandas and the encodings is "Latin-1" 
import pandas as pd
autos= pd.read_csv("autos.csv", encoding="Latin-1")
```


```python
# Explore the "autos" dataframe. 
autos.info()
autos.head()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 50000 entries, 0 to 49999
    Data columns (total 20 columns):
    dateCrawled            50000 non-null object
    name                   50000 non-null object
    seller                 50000 non-null object
    offerType              50000 non-null object
    price                  50000 non-null object
    abtest                 50000 non-null object
    vehicleType            44905 non-null object
    yearOfRegistration     50000 non-null int64
    gearbox                47320 non-null object
    powerPS                50000 non-null int64
    model                  47242 non-null object
    odometer               50000 non-null object
    monthOfRegistration    50000 non-null int64
    fuelType               45518 non-null object
    brand                  50000 non-null object
    notRepairedDamage      40171 non-null object
    dateCreated            50000 non-null object
    nrOfPictures           50000 non-null int64
    postalCode             50000 non-null int64
    lastSeen               50000 non-null object
    dtypes: int64(5), object(15)
    memory usage: 7.6+ MB





<div style="overflow-x:auto;">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th></th>
      <th>dateCrawled</th>
      <th>name</th>
      <th>seller</th>
      <th>offerType</th>
      <th>price</th>
      <th>abtest</th>
      <th>vehicleType</th>
      <th>yearOfRegistration</th>
      <th>gearbox</th>
      <th>powerPS</th>
      <th>model</th>
      <th>odometer</th>
      <th>monthOfRegistration</th>
      <th>fuelType</th>
      <th>brand</th>
      <th>notRepairedDamage</th>
      <th>dateCreated</th>
      <th>nrOfPictures</th>
      <th>postalCode</th>
      <th>lastSeen</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-03-26 17:47:46</td>
      <td>Peugeot_807_160_NAVTECH_ON_BOARD</td>
      <td>privat</td>
      <td>Angebot</td>
      <td>$5,000</td>
      <td>control</td>
      <td>bus</td>
      <td>2004</td>
      <td>manuell</td>
      <td>158</td>
      <td>andere</td>
      <td>150,000km</td>
      <td>3</td>
      <td>lpg</td>
      <td>peugeot</td>
      <td>nein</td>
      <td>2016-03-26 00:00:00</td>
      <td>0</td>
      <td>79588</td>
      <td>2016-04-06 06:45:54</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-04-04 13:38:56</td>
      <td>BMW_740i_4_4_Liter_HAMANN_UMBAU_Mega_Optik</td>
      <td>privat</td>
      <td>Angebot</td>
      <td>$8,500</td>
      <td>control</td>
      <td>limousine</td>
      <td>1997</td>
      <td>automatik</td>
      <td>286</td>
      <td>7er</td>
      <td>150,000km</td>
      <td>6</td>
      <td>benzin</td>
      <td>bmw</td>
      <td>nein</td>
      <td>2016-04-04 00:00:00</td>
      <td>0</td>
      <td>71034</td>
      <td>2016-04-06 14:45:08</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-03-26 18:57:24</td>
      <td>Volkswagen_Golf_1.6_United</td>
      <td>privat</td>
      <td>Angebot</td>
      <td>$8,990</td>
      <td>test</td>
      <td>limousine</td>
      <td>2009</td>
      <td>manuell</td>
      <td>102</td>
      <td>golf</td>
      <td>70,000km</td>
      <td>7</td>
      <td>benzin</td>
      <td>volkswagen</td>
      <td>nein</td>
      <td>2016-03-26 00:00:00</td>
      <td>0</td>
      <td>35394</td>
      <td>2016-04-06 20:15:37</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016-03-12 16:58:10</td>
      <td>Smart_smart_fortwo_coupe_softouch/F1/Klima/Pan...</td>
      <td>privat</td>
      <td>Angebot</td>
      <td>$4,350</td>
      <td>control</td>
      <td>kleinwagen</td>
      <td>2007</td>
      <td>automatik</td>
      <td>71</td>
      <td>fortwo</td>
      <td>70,000km</td>
      <td>6</td>
      <td>benzin</td>
      <td>smart</td>
      <td>nein</td>
      <td>2016-03-12 00:00:00</td>
      <td>0</td>
      <td>33729</td>
      <td>2016-03-15 03:16:28</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2016-04-01 14:38:50</td>
      <td>Ford_Focus_1_6_Benzin_TÜV_neu_ist_sehr_gepfleg...</td>
      <td>privat</td>
      <td>Angebot</td>
      <td>$1,350</td>
      <td>test</td>
      <td>kombi</td>
      <td>2003</td>
      <td>manuell</td>
      <td>0</td>
      <td>focus</td>
      <td>150,000km</td>
      <td>7</td>
      <td>benzin</td>
      <td>ford</td>
      <td>nein</td>
      <td>2016-04-01 00:00:00</td>
      <td>0</td>
      <td>39218</td>
      <td>2016-04-01 14:38:50</td>
    </tr>
  </tbody>
</table>
</div>



### Step 2 Cleaning Column Names

From the work we did in the last screen, we can make the following observations:
* The dataset contains 20 columns, most of which are string. 
* Some columns have null value.
* The column names use camelcase instead of Python's preferred snakecase, which means we can not just replace spaces with underscores. 

Let's convert the comn names from camelcase to snakecase and reword some of the column names based on the data dictionary to be more descriptive. 


```python
# Use autos.columns attribute to print out exiting colunn names. 
autos.columns
```




    Index(['dateCrawled', 'name', 'seller', 'offerType', 'price', 'abtest',
           'vehicleType', 'yearOfRegistration', 'gearbox', 'powerPS', 'model',
           'odometer', 'monthOfRegistration', 'fuelType', 'brand',
           'notRepairedDamage', 'dateCreated', 'nrOfPictures', 'postalCode',
           'lastSeen'],
          dtype='object')




```python
# Rename the column from camelcase to snakecase. 
autos.columns = ["date_crawled",
                 "name","seller",
                 "offer_type",
                 "price","abtest",
                 "vehicle_type",
                 "registration_year",
                 "gearbox",
                 "power_ps",
                 "model",
                 "odometer",
                 "registration_month",
                 "fuel_type","brand",
                 "unrepaired_damage",
                 "ad_created",
                 "nr_of_picture",
                 "postal_code",
                 "last_seen"]
```


```python
autos.head(1)
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
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th></th>
      <th>date_crawled</th>
      <th>name</th>
      <th>seller</th>
      <th>offer_type</th>
      <th>price</th>
      <th>abtest</th>
      <th>vehicle_type</th>
      <th>registration_year</th>
      <th>gearbox</th>
      <th>power_ps</th>
      <th>model</th>
      <th>odometer</th>
      <th>registration_month</th>
      <th>fuel_type</th>
      <th>brand</th>
      <th>unrepaired_damage</th>
      <th>ad_created</th>
      <th>nr_of_picture</th>
      <th>postal_code</th>
      <th>last_seen</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-03-26 17:47:46</td>
      <td>Peugeot_807_160_NAVTECH_ON_BOARD</td>
      <td>privat</td>
      <td>Angebot</td>
      <td>$5,000</td>
      <td>control</td>
      <td>bus</td>
      <td>2004</td>
      <td>manuell</td>
      <td>158</td>
      <td>andere</td>
      <td>150,000km</td>
      <td>3</td>
      <td>lpg</td>
      <td>peugeot</td>
      <td>nein</td>
      <td>2016-03-26 00:00:00</td>
      <td>0</td>
      <td>79588</td>
      <td>2016-04-06 06:45:54</td>
    </tr>
  </tbody>
</table>
</div>



### Step 3  Initial Exploration and cleaning

Initially we will look for: 
*  Text columns where all or almost all values are the same. These can often be dropped as they don't have useful information for analysis.
*  Numeric data stored as text which can be cleaned and converted. 


```python
autos.describe(include='all')
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
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th></th>
      <th>date_crawled</th>
      <th>name</th>
      <th>seller</th>
      <th>offer_type</th>
      <th>price</th>
      <th>abtest</th>
      <th>vehicle_type</th>
      <th>registration_year</th>
      <th>gearbox</th>
      <th>power_ps</th>
      <th>model</th>
      <th>odometer</th>
      <th>registration_month</th>
      <th>fuel_type</th>
      <th>brand</th>
      <th>unrepaired_damage</th>
      <th>ad_created</th>
      <th>nr_of_picture</th>
      <th>postal_code</th>
      <th>last_seen</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>50000</td>
      <td>50000</td>
      <td>50000</td>
      <td>50000</td>
      <td>50000</td>
      <td>50000</td>
      <td>44905</td>
      <td>50000.000000</td>
      <td>47320</td>
      <td>50000.000000</td>
      <td>47242</td>
      <td>50000</td>
      <td>50000.000000</td>
      <td>45518</td>
      <td>50000</td>
      <td>40171</td>
      <td>50000</td>
      <td>50000.0</td>
      <td>50000.000000</td>
      <td>50000</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>48213</td>
      <td>38754</td>
      <td>2</td>
      <td>2</td>
      <td>2357</td>
      <td>2</td>
      <td>8</td>
      <td>NaN</td>
      <td>2</td>
      <td>NaN</td>
      <td>245</td>
      <td>13</td>
      <td>NaN</td>
      <td>7</td>
      <td>40</td>
      <td>2</td>
      <td>76</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>39481</td>
    </tr>
    <tr>
      <th>top</th>
      <td>2016-03-22 09:51:06</td>
      <td>Ford_Fiesta</td>
      <td>privat</td>
      <td>Angebot</td>
      <td>$0</td>
      <td>test</td>
      <td>limousine</td>
      <td>NaN</td>
      <td>manuell</td>
      <td>NaN</td>
      <td>golf</td>
      <td>150,000km</td>
      <td>NaN</td>
      <td>benzin</td>
      <td>volkswagen</td>
      <td>nein</td>
      <td>2016-04-03 00:00:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-04-07 06:17:27</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>3</td>
      <td>78</td>
      <td>49999</td>
      <td>49999</td>
      <td>1421</td>
      <td>25756</td>
      <td>12859</td>
      <td>NaN</td>
      <td>36993</td>
      <td>NaN</td>
      <td>4024</td>
      <td>32424</td>
      <td>NaN</td>
      <td>30107</td>
      <td>10687</td>
      <td>35232</td>
      <td>1946</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>8</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2005.073280</td>
      <td>NaN</td>
      <td>116.355920</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5.723360</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>50813.627300</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>std</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>105.712813</td>
      <td>NaN</td>
      <td>209.216627</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.711984</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>25779.747957</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>min</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1000.000000</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>1067.000000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1999.000000</td>
      <td>NaN</td>
      <td>70.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>30451.000000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2003.000000</td>
      <td>NaN</td>
      <td>105.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>49577.000000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2008.000000</td>
      <td>NaN</td>
      <td>150.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>71540.000000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>max</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9999.000000</td>
      <td>NaN</td>
      <td>17700.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>12.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>99998.000000</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



#### Findings:
###### Text Columns can be dropped:
Columns with seller, offer_type and nr_of_pictures can be dropped. almost all the value are the same.
###### Numeric data stored as text should be converted: 
Price, Odometer numeric data has been stored as object. We need to convert the text into numeric type.


```python
autos['price'].unique()
```




    array(['$5,000', '$8,500', '$8,990', ..., '$385', '$22,200', '$16,995'],
          dtype=object)




```python
autos['odometer'].unique()
```




    array(['150,000km', '70,000km', '50,000km', '80,000km', '10,000km',
           '30,000km', '125,000km', '90,000km', '20,000km', '60,000km',
           '5,000km', '100,000km', '40,000km'], dtype=object)




```python
# Converting column price from object to numeric type

autos['price']= (autos['price']
                .str.replace('$','')
                .str.replace(',','')
                .astype(float)
                 )
```


```python
# Converting column odometer from object to numeric type

autos['odometer'] = (autos['odometer']
                   .str.replace('km','')
                   .str.replace(',','')
                    .astype(float)
                    )
```


```python
# rename "odometer" column to "odometer_km"
autos=autos.rename(columns={'odometer':'odometer_km'})
```

### Step 4 Exploring the "odometer" and "price" columns

Analyze the **"price"** and **"odometer_km"** columns using minimum and maximum value and look for any values that look unrealistically high or low that we want to remove. 


```python
# to find out how many unique value
autos['price'].unique().shape
```




    (2357,)




```python
# to view min/max/median/mean 
autos['price'].describe()
```




    count    5.000000e+04
    mean     9.840044e+03
    std      4.811044e+05
    min      0.000000e+00
    25%      1.100000e+03
    50%      2.950000e+03
    75%      7.200000e+03
    max      1.000000e+08
    Name: price, dtype: float64




```python
# to identify the outlier. 
autos['price'].value_counts().head()
```




    0.0       1421
    500.0      781
    1500.0     734
    2500.0     643
    1200.0     639
    Name: price, dtype: int64




```python
# Pick the most expensive price from the price columns. 
autos['price'].value_counts().sort_index(ascending=False).head(15)
```




    99999999.0    1
    27322222.0    1
    12345678.0    3
    11111111.0    2
    10000000.0    1
    3890000.0     1
    1300000.0     1
    1234566.0     1
    999999.0      2
    999990.0      1
    350000.0      1
    345000.0      1
    299000.0      1
    295000.0      1
    265000.0      1
    Name: price, dtype: int64




```python
# Selecting the price range between $1 to $ 35,000 and using describe() method to analyze the distrubtion of price column. 
autos= autos[autos['price'].between(1,350000)]
autos['price'].describe()
```




    count     48565.000000
    mean       5888.935591
    std        9059.854754
    min           1.000000
    25%        1200.000000
    50%        3000.000000
    75%        7490.000000
    max      350000.000000
    Name: price, dtype: float64



Conclusion: We only select the price range for used car from 1 dolloar to 35,000 dollars. 


```python
#Explore the odometer data
autos['odometer_km'].unique().shape
```




    (13,)




```python
# Descending order of odometer_km column.
autos['odometer_km'].value_counts().sort_index(ascending=False)
```




    150000.0    31414
    125000.0     5057
    100000.0     2115
    90000.0      1734
    80000.0      1415
    70000.0      1217
    60000.0      1155
    50000.0      1012
    40000.0       815
    30000.0       780
    20000.0       762
    10000.0       253
    5000.0        836
    Name: odometer_km, dtype: int64




```python
# Discover the distrubtion of odometer_km columns. 
autos['odometer_km'].describe()
```




    count     48565.000000
    mean     125770.101925
    std       39788.636804
    min        5000.000000
    25%      125000.000000
    50%      150000.000000
    75%      150000.000000
    max      150000.000000
    Name: odometer_km, dtype: float64



It is reasonable to assume more than 70% of the used car which have 150,000 milage in its odometer. 

### Step 5: Exploring the "date" columns


```python
# Select the header of three columns. top 5 rows. 
autos[['date_crawled','ad_created','last_seen']][0:5]
```




<div>
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
      <th>date_crawled</th>
      <th>ad_created</th>
      <th>last_seen</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-03-26 17:47:46</td>
      <td>2016-03-26 00:00:00</td>
      <td>2016-04-06 06:45:54</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-04-04 13:38:56</td>
      <td>2016-04-04 00:00:00</td>
      <td>2016-04-06 14:45:08</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-03-26 18:57:24</td>
      <td>2016-03-26 00:00:00</td>
      <td>2016-04-06 20:15:37</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016-03-12 16:58:10</td>
      <td>2016-03-12 00:00:00</td>
      <td>2016-03-15 03:16:28</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2016-04-01 14:38:50</td>
      <td>2016-04-01 00:00:00</td>
      <td>2016-04-01 14:38:50</td>
    </tr>
  </tbody>
</table>
</div>




```python
# We will notice that the first 10 characters represent the day (e.g.2016-03-26). 
# Therefore, picking the  first 10 characters using str[:10]
(autos['date_crawled']
 .str[:10]
 .value_counts(normalize=True,dropna=False)
 .sort_index()
)
```




    2016-03-05    0.025327
    2016-03-06    0.014043
    2016-03-07    0.036014
    2016-03-08    0.033296
    2016-03-09    0.033090
    2016-03-10    0.032184
    2016-03-11    0.032575
    2016-03-12    0.036920
    2016-03-13    0.015670
    2016-03-14    0.036549
    2016-03-15    0.034284
    2016-03-16    0.029610
    2016-03-17    0.031628
    2016-03-18    0.012911
    2016-03-19    0.034778
    2016-03-20    0.037887
    2016-03-21    0.037373
    2016-03-22    0.032987
    2016-03-23    0.032225
    2016-03-24    0.029342
    2016-03-25    0.031607
    2016-03-26    0.032204
    2016-03-27    0.031092
    2016-03-28    0.034860
    2016-03-29    0.034099
    2016-03-30    0.033687
    2016-03-31    0.031834
    2016-04-01    0.033687
    2016-04-02    0.035478
    2016-04-03    0.038608
    2016-04-04    0.036487
    2016-04-05    0.013096
    2016-04-06    0.003171
    2016-04-07    0.001400
    Name: date_crawled, dtype: float64




```python
(autos['date_crawled']
 .str[:7]
 .value_counts(normalize=True,dropna=False)
 .sort_index()
)
```




    2016-03    0.838073
    2016-04    0.161927
    Name: date_crawled, dtype: float64



**Findings:** From the date_crawled summary reasult, we can see (84%) majority of date was crawled in March, 2016. 


```python
# Checking the 'ad_creacted' column
(autos['ad_created']
 .str[:10]
 .value_counts(normalize=True,dropna=False)
 .sort_index()
)
```




    2015-06-11    0.000021
    2015-08-10    0.000021
    2015-09-09    0.000021
    2015-11-10    0.000021
    2015-12-05    0.000021
    2015-12-30    0.000021
    2016-01-03    0.000021
    2016-01-07    0.000021
    2016-01-10    0.000041
    2016-01-13    0.000021
    2016-01-14    0.000021
    2016-01-16    0.000021
    2016-01-22    0.000021
    2016-01-27    0.000062
    2016-01-29    0.000021
    2016-02-01    0.000021
    2016-02-02    0.000041
    2016-02-05    0.000041
    2016-02-07    0.000021
    2016-02-08    0.000021
    2016-02-09    0.000021
    2016-02-11    0.000021
    2016-02-12    0.000041
    2016-02-14    0.000041
    2016-02-16    0.000021
    2016-02-17    0.000021
    2016-02-18    0.000041
    2016-02-19    0.000062
    2016-02-20    0.000041
    2016-02-21    0.000062
                    ...   
    2016-03-09    0.033151
    2016-03-10    0.031895
    2016-03-11    0.032904
    2016-03-12    0.036755
    2016-03-13    0.017008
    2016-03-14    0.035190
    2016-03-15    0.034016
    2016-03-16    0.030125
    2016-03-17    0.031278
    2016-03-18    0.013590
    2016-03-19    0.033687
    2016-03-20    0.037949
    2016-03-21    0.037579
    2016-03-22    0.032801
    2016-03-23    0.032060
    2016-03-24    0.029280
    2016-03-25    0.031751
    2016-03-26    0.032266
    2016-03-27    0.030989
    2016-03-28    0.034984
    2016-03-29    0.034037
    2016-03-30    0.033501
    2016-03-31    0.031875
    2016-04-01    0.033687
    2016-04-02    0.035149
    2016-04-03    0.038855
    2016-04-04    0.036858
    2016-04-05    0.011819
    2016-04-06    0.003253
    2016-04-07    0.001256
    Name: ad_created, Length: 76, dtype: float64



**Findings:** "ad_created" data distribution is abnomally even and data could be wrong. 


```python
(autos['last_seen']
 .str[:10]
 .value_counts(normalize=True,dropna=False)
 .sort_index()
)
```




    2016-03-05    0.001071
    2016-03-06    0.004324
    2016-03-07    0.005395
    2016-03-08    0.007413
    2016-03-09    0.009595
    2016-03-10    0.010666
    2016-03-11    0.012375
    2016-03-12    0.023783
    2016-03-13    0.008895
    2016-03-14    0.012602
    2016-03-15    0.015876
    2016-03-16    0.016452
    2016-03-17    0.028086
    2016-03-18    0.007351
    2016-03-19    0.015834
    2016-03-20    0.020653
    2016-03-21    0.020632
    2016-03-22    0.021373
    2016-03-23    0.018532
    2016-03-24    0.019767
    2016-03-25    0.019211
    2016-03-26    0.016802
    2016-03-27    0.015649
    2016-03-28    0.020859
    2016-03-29    0.022341
    2016-03-30    0.024771
    2016-03-31    0.023783
    2016-04-01    0.022794
    2016-04-02    0.024915
    2016-04-03    0.025203
    2016-04-04    0.024483
    2016-04-05    0.124761
    2016-04-06    0.221806
    2016-04-07    0.131947
    Name: last_seen, dtype: float64



### Step 6 Dealing with Incorrect Registration Year Data


```python
# Use describe() method to understand the distribution of “registration_year" column. 
autos['registration_year'].describe()
```




    count    48565.000000
    mean      2004.755421
    std         88.643887
    min       1000.000000
    25%       1999.000000
    50%       2004.000000
    75%       2008.000000
    max       9999.000000
    Name: registration_year, dtype: float64



One thing that stands out from the exploration. The registration_year column contains some odd values: 
* The minimum value is 1000, before cars were invented. 
* The maximum value is 9999, many years into the future. 

Because the ca can not first registered after the listing was seen, ebay data was cralwed in around 2016. Any vehicle with a registration year above 2016 is definately inaccuate.  Determing the earlist valid year is more difficult. Realistically, it could be somewhere in the first few decade of the 1900s Let us count the number of listing with cars that fall within the 1900-2016 interval. 



```python
#Selecting the rows with registration_year fall between 1900 to 2016. 
#Pick the top 10 unique value from the registration columns.
autos=autos[autos['registration_year'].between(1990,2016)]
autos['registration_year'].value_counts(normalize=True).head(10)
```




    2000    0.069540
    2005    0.064692
    1999    0.063833
    2004    0.059558
    2003    0.059470
    2006    0.058831
    2001    0.058082
    2002    0.054777
    1998    0.052067
    2007    0.050172
    Name: registration_year, dtype: float64



## Part Two:  Oppotunities for Purchasing Used Cars 

### Step 1 Using Aggreation to Exploring Price by Brand

One of the analysis technique we learned in this course is aggreation. When working with data on cars, it is natual to explore variations across different car brands.  We can use aggreation to understand the brand column.  

Here is the procedures: 
1. Identify the unique values we want to aggreate by
2. Create an empty dictionary to store our aggreate data. 
3. Loop over the unique value, and for each: 
    - Subset the dataframe by the unique values. 
    - Caculate the mean of whichever column we're interested in
    - Assign the value/mean to the dictionary as key/value



```python
# Step 1

# Step 1.1 Selecting the top 20 common brand 
brand_count=autos["brand"].value_counts(normalize=True).head(20)
print(brand_count)
```

    volkswagen       0.210911
    bmw              0.111185
    opel             0.108805
    mercedes_benz    0.093711
    audi             0.087983
    ford             0.069981
    renault          0.048145
    peugeot          0.030672
    fiat             0.025648
    seat             0.018773
    skoda            0.016768
    nissan           0.015512
    mazda            0.015512
    smart            0.014565
    citroen          0.014036
    toyota           0.012846
    hyundai          0.010312
    volvo            0.008946
    mini             0.008924
    mitsubishi       0.008417
    Name: brand, dtype: float64



```python
# Step 1.2 
# Select those that have over a certain percentage of total value （e.g. > 5%)
# Using index() attribute to access the labels. 
common_brands = brand_count[brand_count > .05 ].index
print(common_brands)
```

    Index(['volkswagen', 'bmw', 'opel', 'mercedes_benz', 'audi', 'ford'], dtype='object')



```python
# Step 2. Create an empty dictionary to store our aggreate data. 
brand_mean_prices = {}

# Step 3. Loop over the unique value
for bran in common_brands:
    
    # Step 3.1 Subset the dataframe by unique values. 
    bran_comp = autos[autos['brand']==bran]
    
    # Step 3.2 Caculate the price column
    mean_price = bran_comp['price'].mean()
    
    # Step 3.3 , assign the value/mean to dictionary as key/value.
    brand_mean_prices[bran] = int(mean_price) 

print(brand_mean_prices)
```

    {'volkswagen': 5398, 'bmw': 8361, 'opel': 2953, 'mercedes_benz': 8582, 'audi': 9420, 'ford': 3448}


**Findings:** 
* we aggregated across brands to understand mean price. We observed that in the top 6 brands, there's a distinct price gap.
   * BMW, Audi. Benz are more expensive in the used car listing. 
   * Ford and Opel is less expensive. 
   * Volkwagen is in between. 
      * (It is suprising to see there is no Asian barnd among the common brands)


```python
# Repeat the same procedure in Step 1, Step 2, Step 3 to evaluate milage per brand
brand_mean_milage = {}

for bran in common_brands:
    bran_comp = autos[autos['brand']==bran]
    mean_milage = bran_comp['odometer_km'].mean()
    brand_mean_milage[bran] = int(mean_milage)
    
print(brand_mean_milage)    
                                
```

    {'volkswagen': 129101, 'bmw': 132665, 'opel': 129864, 'mercedes_benz': 130848, 'audi': 128974, 'ford': 125281}


**Findings:**

There is no clear distinct milage gap among the common brand. Most of them had around 13,000 km mileage in its odometer. 

### Step 2 Store the value into a single dataframe (with a shared index) 

**It is hard to compare more than two aggreate series objects. we better create a single table which combine two dictionaries in Step 1.**


```python
# Use the series constructor to create mean_price series from "brand_mean_prices" dictionary. 
mean_price = pd.Series(brand_mean_prices)
mean_milage = pd.Series(brand_mean_milage)
```


```python
# use DataFrame constructor to combine the data from two series into a single dataframe.
Brand_Price_Milage = pd.DataFrame(mean_price, columns=['mean_price'])
Brand_Price_Milage['mean_milage'] = mean_milage
print(Brand_Price_Milage)
```

                   mean_price  mean_milage
    volkswagen           5398       129101
    bmw                  8361       132665
    opel                 2953       129864
    mercedes_benz        8582       130848
    audi                 9420       128974
    ford                 3448       125281


## Conclusion: 

After comparing the price and milage across common brand, there is no visual link with milage and used car price. But we do find there is a distinct price difference among brands. In another words, the most expensive car will have the lowest milage on it. The most exciting finding from this project is figuring out Opel is very competitve in the used car market. Opel has the cheapest price with relatively good mileage on it. 