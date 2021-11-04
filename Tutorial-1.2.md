# Tutorial 1.2: Running an App in Google App Engine
In this interactive tutorial we will learn how to deploy and run an app on App Engine. 

## Step 1. Project setup
GCP organises resources into projects, which collect all of the related resources for a single application in one place.
Begin by creating a new project or selecting an existing project for this tutorial.
<walkthrough-project-setup></walkthrough-project-setup>
Once you select your project_ID proceed and Click Start.

## Step 2. Using Cloud Shell

Cloud Shell is a built-in command-line tool for the console. We're going to use Cloud Shell to deploy our app.

Open Cloud Shell by **clicking the Activate Cloud Shell** button in the navigation bar in the upper-right corner of the console.

Clone the sample code
Use Cloud Shell to clone and navigate to the 'Hello World' code. The sample code is cloned from your project repository to the Cloud Shell.

Note: If the directory already exists, remove the previous files before cloning.
```sh
git clone  https://github.com/GoogleCloudPlatform/nodejs-docs-samples
```

Then switch to the tutorial directory:
```sh
cd  nodejs-docs-samples/appengine/hello-world/standard
```

## Step 3. Configuring your deployment
As you are now in the directory you just created to hold the cloned sample code we'll take the opportunity to look at the files that configure your application.

Enter the following command to view your application code:
```
cat app.js
```

Exploring your configuration file

App Engine uses YAML files to specify a deployment's configuration. The app.yaml files contain information about your application, like the runtime environment, URL handlers and a lot more.
<walkthrough-editor-open-file
    filePath="/home/alasdair_gilchrist/nodejs-docs-samples/appengine/hello-world/standard/app.yaml">
    open app.yaml for editing
</walkthrough-editor-open-file>


## Step 4. Testing your app

**Test your app on Cloud Shell**
Cloud Shell lets you test your app before deploying to make sure that it's running as intended, just like debugging on your local machine.

To test your app, enter the following:
```
export PORT=8080 && npm install
```

Followed by:
```
npm start
```

**Preview your app with 'Web preview'**

Your app is now running on Cloud Shell. You can access the app by clicking the Web preview  button at the top of the Cloud Shell pane and choosing Preview on port 8080.

**Terminating the preview instance**
Terminate the instance of the application by pressing Ctrl+C in Cloud Shell.

## Step 5. Deploying to App Engine

**Create an application**
To deploy your app, you need to create an app in a region:
```
gcloud app create
```

Note: If you get an error you may need to set your project_ID for your current workspace by running this command:
```
gcloud config set project Your Project ID
```

For example:
 gcloud config set project tidal-advantage-287623

**Deploying with Cloud Shell**
You can use Cloud Shell to deploy your app. To deploy your app, enter the following:
```
gcloud app deploy
```

**Visit your app**
Click on the url that is returned in the output. The default URL of your app is a subdomain on appspot.com that starts with your project's ID: 

**Congratulations!** Your app has been deployed. 
