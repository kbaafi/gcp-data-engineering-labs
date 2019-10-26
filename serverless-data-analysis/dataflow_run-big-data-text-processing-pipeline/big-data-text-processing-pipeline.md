# Run a Big Data Text Processing Pipeline in Cloud Dataflow

## Enable the APIs

A number of APIs must be enabled for this lab to work. This section will show you how to check and if necessary, enable an API.

Enable the following APIs

    * Compute Engine API

    * Dataflow API

    * Stackdriver Logging API

    * Google Cloud Storage

    * Google Cloud Storage JSON API

    * BigQuery API

    * Cloud Pub/Sub API

    * Cloud Datastore API

After this, create a GCS bucket

## Create a Maven Project

Create a Maven project that contains the Cloud Dataflow SDK for Java.

```sh

    mvn archetype:generate \
        -DarchetypeGroupId=org.apache.beam \
        -DarchetypeArtifactId=beam-sdks-java-maven-archetypes-examples \
        -DarchetypeVersion=2.8.0 \
        -DgroupId=org.example \
        -DartifactId=first-dataflow \
        -Dversion="0.1" \
        -Dpackage=org.apache.beam.examples \
        -DinteractiveMode=false

```

Now you should see a new directory called `first-dataflow` under your current directory. `first-dataflow` contains a Maven project that includes the Cloud Dataflow SDK for Java and example pipelines

## Run a text processing pipeline on Cloud Dataflow

1. Save your project ID and Cloud Storage bucket names as environment variables

    * `export PROJECT_ID=<your_project_id>`

    * or more explicitly `export PROJECT_ID=$$DEVSHELL_PROJECT_ID`

    * or `export PROJECT_ID=$(gcloud config get-value project)`

    * Then `export BUCKET_NAME=<your_bucket_name>`

2. We're going to run a pipeline called **WordCount**, which reads text, tokenizes the text lines into individual words, and performs a frequency count on each of those words. While the pipeline is running, we'll take a look at what's happening in each step.

    Start the pipeline by running mvn **-Pdataflow-runner** compile exec:java. For the `--project`, `--stagingLocation`, and `--output` arguments, the command below references the environment variables you just set up.

```sh
 mvn -Pdataflow-runner compile exec:java \
      -Dexec.mainClass=org.apache.beam.examples.WordCount \
      -Dexec.args="--project=${PROJECT_ID} \
      --stagingLocation=gs://${BUCKET_NAME}/staging/ \
      --output=gs://${BUCKET_NAME}/output \
      --runner=DataflowRunner"
```

Click the **Navigation menu > Dataflow** to open the **Dataflow** page.

You should see your **wordcount** job with a status of **Running**:

You can check out the whole WordCount.java file on [Github](https://github.com/GoogleCloudPlatform/DataflowSDK-examples/blob/1af22edf182b86a6d1331f7589966841d6e09f82/java/examples-java8/src/main/java/com/google/cloud/dataflow/examples/WordCount.java)

