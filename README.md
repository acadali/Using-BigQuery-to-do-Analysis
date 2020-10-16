# Using-BigQuery-to-do-Analysis

In this lab, we analyze the two different datasets, run queries separately , and willcombine the tables to see the resultant data.

The purpose of accessing the public dataset is that you don&#39;t have to download the data on your machine to work. You can run ad-hoc queries and gets the response.

# Lets Start!

First log in to GCP platform, and from the navigation side menu, select BigQuery and pinned it too.
<p></p>
![Test Image 4]( https://github.com/acadali/Using-BigQuery-to-do-Analysis/blob/main/1.png
)

<p></p>
After clicking BigQuery, click &quot;Add Data&quot; and explore public data.
<p></p>

![Test Image 4]( https://github.com/acadali/Using-BigQuery-to-do-Analysis/blob/main/2.png
)


After selecting any public data, click &quot;View Dataset&quot;. In this case I have selected &#39;nyc bike&#39;.


![Test Image 4]( https://github.com/acadali/Using-BigQuery-to-do-Analysis/blob/main/3.png
)


This dataset will be loaded to your console. You can preview the data and also the schema of that dataset as well.


![Test Image 4]( https://github.com/acadali/Using-BigQuery-to-do-Analysis/blob/main/4.png
)


Paste the following in the Query editor textbox.

    SELECT 
    MIN(start_station_name) AS start_station_name, 
    MIN(end_station_name) AS end_station_name,  
    APPROX_QUANTILES(tripduration, 10)[OFFSET (5)] AS typical_duration, 
    COUNT(tripduration) AS num_trips
    FROM `bigquery-public-data.new_york_citibike.citibike_trips`
    WHERE start_station_id != end_station_id 
    GROUP BY start_station_id,  end_station_id 
    ORDER BY num_trips DESC
    LIMIT 10


Click Run. This will return the typical duration for the 10 most common rentals.


![Test Image 4]( https://github.com/acadali/Using-BigQuery-to-do-Analysis/blob/main/5.png
)


Next, now paste the following query, it will show the total distance travelled by each bicycle only top 5.

    WITH
    trip_distance AS (
    SELECT
     bikeid,
     ST_Distance(ST_GeogPoint(s.longitude,
      s.latitude),
    ST_GeogPoint(e.longitude,
      e.latitude)) AS distance
    FROM
    `bigquery-public-data.new_york_citibike.citibike_trips`,
    `bigquery-public-data.new_york_citibike.citibike_stations`  as s,
    `bigquery-public-data.new_york_citibike.citibike_stations` as e
    `WHERE
        start_station_id = s.station_id
     AND end_station_id = e.station_id )
    SELECT
    bikeid,
     SUM(distance)/1000 AS total_distance
    FROM
     trip_distance
    GROUP BY
    bikeid
    ORDER BY
     total_distance DESC
    LIMIT
     5`


![Test Image 4]( https://github.com/acadali/Using-BigQuery-to-do-Analysis/blob/main/6.png
)


# **Explore the weather dataset**

In the left pane of the BigQuery Console, Select added bigquery-public-data project and select ghcn\_d\&gt;ghcnd\_2015. Then click the preview tab to see whats in the data and also see the schema.

Examine the columns and some of the data values.

Click COMPOSE NEW QUERY and enter the following:

    SELECT
    wx.date,
    wx.value/10.0 AS prcp
    FROM
    `bigquery-public-data.ghcn_d.ghcnd_2015` AS wx
    WHERE
    id =  'USW00094728';
    AND qflag IS NULL
    AND element = 'PRCP'
    ORDER BY
    wx.date


This will return rainfall (in mm) for all days in 2015.


![Test Image 4]( https://github.com/acadali/Using-BigQuery-to-do-Analysis/blob/main/7.png
)


Find Relation between bicycle and rain datasets.


We can join these datasets to identify the bicycle rental on rainy days is few or not.


Click  **COMPOSE NEW QUERY**  and run the following query :

    WITH bicycle_rentals AS (
    SELECT
    COUNT(starttime) as num_trips,
    EXTRACT(DATE from starttime) as trip_date
    FROM `bigquery-public-data.new_york_citibike.citibike_trips`
    GROUP BY trip_date
    ),
    rainy_days AS
    (
    SELECT
     date,
    (MAX(prcp) > 5) AS rainy
    FROM (
    SELECT
    wx.date AS date,
    IF (wx.element = 'PRCP', wx.value/10, NULL) AS prcp
     FROM
    `bigquery-public-data.ghcn_d.ghcnd_2015` AS wx
     WHERE
    wx.id = 'USW00094728'
    )
    GROUP BY
      date
    )
    SELECT
      ROUND(AVG(bk.num_trips)) AS num_trips,
     wx.rainy
    FROM bicycle_rentals AS bk
    JOIN rainy_days AS wx	
    ON wx.date = bk.trip_date
    GROUP BY wx.rainy	


You can see the results of joining these two datasets.

Let me try to explain the query. This query has 3 parts. In the first part of the query, we are getting the bicycle rental data according to the dates.

In the second part, we are getting the data of the rainy day according to the dates.

In the last part, we are joining the first two results on the basis of dates to identify that the bicycles was rented more on rainy days or not.

In the result, num of trips shows the answer.


![Test Image 4]( https://github.com/acadali/Using-BigQuery-to-do-Analysis/blob/main/8.png
)

