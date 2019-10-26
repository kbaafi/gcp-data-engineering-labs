# Serverless MapReduce using Google Dataflow (Python)

## Overview

In this lab, you will identify Map and Reduce operations, execute the pipeline, use command line parameters.

## Objective
    * Identify Map and Reduce operations

    * Execute the pipeline

    * Use command line parameters

## Task 1. Review Preparations

These preparations should already be have been done:

    * Create Cloud Storage bucket

    * Clone github repository to Cloud Shell (https://github.com/GoogleCloudPlatform/training-data-analyst)

```sh 
    git clone https://github.com/GoogleCloudPlatform/training-data-analyst

```

    * Upgrade packages and install Apache Beam

```sh
    sudo apt-get install python-pip -y

    pip install apache-beam[gcp] oauth2client==3.0.0
    # downgrade as 1.11 breaks apitools
    sudo pip install --force six==1.10
    pip install -U pip
    pip -V
    sudo pip install apache_beam
```

## Task 2. Identify Map and Reduce operations

In the Cloud Shell code editor navigate to the directory `/training-data-analyst/courses/data_analysis/lab2/python` and view the file is_popular.py in the Cloud Shell editor. Do not make any changes to the code.
Alternatively, you could view the file with nano. **Do not make any changes to the code.**

```sh
    cd ~/training-data-analyst/courses/data_analysis/lab2/python
    nano is_popular.py
```

>Normally, you would develop this Java code in an Integrated Development Environment such as Eclipse or Intellij (not in CloudShell).

Can you answer these questions about the file is_popular.py?

* What custom arguments are defined?

* What is the default output prefix?

* How is the variable output_prefix in `main()` set?

* How are the pipeline arguments such as `--runner set`?

* What are the key steps in the pipeline?

* Which of these steps happen in parallel?

* Which of these steps are aggregations?

## Task 3. Execute the pipeline

1. Run the pipeline locally:

```sh
    cd ~/training-data-analyst/courses/data_analysis/lab2/python
    python ./is_popular.py
```
>Note: if you see an error that says "No handlers could be found for logger "oauth2client.contrib.multistore_file", you may ignore it. The error is simply saying that logging from the oauth2 library will go to stderr.

2. Identify the output file. It should be output<suffix> and could be a sharded file.

```sh
    ls -al /tmp
```

3. Examine the output file, replacing '-*' with the appropriate suffix.

```sh
    cat /tmp/output-*
```

## Task 4. Use command line parameters

1. Change the output prefix from the default value:

    ```python ./is_popular.py --output_prefix=/tmp/myoutput```

2. What will be the name of the new file that is written out?

3. Note that we now have a new file in the /tmp directory:

    ```ls -lrt /tmp/myoutput*```