### Question 1: 
**What is count for fhv vehicles data for year 2019**  
Can load the data for cloud storage and run a count(*)
> Code:
```sql
SELECT count(*)
FROM `dtc-de-course-339017.trips_data_all.fhv_tripdata`
WHERE DATE(pickup_datetime) BETWEEN "2019-01-01" AND "2019-12-31";
```
>Answer:
```
42084899
```

### Question 2: 
**How many distinct dispatching_base_num we have in fhv for 2019**  
Can run a distinct query on the table from question 1
> Code:
```sql
SELECT count(DISTINCT(dispatching_base_num))
FROM `dtc-de-course-339017.trips_data_all.fhv_tripdata`
WHERE DATE(pickup_datetime) BETWEEN "2019-01-01" AND "2019-12-31";
```
```
Table info
Table ID
 dtc-de-course-339017:nyc_trips.hw-2 
Table size
 6.5 KB 
Long-term storage size
 0 B 
Number of rows
 792 
Created
 14 Feb 2022, 19:34:55 UTC+5:30 
Last modified
 14 Feb 2022, 19:34:55 UTC+5:30 
Table expiry
 NEVER 
Data location
 asia-south1 
Description

```

>Answer:
```
792
```

### Question 3: 
**Best strategy to optimise if query always filter by dropoff_datetime and order by dispatching_base_num**  
Review partitioning and clustering video.   
We need to think what will be the most optimal strategy to improve query 
performance and reduce cost.
>Answer:
```
Partition by dropoff_datetime and cluster by dispatching_base_num
```

### Question 4: 
**What is the count, estimated and actual data processed for query which counts trip between 2019/01/01 and 2019/03/31 for dispatching_base_num B00987, B02060, B02279**  
Create a table with optimized clustering and partitioning, and run a 
count(*). Estimated data processed can be found in top right corner and
actual data processed can be found after the query is executed.
>Code:
```sql
CREATE OR REPLACE TABLE `dtc-de-course-339017.trips_data_all.fhv_tripdata_clustered`
PARTITION BY DATE(pickup_datetime)
CLUSTER BY dispatching_base_num AS
SELECT * FROM `dtc-de-course-339017.trips_data_all.fhv_tripdata`;

SELECT count(*)
FROM `dtc-de-course-339017.trips_data_all.fhv_tripdata_clustered`
WHERE DATE(pickup_datetime) BETWEEN "2019-01-01" AND "2019-03-31"
AND dispatching_base_num IN ('B00987','B02060','B02279');
```
>Answer:
```
Estimated data: 400.1MB
Processed data: 145.4MB
Count: 26647

Note: Processed data is only 145 (155 in the form) if we use a clustered table. On a non-clustered table it would process the full 400MB.
```

### Question 5: 
**What will be the best partitioning or clustering strategy when filtering on dispatching_base_num and SR_Flag**  
Review partitioning and clustering video. 
Clustering cannot be created on all data types.
>Code:
```sql
SELECT count(DISTINCT(SR_Flag))
FROM `dtc-de-course-339017.trips_data_all.fhv_tripdata`
WHERE DATE(pickup_datetime) BETWEEN "2019-01-01" AND "2019-03-31";
```
>Answer:
```
Partition by SR_Flag and cluster by dispatching_base_num

Partitions can only be done on timestamps, dates or integers and there's a limit of 4000 partitions. SR_Flag is an integer and using the code above we can see that there are 43 distinct SR_Flag values, so we can use it for partitioning. Clustering can be done on Strings and dispatching_base_num is a string, so we cluster by it.
```

### Question 6: 
**What improvements can be seen by partitioning and clustering for data size less than 1 GB**  
Partitioning and clustering also creates extra metadata.  
Before query execution this metadata needs to be processed.

>Answer:
```
(Multiple choice)
No improvements
Can be worse due to metadata
```

### (Not required) Question 7: 
**In which format does BigQuery save data**  
Review big query internals video.

>Answer:
```
Columnar
```
