
## Week 2 Homework

## Question 1. Start date for the Yellow taxi data

You'll need to parametrize the DAG for processing the yellow taxi data that
we created in the videos. 

What should be the start date for this dag?

* 2019-01-01
* 2020-01-01
* 2021-01-01
* days_ago(1)


>Answer:
```
2019-01-01
```

## Question 2: Frequency for the Yellow taxi data (1 point)

How often do we need to run this DAG?

* Daily
* Monthly
* Yearly
* Once

>Answer:
```
Monthly
```

## Question 3. Count records 

How many taxi trips were there on January 15?

Consider only trips that started on January 15.

> Answer
```
SELECT COUNT(1) FROM trips WHERE to_char(tpep_pickup_datetime,'DD-MM-YYYY') = '15-01-2021';
+-------+
| count |
|-------|
| 53024 |
+-------+ 
```

## Question 4. Largest tip for each day

Find the largest tip for each day. 
On which day it was the largest tip in January?

Use the pick up time for your calculations.

(note: it's not a typo, it's "tip", not "trip")

> Answer
```
SELECT
 to_char(tpep_pickup_datetime,'DD-MM-YYYY') As "Day",
 MAX(tip_amount) as "tip_max"
 FROM trips
 WHERE to_char(tpep_pickup_datetime,'MM-YYYY') = '01-2021'
 GROUP BY 1
 ORDER BY 2 DESC

 LIMIT 1;
+------------+---------+
| Day        | tip_max |
|------------+---------|
| 20-01-2021 | 1140.44 |
+------------+---------+
```

## Question 5. Most popular destination

What was the most popular destination for passengers picked up 
in central park on January 14?

Use the pick up time for your calculations.

Enter the zone name (not id). If the zone name is unknown (missing), write "Unknown" 

> Answer
```
SELECT z1."LocationID", z1."Zone", res."count" from zones z1
 join
 (SELECT trips."DOLocationID", count(trips."DOLocationID")
 FROM trips, zones
 WHERE trips."PULocationID" = zones."LocationID" AND
 to_char(trips.tpep_pickup_datetime,'DD-MM-YYYY') = '14-01-2021' AND
 zones."LocationID" = (select "LocationID" from zones where "Zone" ='Central Park')
 GROUP BY 1
 ORDER BY 2 DESC
 LIMIT 1) res
 ON z1."LocationID" = res."DOLocationID";
 
+------------+-----------------------+-------+
| LocationID | Zone                  | count |
|------------+-----------------------+-------|
| 237        | Upper East Side South | 97    |
+------------+-----------------------+-------+
```

## Question 6. Most expensive locations

What's the pickup-dropoff pair with the largest 
average price for a ride (calculated based on `total_amount`)?

Enter two zone names separated by a slash

For example:

"Jamaica Bay / Clinton East"

If any of the zone names are unknown (missing), write "Unknown". For example, "Unknown / Clinton East". 

> Answer
```
select ZP."Zone" as pu_zone, ZD."Zone" as du_zone
 , concat(ZP."Zone", '/' ,coalesce(ZD."Zone",'Unknown')) as pair
 from
 (SELECT T."PULocationID", T."DOLocationID",
 AVG (T."total_amount") AS avg_amt
 FROM trips T
 GROUP BY 1, 2
 ORDER BY 3 DESC
 LIMIT 1) PD
 JOIN zones ZP on ZP."LocationID" = PD."PULocationID"
 JOIN zones ZD on ZD."LocationID" = PD."DOLocationID";
 
+---------------+---------+-----------------------+
| pu_zone       | du_zone | pair                  |
|---------------+---------+-----------------------|
| Alphabet City | <null>  | Alphabet City/Unknown |
+---------------+---------+-----------------------+
```


## Submitting the solutions

* Form for submitting: https://forms.gle/yGQrkgRdVbiFs8Vd7
* You can submit your homework multiple times. In this case, only the last submission will be used. 

Deadline: 24 January, 17:00 CET
