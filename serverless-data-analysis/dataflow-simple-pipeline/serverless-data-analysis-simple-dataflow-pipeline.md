# Serverless Data Analysis with Dataflow: A Simple Dataflow Pipeline w/ Python

## Overview

In this lab, you will open a Dataflow project, use pipeline filtering, and execute the pipeline locally and on the cloud.

* Open Dataflow project

* Pipeline filtering

* Execute the pipeline locally and on the cloud

## Objective

In this lab, you learn how to write a simple Dataflow pipeline and run it both locally and on the cloud.

* Setup a Python Dataflow project using Apache Beam

* Write a simple pipeline in Python

* Execute the query on the local machine

* Execute the query on the cloud

## Task 1: Preparation

### Verify that the repository files are in Cloud Shell Editor

1. Clone the repository from the Cloud Shell command line:
    ```sh
    git clone https://github.com/GoogleCloudPlatform/training-data-analyst
    ```

2. Click on **File > Refresh** in the left navigator panel. You should see the **training-data-analyst** directory.

### Verify that you have a bucket

Create a bucket with your project_id as the bucket name, and use the following settings:

* Default storage class: Multi-Regional
* Location: <Your location>

In the cloudshell create a variable to hold the name of your bucket
    ```sh
    BUCKET="<your unique bucket name (Project ID)>"
    echo $BUCKET
    ```

### Verify that Dataflow API is enabled

## Task 2. Open Dataflow project

The goal of this lab is to become familiar with the structure of a Dataflow project and learn how to execute a Dataflow pipeline. You will need to update some files to install Apache Beam. Apache Beam is an open source platform for executing data processing workflows.

1. Return to the browser tab containing Cloud Shell. In Cloud Shell navigate to the directory for this lab:

    ```cd ~/training-data-analyst/courses/data_analysis/lab2/python```

2. Install the necessary dependencies for Python dataflow:

    ```sudo ./install_packages.sh```

3. Verify that you have the right version of pip. (It should be > 8.0):

    ```pip -V```

    If not, open a new Cloud Shell tab and it should pick up the updated version of pip.

4. Use File > Refresh in Cloud Shell editor to view the local copy of the repository.

## Task 3: Pipeline filtering
1. In the Cloud Shell code editor navigate to the directory /training-data-analyst/courses/data_analysis/lab2/python and view the file `grep.py`. Do not make any changes to the code.

## Task 4: Execute the pipeline locally

1. Run
    ```sh 
    cd ~/training-data-analyst/courses/data_analysis/lab2/python
    python grep.py
    ```

2. The output file will be output.txt. If the output is large enough, it will be sharded into separate parts with names like: output-00000-of-00001. If necessary, you can locate the correct file by examining the file's time.

    ```ls -al /tmp  ```

## Task 5: Execute the pipeline in the cloud

1. Copy some Java helpers to the cloud

    ```gsutil cp ../javahelp/src/main/java/com/google/cloud/training/dataanalyst/javahelp/*.java gs://$BUCKET/javahelp```

2. Edit the Dataflow pipeline in grepc.py. In the Cloud Shell code editor navigate to the directory /training-data-analyst/courses/data_analysis/lab2/python in and edit the file grepc.py.

3. Replace PROJECT and BUCKET with your Project ID and Bucket name. Here are easy ways to retrieve the values:
    ```sh
    echo $DEVSHELL_PROJECT_ID
    echo $BUCKET
    ```

    Example strings before:

    ```python
    PROJECT='cloud-training-demos'
    BUCKET='cloud-training-demos'
    ```

    Example strings after edit (use your values):

    ```python
    PROJECT='qwiklabs-gcp-your-value'
    BUCKET='qwiklabs-gcp-your-value'
    ```

4. Submit the Dataflow job to the cloud:

    ```
    python grepc.py
    ```
5. Examine the output in the Cloud Storage bucket. On the Navigation menu (7a91d354499ac9f1.png), click Storage > Browser and click on your bucket. Click the javahelp directory. This job will generate the file output.txt. If the file is large enough it will be sharded into multiple parts with names like: output-0000x-of-000y. You can identify the most recent file by name or by the Last modified field. Click on the file to view it.

Alternatively, you could download the file in Cloud Shell and view it:

    ```sh
    gsutil cp gs://$BUCKET/javahelp/output* .
    cat output*
    ```