# Serverless Data Analysis with BigQuery: Loading and Exporting Data

In this lab, you will load data in different formats into BigQuery tables. You will load data into BigQuery in multiple ways, transform the data that you load, and query the data.

* Load a CSV file into a BigQuery table using the web UI
* Load a JSON file into a BigQuery table using the CLI
* Export a table using the web UI

## Task 1. Upload the data using web UI

In this task, you upload a CSV file to BigQuery using the BigQuery web UI. BigQuery supports the following data formats when loading data into tables: CSV, JSON, AVRO, or Cloud Datastore backups. This example focuses on loading a CSV file into BigQuery.

1. In BigQuery console create a new dataset within your project by selecting your project in the **Resources** section, then clicking on **CREATE DATASET** on the right.

2. In the Create Dataset dialog, for Dataset ID, type `cpb101_flight_data` and click Create dataset.

3. Download the following file to your local machine. This file contains the data that will populate the first table.
    https://storage.googleapis.com/cloud-training/CPB200/BQ/lab4/airports.csv

4. Create a new table in the `cpb101_flight_data` dataset to store the data from the CSV file. Click on `cpb101_flight_data` dataset then click on Create table.

5. On the Create Table page, in the Source section:

    * For Create table from:, select upload.
    * In the Select file, click Browse, then browse to and select airports.csv.
    * Verify File format is set to CSV.

    > Note: When you have created a table previously, the Create from Previous Job option allows you to quickly use your settings to    create similar tables.

6. In the Destination section:

    * For Dataset name, leave cpb101_flight_data selected.

    * For Table name, type AIRPORTS.

    * For Table type, Native table should be selected and unchangeable.

7. In the **Schema** section:

    Add fields one at a time. The airports.csv has the following fields: `IATA`, `AIRPORT`, `CITY`, `STATE`,` COUNTRY` which are of type **STRING** and `LATITUDE`, `LONGITUDE` which are of type **FLOAT**. Make all these field mode is **REQUIRED**.

8. In the Advanced options section

    * For Field delimiter, verify Comma is selected.

    * Since airports.csv contains a single header row, for Header rows to skip, type 1.

    * Accept the remaining default values and click Create Table. BigQuery creates a load job to create the table and upload data into the table (this may take a few seconds). You can track job progress by clicking Job History.

9. Once the load job is complete, click **cpb101_flight_data > AIRPORTS**.

10. On the Table Details page, click Details to view the table properties and then click Preview to view the table data.

## Task 2. Upload the data using the CLI

In this task, you will upload multiple JSON files and an associated schema file to BigQuery using the CLI.

1. In Cloud Shell, enter the following command to download the schema file for the table to your working directory. (The file is schema_flight_performance.json(https://storage.googleapis.com/cloud-training/CPB200/BQ/lab4/schema_flight_performance.json) )

    ```sh
    curl https://storage.googleapis.com/cloud-training/CPB200/BQ/lab4/schema_flight_performance.json -o schema_flight_performance.json

    ```

    Next, you will create a table in the dataset using the schema file you downloaded to Cloud Shell and data from JSON files that are in Cloud Storage. The JSON files have URIs like the following:

    `gs://cloud-training/CPB200/BQ/lab4/domestic_2014_flights_*.json`

    > Note that your Project ID is stored as a variable in Cloud Shell `$DEVSHELL_PROJECT_ID` so there's no need for you to remember it. If you require it, you can view your Project ID in the command line to the right of your username (after the @ symbol).

2. In Cloud Shell, create a table named flights_2014 in the cpb101_flight_data dataset with this command:


    ```sh
    bq load --source_format=NEWLINE_DELIMITED_JSON $DEVSHELL_PROJECT_ID:cpb101_flight_data.flights_2014 gs://cloud-training/CPB200/BQ/lab4/domestic_2014_flights_*.json ./schema_flight_performance.json
    ```

    There are multiple JSON files in the Cloud Storage bucket. They are named according to the convention: domestic_2014_flights*.json_. The wildcard (*) character in the command is used to include all of the .json files in the bucket.

3. Once the table is created, type the following command to verify table flights_2014 exists in dataset cpb101_flight_data.

    ```sh
    bq ls $DEVSHELL_PROJECT_ID:cpb101_flight_data
    ```

## Task 3: Export table

1. Create a bucket to receive the exported data

2. Select the **AIRPORTS** table that you recently created, and using the button to its right most corner, select the option **Export > Export to GCS**. 

3. In Select GCS location, specify `gs://<YOUR-BUCKET>/bq/airports.csv` and click **Export**.

Use the CLI to export the table:

```
bq extract cpb101_flight_data.AIRPORTS gs://$BUCKET/bq/airports2.csv
```

Click Check my progress to verify the objective.