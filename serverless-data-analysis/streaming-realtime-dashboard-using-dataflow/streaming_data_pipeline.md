# A Streaming Data Pipeline for a Real-Time Dashboard with Google Cloud Dataflow

## Overview

In this lab, you own a fleet of New York City taxi cabs and are looking to monitor how well your business is doing in real-time. You will build a streaming data pipeline to capture taxi revenue, passenger count, ride status, and much more and visualize the results in a management dashboard.

## Confirm that needed APIs are Enabled

In the GCP Console, in the Navigation menu, click **APIs & services** and ensure that the following APIs are enabled:

* Google Cloud Pub/Sub
* Dataflow API

## Create a Cloud Pub/Sub Topic

Cloud Pub/Sub is an asynchronous global messaging service. By decoupling senders and receivers, it allows for secure and highly available communication between independently written applications. Cloud Pub/Sub delivers low-latency, durable messaging.

In Cloud Pub/Sub, publisher applications and subscriber applications connect with one another through the use of a shared string called a topic. A publisher application creates and sends messages to a topic. Subscriber applications create a subscription to a topic to receive messages from it.

Google maintains a few public Pub/Sub streaming data topics for labs like this one. We'll be using the NYC Taxi & Limousine Commission’s open dataset (https://data.cityofnewyork.us/) 

## Create a BigQuery dataset

BigQuery is a serverless data warehouse. Tables in BigQuery are organized into datasets. In this lab, messages published into Pub/Sub will be aggregated and stored in BigQuery.

To create a new BigQuery dataset:

### Option 1: Command Line

1. Open Cloud Shell and run the below command to create the taxirides dataset

```sh 
bq mk taxirides
```

2. Run this command to create the taxirides.realtime table (empty schema we will stream into later)

```sh
bq mk \
--time_partitioning_field timestamp \
--schema ride_id:string,point_idx:integer,latitude:float,longitude:float,\
timestamp:timestamp,meter_reading:float,meter_increment:float,ride_status:string,\
passenger_count:integer -t taxirides.realtime
```

### Option 1: Console UI

1. In the GCP Console, go to **Navigation menu> BigQuery**.

2. Once there, click on your GCP Project ID from the left-hand menu

3. Now on the right-hand side of the console underneath the query editor click **CREATE DATASET**.

4. Give the new dataset the name **taxirides**, leave all the other fields the way they are, and click **Create dataset**.

5. If you look at the left-hand resources menu, you should see your newly created dataset

6. Click on the **taxirides** dataset

7. Click **create table**

8. Name the table **realtime**

9. For schema, click **edit as text** and paste in the below

    ```javascript

    ride_id:string,
    point_idx:integer,
    latitude:float,
    longitude:float,
    timestamp:timestamp,
    meter_reading:float,
    meter_increment:float,
    ride_status:string,
    passenger_count:integer
    ```

10. Under **Partition and cluster settings**, select the **timestamp** option for the Partitioning field.

11. Click the **Create Table** button.

## Create a Cloud Storage Bucket

1. In the GCP Console, go to **Navigation menu > Storage**.
2. Click **CREATE BUCKET**.
3. For **Name**, paste in your GCP project ID.
4. For **Default storage class**, click **Multi-regional** if it is not already selected.
5. For **Location**, choose the selection closest to you.
6. Click **Create**.

## Set up a Cloud Dataflow Pipeline

Cloud Dataflow is a serverless way to carry out data analysis. In this lab, you will set up a streaming data pipeline to read sensor data from Pub/Sub, compute the maximum temperature within a time window, and write this out to BigQuery.

1. In the GCP Console, go to **Navigation menu > Dataflow**.

2. In the top menu bar, click **CREATE JOB FROM TEMPLATE**.

3. Enter **streaming-taxi-pipeline** as the Job name for your Cloud Dataflow job.

4. Under Cloud Dataflow template, select the Cloud Pub/Sub Topic to BigQuery template.

5. Under Cloud Pub/Sub input topic, enter `projects/pubsub-public-data/topics/taxirides-realtime`

6. Under BigQuery output table, enter `<myprojectid>:taxirides.realtime`

    > Note: there is a colon : between the project and dataset name and a dot . between the dataset and table name

7. Under Temporary Location, enter `gs://<mybucket>/tmp/`

8. Click the Run job button.

A new streaming job has started! You can now see a visual representation of the data pipeline.

## Analyzing the Taxi Data Using Big Query

To analyze the data as it is streaming:

1. In the GCP Console, open the Navigation menu and select BigQuery.

2. Enter the following query in the Query editor and click RUN:

```sql
    SELECT * FROM taxirides.realtime LIMIT 10
```

## Performing aggregations on the stream for reporting

1. Run the below query

```sql 
WITH streaming_data AS (

SELECT
  timestamp,
  TIMESTAMP_TRUNC(timestamp, HOUR, 'UTC') AS hour,
  TIMESTAMP_TRUNC(timestamp, MINUTE, 'UTC') AS minute,
  TIMESTAMP_TRUNC(timestamp, SECOND, 'UTC') AS second,
  ride_id,
  latitude,
  longitude,
  meter_reading,
  ride_status,
  passenger_count
FROM
  taxirides.realtime
WHERE ride_status = 'dropoff'
ORDER BY timestamp DESC
LIMIT 100000

)

# calculate aggregations on stream for reporting:
SELECT
 ROW_NUMBER() OVER() AS dashboard_sort,
 minute,
 COUNT(DISTINCT ride_id) AS total_rides,
 SUM(meter_reading) AS total_revenue,
 SUM(passenger_count) AS total_passengers
FROM streaming_data
GROUP BY minute, timestamp

```

The result shows key metrics by the minute for every taxi drop-off

## Create a Real-Time Dashboard

1. Click Explore with Data Studio

2. pecify the below settings:

    * Chart type: column chart
    * Date range dimension: dashboard_sort
    * Dimension: dashboard_sort, minute
    * Drill Down: dashboard_sort
    * Metric: SUM() total_rides, SUM() total_passengers, SUM() total_revenue
    * Sort: dashboard_sort Ascending (latest rides first)

    Note: Visualizing data at a minute-level granularity is currently not supported in Data Studio as a timestamp. This is why we created our own dashboard_sort dimension.

3. When you're happy with your dashboard, click Save to save this data source

4. Whenever anyone visits your dashboard, it will be up-to-date with the latest transactions. You can try it yourself by clicking on the Refresh button near the Save button