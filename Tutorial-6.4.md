# Interactive Tutorial 6.4 Stream Processing with Cloud Pub/Sub and Dataflow

## Overview

In this Tutorial we will set upo a data streaming pipeline using Cloud Dataflow to handle the streaming traffic.

Step 1 Set the Project Resources

In Cloud Shell, create variables for your bucket, project, and region:
```
PROJECT_ID=$(gcloud config get-value project)
BUCKET_NAME=$PROJECT_ID
TOPIC_ID=my-id
REGION=us-central1
```

Step 2 Configure a Cloud Storage Bucket

Cloud Storage bucket names must be globally unique. Your Project ID is always unique, so that is used for your bucket name in this lab.

Create a Cloud Storage bucket owned by this project:
```
gsutil mb gs://$BUCKET_NAME
```
 
Step 3. Create a Pub/Sub topic in this project:
```
gcloud pubsub topics create $TOPIC_ID
```

Step 4. Create a Cloud Scheduler job in this project. The job publishes a message to a Pub/Sub topic at one-minute intervals.
If an App Engine app does not exist for the project, this step will create one.
```
gcloud scheduler jobs create pubsub publisher-job --schedule="* * * * *" \
    --topic=$TOPIC_ID --message-body="Hello!"
```

If prompted to enable the Cloud Scheduler API, press y and enter.

If prompted to create an App Engine app, press y and select us-central for its region.

If you get an error about your project not having an App Engine app after this, run the gcloud scheduler jobs create command again.

Step 5 Start the job.
```
gcloud scheduler jobs run publisher-job
```

Step 6. Use the following commands to clone the quickstart repository and navigate to the sample code directory:
```
virtualenv env
source env/bin/activate
git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git
cd python-docs-samples/pubsub/streaming-analytics
pip install -U -r requirements.txt  # Install Apache Beam dependencies
```
Step 7.
start the pipeline, run the following command:
```
python PubSubToGCS.py \
    --project=$PROJECT_ID \
    --region=$REGION \
    --input_topic=projects/$PROJECT_ID/topics/$TOPIC_ID \
    --output_path=gs://$BUCKET_NAME/samples/output \
    --runner=DataflowRunner \
    --window_size=2 \
    --num_shards=2 \
    --temp_location=gs://$BUCKET_NAME/temp

```
 
The preceding command runs locally and launches a Dataflow job that runs in the cloud. When the command returns JOB_MESSAGE_DETAILED: Workers have started successfully, exit the local program using Ctrl+C.

Step 8.
Observe Job and Pipeline Progress
You can observe the job's progress in the Dataflow console.
Go to the Dataflow console and Open the job details view to see:
Job structure
Job logs
Stage metrics
 
You may have to wait a few minutes to see the output files in Cloud Storage. You can see the output files by navigating to the Navigation menu > Cloud Storage. Click on your bucket name and then click Samples.

Alternatively, use the command line below to check which files have been written out.
```
gsutil ls gs://${BUCKET_NAME}/samples/
```


**Congratulations!** You have just built a Dataflow and scheduler pipeline
 
**Clenup !**

Delete the Cloud Scheduler job:
```
gcloud scheduler jobs delete publisher-job
```
If prompted, Do you want to continue, press Y and enter.
Press ctrl + c in your Cloud Shell if it's still busy printing output of your Dataflow job.
In the Dataflow console, stop the job.
With your job selected from the Dataflow Console, press the Stop button. Select the Cancel bubble to cancel the pipeline without draining.
Delete the topic:
```
gcloud pubsub topics delete $TOPIC_ID
```
Delete the files created by the pipeline:
```
gsutil -m rm -rf "gs://${BUCKET_NAME}/samples/output*"
gsutil -m rm -rf "gs://${BUCKET_NAME}/temp/*"
```
Remove the Cloud Storage bucket:
```
gsutil rb gs://${BUCKET_NAME}
```
