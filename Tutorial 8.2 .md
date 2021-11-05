# Interactive Tutorial 8.2 
 ## Working with Cloud Tasks
In this tutorial you will learn how to create tasks and a task queue.
Before you begin you need to prepare your environment:
Create or select a GCP project:
Enable the Cloud Tasks API.
Set up authentication to the API.
Follow the instructions to create a service account. Download the file with the JSON key associated with the account and store it locally.
A service account allows you to authenticate with Google Cloud programmatically.

Step 1.

1.0 Download and install the code sample: 
```
git clone https://github.com/googleapis/nodejs-tasks.git
```
1.1 Navigate to the directory that contains the sample code:
```
cd nodejs-tasks/samples
```
1.2 Install all dependencies.
You can use npm:
```
npm install
```
1.3 Deploy the worker service (server.js) to App Engine standard environment:
```
gcloud app deploy app.yaml
```
1.4 Check to make sure the app containing the service is running:
```
gcloud app browse
```
Your browser opens https://{YOUR_PROJECT_ID}.appspot.com/ and displays Hello, World!.

## Step 2.

2.0 Create a Cloud Tasks Queue

In this tutorial you will use the Cloud Shell with its inbuilt Cloud SDK gcloud queue management function to create your queue in the environment you prepared above.

2.1 At the command line, enter the following:
```
gcloud tasks queues create my-queue
```
2.2 Wait a bit for the queue to initialize and then proceed to the next step. 

## Step 3. 

3.0 Check the Task queue is built successfully
```
gcloud tasks queues describe my-queue
```
## Step 4. 

4.0 Add a task to the Cloud Tasks queue

4.1 Create a task locally, add it to the queue you set up, and deliver that task to an asynchronous worker to do this Set the following environment variables on your machine either manually, in the code in your sample app, or via an alias. The client uses this information to create the request:

Note: You can find the location ID by using the following gcloud command:  gcloud tasks locations list
```
export PROJECT_ID=PROJECT_ID # The project ID from above
export LOCATION_ID=LOCATION_ID # The region in which your queue is running
export QUEUE_ID=my-queue # The queue you created above
```
4.2 Create a task with a payload of hello and add that task to your queue. The payload can be any data from the request that the worker needs to complete processing the task:
```
node createTask.js $PROJECT_ID $QUEUE_ID $LOCATION_ID hello
```
Verify that the payload was received by displaying the logs of the worker service:
```
gcloud app browse
```

**Congratulations!** You have successfully completed the Tutorial

## Clean up
To avoid incurring charges to your Google Cloud account for the resources used in this page, follow these steps.

Caution: Deleting a project has the following effects:

Everything in the project is deleted. If you used an existing project for this tutorial, when you delete it, you also delete any other work you've done in the project.

In the Cloud Console, go to the Manage resources page. 

In the project list, **select the project** that you want to delete, and then **click Delete**.

In the dialog, **type the project ID,** and then **click Shut down to delete the project**.
