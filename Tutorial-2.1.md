# Interactive Tutorial 2.1: 
Developing with Cloud Code using the Cloud Shell Editor
In this interactive turoial you will Create and deploy a Cloud Run serverless service
By doing so you will learn how to:
* Create a simple web service.
* Build the service and deploy it to Cloud Run.
* View logs to make sure the service is running without issues.
* Clean up to avoid billing charges.
## Step 1 -Create a web service
These steps show you how to work in Cloud Shell Editor. You can use a similar process if you use the Cloud Shell extension in your IDE.
### Open Cloud Shell Editor
Click Cloud Code in the Cloud Shell Editor status bar.
### Select New Application.
Select Cloud Run application as the sample type.
From the list of sample Cloud Run services, **select the Go: Cloud Run option.**
**Select a folder** for your service location and **click Create New Application**.
Cloud Shell Editor loads your service in a new workspace. Once it reloads, your service is accessible with the explorer view.
The service consists of:
**main.go**, which responds to requests with "It's running!"
When directly invoked for local use, this code creates a basic web server that listens on the port defined by the PORT environment variable.
**go.mod**, which declares Go dependencies.
**Dockerfile**, which Cloud Run uses to build an executable container image.
## Step 2. Build and deploy the service to Cloud Run
**Click Cloud Code** located in the lower status bar.
### Select Deploy to Cloud Run.
If prompted, authorize Cloud Shell to make Google Cloud API calls.
In the Deploy to Cloud Run tab, click Select GCP Project and select a project.
If you don't already have a Google Cloud project, you'll need to create a new one and enable billing.
### Click Enable API to enable the Cloud Run API.
After you enable the API, additional settings appear in the tab.
Under Service settings, select an existing service or do the following to create a new one:
In the Service list, **Select + Create a service**.
For the Deployment platform, **choose Cloud Run (fully managed)**.
For Authentication, **choose Allow unauthenticated invocations** to make this a public service.
Step 3. Use Cloud Build.
### Click Deploy.
Cloud Code builds an image, pushes it to the container registry, and deploys your service to Cloud Run.
When you see the message "Deployment completed successfully! URL: your URL", visit the service by **clicking the URL at the end of the message**.
### View your service logs
Navigate to the Cloud Run Explorer.
Right-click your Cloud Run service and choose View Logs.
In the Logs Viewer tab, click the refresh button that appears above the logs display.
The latest logs from your service appear in the logs display.
To view more logs, re-visit your service's URL, then in the Logs Viewer tab, click the refresh button again.
🎉 **Congratulations!**
You have successfully created and deployed a service on Cloud Run!
