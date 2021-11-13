# Interactive Tutorial 6.2 - Working with Cloud Tasks

## Overview:
In this tutorial you will learn how to create tasks and a task queue.

Before you begin you need to prepare your environment:
Create or select a GCP project:
Enable the Cloud Tasks API.
Set up authentication to the API.
Follow the instructions to create a service account. Download the file with the JSON key associated with the account and store it locally. A service account allows you to authenticate with Google Cloud programmatically.
The samples use the Google Cloud Client Libraries to interact with App Engine, so you need to set an environment variable to point to the key you downloaded above, which the library uses to authenticate your requests. It is possible to create your requests without using the client libraries, but they can help you manage details of low level communication with the server, including authentication.

## Step 1. Download and install the code sample:
The Node.js sample consists of two files, one (createTask.js) run locally as a command-line tool to create and add tasks to the queue and one (server.js) deployed on App Engine as a worker to "process" the task. The queue itself runs on App Engine.

To download and install the sample:
Clone the sample application repository:
```
git clone https://github.com/googleapis/nodejs-tasks.git
```

Step 1.1 Navigate to the directory that contains the sample code:
```
cd nodejs-tasks/samples
```
Step 1.3 Install all dependencies.
You can use npm:
```
npm install
```
Step 1.4 Deploy the worker service (server.js) to App Engine standard environment:
```
gcloud app deploy app.yaml
```
Steo 1.5 Check to make sure the app containing the service is running:
```
gcloud app browse
```
Your browser opens https://{YOUR_PROJECT_ID}.appspot.com/ and displays Hello, World!.

## Step 2 Create a Cloud Tasks Queue
In this tutorial you will use the Cloud Shell with its inbuilt Cloud SDK gcloud queue management function to create your queue in the environment you prepared above.
Step 2.1 At the command line, enter the following:
```
gcloud tasks queues create my-queue
```
Wait a bit for the queue to initialize and then proceed to the next step. 

Step 2.2 Check the Task queue is built successfully
```
gcloud tasks queues describe my-queue
```

## Step 3. Add a task to the Cloud Tasks queue
Step 3.1 Create a task locally, add it to the queue you set up, and deliver that task to an asynchronous worker:
Set the following environment variables on your machine either manually, in the code in your sample app, or via an alias. The client uses this information to create the request:
```
export PROJECT_ID=PROJECT_ID # The project ID from above
export LOCATION_ID=LOCATION_ID # The region in which your queue is running
```
Note: You can find the location ID by using the following gcloud command:  gcloud tasks locations list
export QUEUE_ID=my-queue # The queue you created above

Step 3.2 Create a task with a payload of hello and add that task to your queue. The payload can be any data from the request that the worker needs to complete processing the task:
```
node createTask.js $PROJECT_ID $QUEUE_ID $LOCATION_ID hello
```

Verify that the payload was received by displaying the logs of the worker service:
```
gcloud app browse
```
**Congratulations!** you have successfully completed the Tutorial
