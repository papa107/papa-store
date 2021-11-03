Tutorial 1.3: Running a Container in Cloud Run
Introduction
This tutorial shows you how to get started with Cloud Run. 
You'll set up a Hello World service, deploy it to Cloud Run, 
and view the status of your live service's resources.

Learn how to:

Create a sample Hello World Cloud Run service.

Build/deploy your service to Cloud Run.

Edit and redeploy your service.

View and navigate through your service logs.

Clean up the resources you created for this walkthrough.

Create your web service
For this tutorial, you'll be using the Cloud Shell Editor as your environment for creating your service. The editor comes preloaded with the tools needed for Cloud development. To create your service:

Launch the Cloud Code menu from the status bar.

Select New Application.

Select the Cloud Run application for type of sample app.

From the list of sample Cloud Run services, select the Node.js: Cloud Run option.

Select a folder for your service location and click Create New Application.

Cloud Shell Editor loads your service in a new workspace. Once it reloads, your service is accessible with the explorer view.

Click Next to learn how to build and deploy this service.

Build and deploy your service to Cloud Run
Now that you've created your service, you can deploy it to Cloud Run.

If you don't already have a Google Cloud project, you'll need to create a new one and enable billing to complete this step.

To build and deploy your service:

Launch the Cloud Code menu from the status bar.

Select Deploy to Cloud Run.

If prompted, Authorize Cloud Shell to make Google Cloud API calls.

When prompted, select your Google Cloud project.

If prompted, enable your Cloud Run API by clicking Enable API.

In the Deploy to Cloud Run dialog, under Service settings, select 
an existing service or create a new one.

Choose a region to deploy to (e.g. us-east1).

Select Allow unauthenticated invocations to make this a public service.

Click Deploy.

Cloud Code now builds your image, pushes it to the container registry, 
and deploys your service to Cloud Run.

Once you see the message "Deployment completed successfully...", 
your service is live and accessible via the URL displayed 
in the Deploy to Cloud Run dialog.

Edit your service
Before editing the service, let's review what the Hello World sample service does:

The service source code, index.js, implements the service behavior, responding to
 requests with a "Hello World" greeting.

When directly invoked for local use, this code creates a basic web server that
 listens on the port defined by the PORT environment variable.

This service is containerized via a Dockerfile.

To modify and recompile your service:

Change your app.py message to "It's redeployed!". The file saves automatically.

Select Deploy to Cloud Run from the Cloud Code menu options, accessible 
via the status bar.

In the Deploy to Cloud Run dialog, under Service settings, check that 
the service you've just created is selected.

Click Deploy.

Once your service finishes building and deploying, you can launch your service
 with the link in your the Deploy to Cloud Run dialog

View your service logs
To analyze your service while it's running, you can view its logs via the
 integrated Log Viewer:

Navigate to the Cloud Run Explorer.

Right-click your Cloud Run service and choose View Logs to access the logs
 for your service.

Refresh your service in the browser.

To view the newly generated logs within the Log Viewer, click on the
 Logs refresh button.

🎉 Success
You have successfully created and deployed your first 
serverless service on Cloud Run!
 or simply refresh your open tab to see your updated service.
