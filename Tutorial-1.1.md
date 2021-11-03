
Step 1

Create an HTTP function

Learn how to:

1. Enable Cloud Functions and Cloud Build in a project.

2. Create and deploy a simple "Hello World" function that is triggered by an HTTP request.

3. Test the function.

4. View the logs for a history of the function's actions.

Enable Cloud Functions and Cloud Build in a project
Select a project, or create a new one.


My Project xxxxx

The necessary APIs have been enabled
Cloud Build API
Cloud Functions API

When you deploy your code, Cloud Build builds the code into a container image
 and Cloud Functions accesses this image when it needs to execute your function.

To learn how to create a function that is triggered by an HTTP request, 
click Next.

Step 2
Create and deploy a function
In the Cloud Console navigation menu, click Cloud Functions.

You can see where it is by clicking the following button:

 Cloud Functions

On the Functions page, click Create function. Show me

If your project already contains functions, click Create function instead.

In Function name, enter the following name:

functions-hello-world
In HTTP Trigger type. select HTTP.

Select Allow unauthenticated invocations.

Click Save, and then click Next.

In Runtime, select your preferred language.

In Source code, select Inline editor and keep the default function that is provided in the inline editor, which appears next to the Source code list.

Click Deploy.

Your new function is listed on the Functions page. When the deployment is complete, a check mark appears next to the function.

To learn how to test the deployed function, click Next.

Step 3
Test the function
On the Functions page, look in the Name column for the cloud-source-repositories-test function you created.

In the function's Actions column, click More Actions and select Test function.

On the Function details page, click the Testing tab if it isn't already selected.

Click Test the function.

When the function finishes, the Output field displays "Hello World!", and the Logs field displays a status code of 200, indicating success.

To learn how to see the function's actions in the log history, click Next.

Review the function's logs
On the Function details page, click the back button to return to the Functions page.

On the Functions page, click the function's More Actions menu and select View logs.

The Query results section shows messages that describe the execution time and status for each time you tested the function.

🎉 Success!
You have successfully created, deployed, tested, and viewed logs for a Cloud Function!
