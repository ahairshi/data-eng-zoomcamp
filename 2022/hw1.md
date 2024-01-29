## Week 1 Homework

In this homework we'll prepare the environment 
and practice with terraform and SQL

## Question 1. Google Cloud SDK

Install Google Cloud SDK. What's the version you have? 

To get the version, run `gcloud --version`

>Answer:
```
369.0.0
```
>gcloud --version:
```
(data-engineering-zoomcamp) % gcloud -v
Google Cloud SDK 369.0.0
bq 2.0.72
core 2022.01.14
gsutil 5.6
```

## Google Cloud account 

Create an account in Google Cloud and create a project.


## Question 2. Terraform 

Now install terraform and go to the terraform directory (`week_1_basics_n_setup/1_terraform_gcp/terraform`)

After that, run

* `terraform init`
* `terraform plan`
* `terraform apply` 

Apply the plan and copy the output (after running `apply`) to the form.

It should be the entire output - from the moment you typed `terraform init` to the very end.

>Output
```
(data-engineering-zoomcamp) % terraform apply -var="project=dtc-de-course-339017"

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_bigquery_dataset.dataset will be created
  + resource "google_bigquery_dataset" "dataset" {
      + creation_time              = (known after apply)
      + dataset_id                 = "trips_data_all"
      + delete_contents_on_destroy = false
      + etag                       = (known after apply)
      + id                         = (known after apply)
      + last_modified_time         = (known after apply)
      + location                   = "europe-west6"
      + project                    = "dtc-de-course-339017"
      + self_link                  = (known after apply)

      + access {
          + domain         = (known after apply)
          + group_by_email = (known after apply)
          + role           = (known after apply)
          + special_group  = (known after apply)
          + user_by_email  = (known after apply)

          + view {
              + dataset_id = (known after apply)
              + project_id = (known after apply)
              + table_id   = (known after apply)
            }
        }
    }

  # google_storage_bucket.data-lake-bucket will be created
  + resource "google_storage_bucket" "data-lake-bucket" {
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "EUROPE-WEST6"
      + name                        = "dtc_data_lake_dtc-de-course-339017"
      + project                     = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + uniform_bucket_level_access = true
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type = "Delete"
            }

          + condition {
              + age                   = 30
              + matches_storage_class = []
              + with_state            = (known after apply)
            }
        }

      + versioning {
          + enabled = true
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_bigquery_dataset.dataset: Creating...
google_storage_bucket.data-lake-bucket: Creating...
google_bigquery_dataset.dataset: Creation complete after 2s [id=projects/dtc-de-course-339017/datasets/trips_data_all]
google_storage_bucket.data-lake-bucket: Creation complete after 2s [id=dtc_data_lake_dtc-de-course-339017]
```

## Prepare Postgres 

Run Postgres and load data as shown in the videos

We'll use the yellow taxi trips from January 2021:

```bash
wget https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2021-01.csv
```

You will also need the dataset with zones:

```bash 
wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
```

Download this data and put it to Postgres

> After Loading data

```
SELECT
    table_name,
    column_name,
    data_type
 FROM
    information_schema.columns
 WHERE
    table_name = 'trips';
+------------+-----------------------+-----------------------------+
| table_name | column_name           | data_type                   |
|------------+-----------------------+-----------------------------|
| trips      | congestion_surcharge  | double precision            |
| trips      | VendorID              | bigint                      |
| trips      | tpep_pickup_datetime  | timestamp without time zone |
| trips      | tpep_dropoff_datetime | timestamp without time zone |
| trips      | passenger_count       | bigint                      |
| trips      | trip_distance         | double precision            |
| trips      | RatecodeID            | bigint                      |
| trips      | index                 | bigint                      |
| trips      | PULocationID          | bigint                      |
| trips      | DOLocationID          | bigint                      |
| trips      | payment_type          | bigint                      |
| trips      | fare_amount           | double precision            |
| trips      | extra                 | double precision            |
| trips      | mta_tax               | double precision            |
| trips      | tip_amount            | double precision            |
| trips      | tolls_amount          | double precision            |
| trips      | improvement_surcharge | double precision            |
| trips      | total_amount          | double precision            |
| trips      | store_and_fwd_flag    | text                        |
+------------+-----------------------+-----------------------------+

select COUNT(1) from trips;
+---------+
| count   |
|---------|
| 1369765 |
+---------+
```

```
SELECT
    table_name,
    column_name,
    data_type
 FROM
    information_schema.columns
 WHERE
    table_name = 'zones';
+------------+--------------+-----------+
| table_name | column_name  | data_type |
|------------+--------------+-----------|
| zones      | index        | bigint    |
| zones      | LocationID   | bigint    |
| zones      | Borough      | text      |
| zones      | Zone         | text      |
| zones      | service_zone | text      |
+------------+--------------+-----------+

select COUNT(1) from zones;
+-------+
| count |
|-------|
| 265   |
+-------+
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

