---
layout: post
title: Analyzing Sales Performance Using SQL
permlink: /sql
---


# Analyzing Sales Performance Using SQL

## Proejct Goals: 
In this SQL project, we need to answer allowing questions: 
1. What is the most popular genre in USA? 
2. Which new albums the record store should purchase? 
3. Who is the most productive sales support agent? 
4. How much our customers spent on average per order in different countries? 
5. Shall we keep both purchase options open? Albums vs Individual Tracks

We'll use the Chinook database in this project and here is a copy of the database schema is below: 
<img src="Database Schema.png">

## Step 1.Connent to the Database


```python
conda install -yc conda-forge ipython-sql
```

    Collecting package metadata (current_repodata.json): done
    Solving environment: done
    
    # All requested packages already installed.
    
    
    Note: you may need to restart the kernel to use updated packages.



```python
%%capture
%load_ext sql
%sql sqlite:///chinook.db
```

## Step 2. Overview of the Data And Views

To run SQL queries in Jupyter Notebook, we have to add %%sql on its own line to the start of our query. So to execute the query above:


```python
%%sql
SELECT
    name,
    type
FROM sqlite_master
WHERE type IN ("table","view");
```

     * sqlite:///chinook.db
    Done.





<table>
    <tr>
        <th>name</th>
        <th>type</th>
    </tr>
    <tr>
        <td>album</td>
        <td>table</td>
    </tr>
    <tr>
        <td>artist</td>
        <td>table</td>
    </tr>
    <tr>
        <td>customer</td>
        <td>table</td>
    </tr>
    <tr>
        <td>employee</td>
        <td>table</td>
    </tr>
    <tr>
        <td>genre</td>
        <td>table</td>
    </tr>
    <tr>
        <td>invoice</td>
        <td>table</td>
    </tr>
    <tr>
        <td>invoice_line</td>
        <td>table</td>
    </tr>
    <tr>
        <td>media_type</td>
        <td>table</td>
    </tr>
    <tr>
        <td>playlist</td>
        <td>table</td>
    </tr>
    <tr>
        <td>playlist_track</td>
        <td>table</td>
    </tr>
    <tr>
        <td>track</td>
        <td>table</td>
    </tr>
</table>



## Step 3. Selecting Popular Albums and Genre to Purchase
### Q1.What's most popular genre in USA? 


```python

%%sql

WITH usa_track_sold AS
   (SELECT il.* from invoice_line il
    INNER JOIN invoice i on il.invoice_id = i.invoice_id
    INNER JOIN customer c on i.customer_id = c.customer_id
    WHERE c.country = "USA"
   )
    
SELECT g.name genre, 
       count(uts.invoice_line_id) track_sold,
       cast(count(uts.invoice_line_id) AS FLOAT)/(
         SELECT COUNT(*) from usa_track_sold) percentage_sold
from usa_track_sold uts
INNER JOIN track t on t.track_id = uts.track_id
INNER JOIN genre g on g.genre_id = t.genre_id 
GROUP BY genre
ORDER BY track_sold DESC;

```

     * sqlite:///chinook.db
    Done.





<table>
    <tr>
        <th>genre</th>
        <th>track_sold</th>
        <th>percentage_sold</th>
    </tr>
    <tr>
        <td>Rock</td>
        <td>561</td>
        <td>0.5337773549000951</td>
    </tr>
    <tr>
        <td>Alternative &amp; Punk</td>
        <td>130</td>
        <td>0.12369172216936251</td>
    </tr>
    <tr>
        <td>Metal</td>
        <td>124</td>
        <td>0.11798287345385347</td>
    </tr>
    <tr>
        <td>R&amp;B/Soul</td>
        <td>53</td>
        <td>0.05042816365366318</td>
    </tr>
    <tr>
        <td>Blues</td>
        <td>36</td>
        <td>0.03425309229305423</td>
    </tr>
    <tr>
        <td>Alternative</td>
        <td>35</td>
        <td>0.03330161750713606</td>
    </tr>
    <tr>
        <td>Pop</td>
        <td>22</td>
        <td>0.02093244529019981</td>
    </tr>
    <tr>
        <td>Latin</td>
        <td>22</td>
        <td>0.02093244529019981</td>
    </tr>
    <tr>
        <td>Hip Hop/Rap</td>
        <td>20</td>
        <td>0.019029495718363463</td>
    </tr>
    <tr>
        <td>Jazz</td>
        <td>14</td>
        <td>0.013320647002854425</td>
    </tr>
    <tr>
        <td>Easy Listening</td>
        <td>13</td>
        <td>0.012369172216936251</td>
    </tr>
    <tr>
        <td>Reggae</td>
        <td>6</td>
        <td>0.005708848715509039</td>
    </tr>
    <tr>
        <td>Electronica/Dance</td>
        <td>5</td>
        <td>0.004757373929590866</td>
    </tr>
    <tr>
        <td>Classical</td>
        <td>4</td>
        <td>0.003805899143672693</td>
    </tr>
    <tr>
        <td>Heavy Metal</td>
        <td>3</td>
        <td>0.0028544243577545195</td>
    </tr>
    <tr>
        <td>Soundtrack</td>
        <td>2</td>
        <td>0.0019029495718363464</td>
    </tr>
    <tr>
        <td>TV Shows</td>
        <td>1</td>
        <td>0.0009514747859181732</td>
    </tr>
</table>



### Q2. Which new albums the record store should purchase?

Record store has just signed a deal with a new record label, and we need to select the first three albums that will be added to the store, from a list of four. 


|Artist Name | Genre|
|------------|-------|
|Regal|Hip-Hop|
|Red Tone|Punk|
|Meteor and the Girls|Pop|
|slim Jim Bites| Blues|


Based on the sales of tracks accross different genres in the USA, we should purchase the new albums by the following artists:

* Red Tone (Punk)
* Slim Jim Bites (Blues)
* Meteor and the Girls (Pop)

These three genres only accounts 17% of total sales, so we should lookout for artists and albums for the 'rock' genre, which make up 53% of sales. 

## Step 4.  Analyzing Employee Sales Performance
### Q3. Who is the most productive sales agent? 


```python
%%sql

WITH customer_support_rep_sales AS
    (
     SELECT
         i.customer_id,
         c.support_rep_id,
         SUM(i.total) total
     FROM invoice i
     INNER JOIN customer c ON i.customer_id = c.customer_id
     GROUP BY 1,2
    )

SELECT
    e.first_name || " " || e.last_name employee,
    e.hire_date,
    SUM(csrs.total) total_sales
FROM customer_support_rep_sales csrs
INNER JOIN employee e ON e.employee_id = csrs.support_rep_id
GROUP BY 1;
```

     * sqlite:///chinook.db
    Done.





<table>
    <tr>
        <th>employee</th>
        <th>hire_date</th>
        <th>total_sales</th>
    </tr>
    <tr>
        <td>Jane Peacock</td>
        <td>2017-04-01 00:00:00</td>
        <td>1731.5099999999998</td>
    </tr>
    <tr>
        <td>Margaret Park</td>
        <td>2017-05-03 00:00:00</td>
        <td>1584.0000000000002</td>
    </tr>
    <tr>
        <td>Steve Johnson</td>
        <td>2017-10-17 00:00:00</td>
        <td>1393.92</td>
    </tr>
</table>



Jane Peacock is the most productive employee in the store, and there is a 20% difference in sales between Jane and Steve. However, there is six months difference in their hiring date. This may explain why Steve ranks at the bottom. 

## Step 5. Analyzing Sales by Country
### Q4. How much our customers spent on average per order in different countries? 

We need to analyze the sales data for customers from each different country on: 
* total number of customers
* total value of sales
* average value of sales per customer
* average order value

Since some countries with only one customer, we will group these customers as “Other.” In this project, we want to keep “Others” to the bottom of our results. Inside the subquery, we selected all the values from invoce_line (il) and customer_id and added a country column using a case statement before sorting using that new column in the outer query.


```python
%%sql

WITH country_or_other AS
    (
     SELECT
       CASE
           WHEN (
                 SELECT count(*)
                 FROM customer
                 where country = c.country
                ) = 1 THEN "Other"
           ELSE c.country
       END AS country,
       c.customer_id,
       il.*
     FROM invoice_line il
     INNER JOIN invoice i ON i.invoice_id = il.invoice_id
     INNER JOIN customer c ON c.customer_id = i.customer_id
    )
    
SELECT
        country,
        count(distinct customer_id) customers,
        SUM(unit_price) total_sales,
        SUM(unit_price) / count(distinct customer_id) customer_lifetime_value,
        SUM(unit_price) / count(distinct invoice_id) average_order,
        CASE
            WHEN country = "Other" THEN 1
            ELSE 0
        END AS sort
    FROM country_or_other
    GROUP BY country
    ORDER BY sort ASC, total_sales DESC
```

     * sqlite:///chinook.db
    Done.





<table>
    <tr>
        <th>country</th>
        <th>customers</th>
        <th>total_sales</th>
        <th>customer_lifetime_value</th>
        <th>average_order</th>
        <th>sort</th>
    </tr>
    <tr>
        <td>USA</td>
        <td>13</td>
        <td>1040.490000000008</td>
        <td>80.03769230769292</td>
        <td>7.942671755725252</td>
        <td>0</td>
    </tr>
    <tr>
        <td>Canada</td>
        <td>8</td>
        <td>535.5900000000034</td>
        <td>66.94875000000043</td>
        <td>7.047236842105309</td>
        <td>0</td>
    </tr>
    <tr>
        <td>Brazil</td>
        <td>5</td>
        <td>427.68000000000245</td>
        <td>85.53600000000048</td>
        <td>7.011147540983647</td>
        <td>0</td>
    </tr>
    <tr>
        <td>France</td>
        <td>5</td>
        <td>389.0700000000021</td>
        <td>77.81400000000042</td>
        <td>7.781400000000042</td>
        <td>0</td>
    </tr>
    <tr>
        <td>Germany</td>
        <td>4</td>
        <td>334.6200000000016</td>
        <td>83.6550000000004</td>
        <td>8.161463414634186</td>
        <td>0</td>
    </tr>
    <tr>
        <td>Czech Republic</td>
        <td>2</td>
        <td>273.24000000000103</td>
        <td>136.62000000000052</td>
        <td>9.108000000000034</td>
        <td>0</td>
    </tr>
    <tr>
        <td>United Kingdom</td>
        <td>3</td>
        <td>245.52000000000078</td>
        <td>81.84000000000026</td>
        <td>8.768571428571457</td>
        <td>0</td>
    </tr>
    <tr>
        <td>Portugal</td>
        <td>2</td>
        <td>185.13000000000022</td>
        <td>92.56500000000011</td>
        <td>6.3837931034482835</td>
        <td>0</td>
    </tr>
    <tr>
        <td>India</td>
        <td>2</td>
        <td>183.1500000000002</td>
        <td>91.5750000000001</td>
        <td>8.72142857142858</td>
        <td>0</td>
    </tr>
    <tr>
        <td>Other</td>
        <td>15</td>
        <td>1094.9400000000085</td>
        <td>72.99600000000056</td>
        <td>7.448571428571486</td>
        <td>1</td>
    </tr>
</table>



Based on the data, there maybe opportunities in the following countries: 

 * Czech Republic
 * United Kingdom
 * India

It’s worth keeping in mind that the amount of data from each of these countries is relatively small. Thus, we should be cautious about spending too much money on new marketing campaigns, as the sample size is not large enough to give us high confidence. A better approach would be to run small campaigns in these countries, collecting and analyzing new customers to make sure that these trends hold with new customers.


## Step 6. Albums vs Individual Tracks
### Q5. Shall we keep both purchase options open? 


```python
%%sql

WITH invoice_first_track AS
    (
     SELECT
         il.invoice_id invoice_id,
         MIN(il.track_id) first_track_id
     FROM invoice_line il
     GROUP BY 1
    )

SELECT
    album_purchase,
    COUNT(invoice_id) number_of_invoices,
    CAST(count(invoice_id) AS FLOAT) / (
                                         SELECT COUNT(*) FROM invoice
                                      ) percent
FROM
    (
    SELECT
        ifs.*,
        CASE
            WHEN
                 (
                  SELECT t.track_id FROM track t
                  WHERE t.album_id = (
                                      SELECT t2.album_id FROM track t2
                                      WHERE t2.track_id = ifs.first_track_id
                                     ) 

                  EXCEPT 

                  SELECT il2.track_id FROM invoice_line il2
                  WHERE il2.invoice_id = ifs.invoice_id
                 ) IS NULL
             AND
                 (
                  SELECT il2.track_id FROM invoice_line il2
                  WHERE il2.invoice_id = ifs.invoice_id

                  EXCEPT 

                  SELECT t.track_id FROM track t
                  WHERE t.album_id = (
                                      SELECT t2.album_id FROM track t2
                                      WHERE t2.track_id = ifs.first_track_id
                                     ) 
                 ) IS NULL
             THEN "yes"
             ELSE "no"
         END AS "album_purchase"
     FROM invoice_first_track ifs
    )
GROUP BY album_purchase;
```

     * sqlite:///chinook.db
    Done.





<table>
    <tr>
        <th>album_purchase</th>
        <th>number_of_invoices</th>
        <th>percent</th>
    </tr>
    <tr>
        <td>no</td>
        <td>500</td>
        <td>0.8143322475570033</td>
    </tr>
    <tr>
        <td>yes</td>
        <td>114</td>
        <td>0.18566775244299674</td>
    </tr>
</table>



Album purchases account for 18.6% of sales. Based on the above table. We should encourage customers to purchase only tracks from the album and keep both purchase options available for potential customers.

## Conclusion: 

Based on our data, Rock is the most popular genre in the United States, and we recommend record store lookout for artists and albums for the 'rock' genre.   The business should select the following new record labels based on historical sales data:  Red Tone, Slim Jim Bites & Meteor and the Girls. 

In terms of employee performance, Jane Peacock is the top-performing agent. We recommend giving her a promotion and letting her open a workshop to share her experience with other employees. Besides that, our record store might have opportunities to expand into the Czech Republic, UK and India.  However, we recommend conducting more market research before spending a large campaign budget in these countries.  

Finally, we would recommend the record store keeping both 'tracks and the album' purchase options available for our customers. 
