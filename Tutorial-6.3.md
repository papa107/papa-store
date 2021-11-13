# Interactive Tutorial 6.3 Building a Serverless Pipeline with Pub/Sub, Functions and BigQuery

## Overview
In this tutorial we are going to step through a simple process to build a serverless data pipeline that obtains data from an API/service and loads it into BigQuery, which is an analytics engine so that our data can be explored further.
The technology stack that we will use comprises of some of Google Cloud Platform’s serverless offerings, such as:

* Cloud Scheduler: This cron service will allow us to schedule the intervals of the execution of our Cloud Function, invoking the Cloud Pub/Sub trigger to import data from the API.
* Cloud Pub/Sub: Our weather topic that will trigger our Cloud Function.
* Cloud Function: Our code being subscribed to our execution topic of Pub/Sub will run and obtain data from an API/service and will load it into BigQuery, being subscribed to our execution topic of Pub/Sub.
* BigQuery: The data will be loaded here. It is our analytics engine, allowing us to make simple or complex queries and data processes.

Along with GCP, the role of API/service will be covered by the current weather API by OpenWeatherMap. 

# Step 1: Clone the repo
You have to clone the repo that contains the function code. 
```
git clone https://github.com/jovald/gcp-serverless-data-pipeline.git
cd gcp-serverless-data-pipeline
```

## Step 2: Setting up environmental variables
As you will be using the Cloud Shell terminal to run most of your commands it is a good practice to plan out your data model beforehand as that will let us preconfigure our environment with the required variable names rather than making them up as we go along. 

Step 2.1 In this step you will export some core variables:
```
PROJECT_ID: Your project ID,
REGION_ID: Your Region
ZONE_ID: Your Zone
```

Now we want to expand on this methodology and export some of the other core variables we have already decided upon:

|    Variable    |         Value        |             Description            |
|:--------------:|:--------------------:|:----------------------------------:|
|   TOPIC_NAME:  |        weather       |     This is the Pub/Sub topic      |
|    JOB_NAME:   |    getWeather-job    |   This is the Cloud Scheduler job  |
| FUNCTION_NAME: | loadDataIntoBigQuery |  This is the name of the function  |
| SCHEDULE_TIME: |    "every 1 hours"   | This is the frequency of execution |
|   BQ_DATASET:  |    weather_dataset   |    This is the BigQuery dataset    |
|    BQ_TABLE:   |   open_weather_data  |     This is the BigQuery table     |


Step 2.2 Run the following commands in Cloud Shell Terminal
```
export TOPIC_NAME= weather
export JOB_NAME= getWeather-job
export FUNCTION_NAME= loadDataIntoBigQuery
export SCHEDULE_TIME="every 1 hours"
export BQ_DATASET= weather_dataset
export BQ_TABLE= open_weather_data
```

## Step 3. Get the API key for the Open-weather service.
To do this you will need to go and register with OpenWeather and download the API Key. Once you copy and paste the API default-key from their site you can export it into the Cloud Shell environment.

OPEN_WEATHER_MAP_API_KEY: Sign up on OpenWeather, and then you will have access to copy and paste the default API key using this link https://openweathermap.org/. 
```
export OPEN_WEATHER_MAP_API_KEY="the API key"
```

## Step 4: Activate the GCP project
Activating the project helps to execute the next commands more quickly:
```
gcloud config set project $PROJECT_ID
```

## Step 5: Create the Cloud Pub/Sub topic
```
gcloud pubsub topics create $TOPIC_NAME
```

## Step 6: Create the Cloud Scheduler job
This command will create a Cloud Scheduler job, named as JOB_NAME, that is going to send a message through the Pub/Sub TOPIC_NAME every SCHEDULE_TIME (frequency).
```
gcloud scheduler jobs create pubsub $JOB_NAME --schedule="$SCHEDULE_TIME" --topic=$TOPIC_NAME --message-body="execute"
```

## Step 7: Create the BigQuery dataset and table
```
bq mk $BQ_DATASET
```

## Step 8: Create the BigQuery table

The BQ_TABLE will contain our data:
```
bq mk --table $PROJECT_ID:$BQ_DATASET.$BQ_TABLE
```

## Step 9: Deploy the function
```
gcloud functions deploy $FUNCTION_NAME --trigger-topic $TOPIC_NAME --runtime nodejs10 --set-env-vars OPEN_WEATHER_MAP_API_KEY=$OPEN_WEATHER_MAP_API_KEY,BQ_DATASET=$BQ_DATASET,BQ_TABLE=$BQ_TABLE
```

## Step 10: Go to Cloud Functions and check the service is running with no errors

Test the Function execution by **clicking on the 3 dots** and select **Run Test**, then **Test Function**.

## Step 11. Test the Scheduler is executing;

Go to Cloud Scheduler in the Console and **click on Run Now!**

## Step 12. Check BigQuery’s data table is being populated

Go to BigQuery and then **click Open**

When the Schema page opens **click on Preview** to see the data that has been loaded. Wait a few minutes then repeat this step.

**Congratulations!** you have just successfully built a serverless data pipeline.

**Clean Up!**

**Delete the instance of Cloud Scheduler**

**Delete the instance of Cloud Function**

**Delete the Topic in Cloud Pub/Sub**

**Delete the Dataset in BigQuery**

**Delete the Cloud Storage Buckets**
