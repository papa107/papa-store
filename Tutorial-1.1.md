# Tutorial 1.1 - Creating an HTTP Trigger using Cloud Functions
In this tutorial you will learn how to create an HTTP trigger and then run it in 
Google Cloud Platform's serverless Cloud Functions platform.

## Step 1

### Create an HTTP function

Learn how to:

1. Enable Cloud Functions and Cloud Build in a project.

2. Create and deploy a simple "Hello World" function that is triggered by an HTTP request.

3. Test the function.

4. View the logs for a history of the function's actions.

To enable Cloud Functions and Cloud Build you need to have a project
Select a project, or create a new one.

<walkthrough-project-setup></walkthrough-project-setup>

Now enable the Cloud Build and Cloud Functions API by entering at the CLI:
```
gcloud services enable cloudbuild.googleapis.com
gcloud services enable cloudfunction.googleapis.com
```
When you deploy your code, Cloud Build builds the code into a container image
 and Cloud Functions accesses this image when it needs to execute your function.

Now that you are all set up let's go learn how to create a function 
that is triggered by an HTTP request, 

**Click Next**

## Step 2 Create and deploy a function

In the Cloud Console navigation menu, click Cloud Functions.

You can see where it is by clicking the following button:

<walkthrough-spotlight-pointer spotlightId="console-nav-menu">Find Function tab here</walkthrough-spotlight-pointer> 

### On the Functions page, click Create function.

If your project already contains functions, click Create function instead.

In Function name, enter the following name:

```
functions-hello-world
```

In HTTP Trigger type. **select HTTP.**

**Select Allow unauthenticated invocations**

**Click Save** and then **click Next**.

**In Runtime,** select your preferred language.

**In Source code, select Inline editor** and keep the default function 
that is provided in the inline editor, 
which appears next to the Source code list.

**Click Deploy.**

Your new function is listed on the Functions page. When the deployment is complete, a check mark appears next to the function.

To learn how to test the deployed function, **click Next.**

## Step 3 Test the function

On the Functions page, look in the Name column for the cloud-source-repositories-test function you created.

In the function's Actions column, **click More Actions and select Test function.**

On the Function details page, click the **Testing Function** tab if it isn't already selected.

**Click Test the Function.**

When the function finishes, the Output field displays 
```
"Hello World!", 
```
and the Logs field displays a status code of 200, indicating success.

To learn how to see the function's actions in the log history, **click Next.**

Review the function's logs
On the Function details page, click the back button to return to the Functions page.

On the Functions page, click the function's More Actions menu and select View logs.

The Query results section shows messages that describe the execution time and status for each time you tested the function.

🎉 Success!
You have successfully created, deployed, tested, and viewed logs for a Cloud Function!

