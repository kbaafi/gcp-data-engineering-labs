# Creating a Data Transformation Pipeline with Cloud Dataprep

## Overview

Cloud Dataprep by Trifacta is an intelligent data service for visually exploring, cleaning, and preparing structured and unstructured data for analysis. In this lab you explore the Cloud Dataprep UI to build a data transformation pipeline that runs at a scheduled interval and outputs results into BigQuery.

The dataset you'll use is an ecommerce dataset that has millions of Google Analytics session records for the Google Merchandise Store loaded into BigQuery. You have a copy of that dataset for this lab and will explore the available fields and row for insights.

## Objectives

In this lab, you learn how to perform these tasks:

* Connect BigQuery datasets to Cloud Dataprep.
* Explore dataset quality with Cloud Dataprep.
* Create a data transformation pipeline with Cloud Dataprep.
* Schedule transformation jobs outputs to BigQuery.

Although this lab is largely focused on Cloud Dataprep, you need BigQuery as an endpoint for dataset ingestion to the pipeline and as a destination for the output when the pipeline is completed.

## 1. Create the dataset

* Create a dataset in BigQuery

* Run the following SQL Query

    ```sql
    #standardSQL
    CREATE OR REPLACE TABLE ecommerce.all_sessions_raw_dataprep
    OPTIONS(
    description="Raw data from analyst team to ingest into Cloud Dataprep"
    ) AS
    SELECT * FROM `next-marketing-analytics.ecommerce.all_sessions_raw`
    WHERE date = '20170801';
    ```

## 2. Connect to BigQuery from Cloud Dataprep

1. Open Dataprep and allow **Trifacta** access to your project

2. Create a Flow, specifying the following details

    * For Flow Name, type Ecommerce Analytics Pipeline
    * For Flow Description, type Revenue reporting table

3. Import your data by clicking **Import & Add Datasets** and select BigQuery as your source

4. Select your **ecommerce** dataset

5. Click on the **Create dataset** icon (+ sign) on the left of the `all_sessions_raw_dataprep` table.

6. Click **Import & Add to Flow** in the bottom right corner.

## 3. Exploring the fields with the UI

1. Add a **New Recipe** and click **Edit Recipe**

2. In the UI peruse the fields and try to understand the data. Try to answer the following questions:

    * How many rows does the sample contain?

    * What is the most common value of in the `channelGrouping` column?

    * What are the top three countries from which sesseions are originated?

    * What is the maximum `timeOnSite` in seconds, maximum `pageviews`, and maximum `sessionQualityDim` for the data sample?

    * Looking at the histogram for `sessionQualityDim`, are the data values evenly distributed?

    * You might see a red bar under the `productSKU` column. If so, what might that mean?

    * Looking at the `v2ProductName` column, what are the most popular products?

    * Looking at the `v2ProductCategory` column, what are some of the most popular product categories?

    * True or False? The most common `productVariant` is `COLOR`

    * What are the two values in the `type` column?

    * What is the maximum `productQuantity`?

    * What is the dominant `currencyCode` for transactions?

    * Are there valid values for `itemQuantity` or `itemRevenue`?

    * What percentage of `transactionId` values are valid? What does this represent for our `ecommerce` dataset?

    * How many `eCommerceAction_type` values are there, and what is the most common value?

    * Using the **schema**, what does `eCommerceAction_type` = `6` represent?

## 4. Cleaning the data

In this task, you will clean the data by deleting unused columns, eliminating duplicates, creating calculated fields, and filtering out unwanted rows.

* Change the **productSKU** type to string

* Delete **itemQuantity** and **itemRevenue** fields as they only contain NULL values are not useful for the purpose of this lab

* Remove duplicate rows: Click the **Filter rows** icon in the toolbar, then click **Remove duplicate rows**. Add this step to the recipe.

* Filter out sessions without revenue:

    1. Under the **totalTransactionRevenue** column, click the grey **Missing values** bar. All rows with a missing value for **totalTransactionRevenue** are now highlighted in red.

    2. In the **Suggestions** panel, in **Delete rows** , click **Add**. This step filters your dataset to only include transactions with revenue (where **totalTransactionRevenue** is not NULL)

* Filtering sessions for PAGE views:

    The dataset contains sessions of different types, for example **PAGE** (for page views) or **EVENT** (for triggered events like "viewed product categories" or "added to cart"). To avoid double counting session pageviews, add a filter to only include page view related hits.

    1. In the histogram below the `type` column, click the bar for **PAGE**. All rows with the type **PAGE** are now highlighted in green.

    2. In the **Suggestions** panel, in **Keep rows**, and click **Add**.

## 5. Enriching the data

Search your schema documentation for visitId and read the description to determine if it is unique across all user sessions or just the user.

>`visitId`: an identifier for this session. This is part of the value usually stored as the `utmb` cookie. This is only unique to the user. For a completely unique ID, you should use a combination of `fullVisitorId` and visitId.

As we see, `visitId` is not unique across all users. We will need to create a unique identifier.

1. Create a new column for a unique session ID

    1. Click on the **Merge columns** icon in the toolbar.

    2. For Columns, select **fullVisitorId** and **visitId**.

    3. For **Separator** type a single hyphen character: -

    4. For the **New column name**, type **unique_session_id**.

    5. Add this to the recipe

2. Creating a case statement for the ecommerce action type

    As you saw earlier, values in the eCommerceAction_type column are integers that map to actual ecommerce actions performed in that session

    1. Click on the Conditions icon in the toolbar, then click Case on single column.

    2. For Column to evaluate, specify eCommerceAction_type.

    3. Next to Cases (1), click Add 8 times for a total of 9 cases.

    4. For each Case, specify the following mapping values (including the single quote characters):

        >   0: 'Unknown'
        >
        >   1: 'Click through of product lists'
        >
        >    2: 'Product detail views'
        >
        >    3: 'Add product(s) to cart'
        >
        >    4: 'Remove product(s) from cart'
        >
        >    5: 'Check out'
        >
        >    6: 'Completed purchase'
        >
        >    7: 'Refund of purchase'
        >
        >    8: 'Checkout options'

    5. For **New column name**, type **eCommerceAction_label**. Leave the other fields at their default values.

    6. Click **Add**

3. The **totalTransactionRevenue** column contains values passed to Analytics multiplied by 10^6 (e.g., 2.40 would be given as 2400000). You now divide contents of that column by 10^6 to get the original values.

    1. Open the menu to the right of the **totalTransactionRevenue** column  then select **Calculate > Custom formula**.

    2. For Formula, type: **DIVIDE(totalTransactionRevenue,1000000)** and for New column name, type: **totalTransactionRevenue1**.

    3. Click **Add**

    4. To convert the new **totalTransactionRevenue1** column's type to a **decimal** data type


## Running and scheduling Cloud Dataprep jobs to BigQuery

> **Challenge**: Now that you are satisfied with the flow, it's time to execute the transformation recipe against your source dataset. The challenge for you is to load the output of the job into the BigQuery dataset that you created earlier. Make sure you load the output into a separate table and name it revenue_reporting.

You can also schedule the execution of pipeline in the next step so the job can be re-run automatically on a regular basis to account for newer data. Note: You can navigate and perform other operations while jobs are running.

1. You will now schedule a recurrent job execution. Click the Flows icon on the left of the screen.

2. On the right of your **Ecommerce Analytics Pipeline** flow click the **More** icon, then click **Schedule Flow**.

3. In the **Add Schedule** dialog:

4. For **Frequency**, select **Weekly**.

5. For day of week, select **Saturday** and unselect **Sunday**.

6. For time, enter **3:00** and select AM.

7. Click **Save**.

8. Click the **Jobs** icon on the left of the screen.