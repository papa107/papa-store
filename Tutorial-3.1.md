
# Interactive Tutorial 3.1 
In this tutorial we will be **Creating a Cloud Repository**
In the Google Cloud Console, on the project selector page, select or create a Google Cloud project.
Make sure that billing is enabled for your Cloud project. 
Enable the Cloud Source Repositories API.
## Step 1. Create a repository
In a terminal window, use the gcloud source repos create command to create a Google Cloud repository named hello-world:
```
gcloud source repos create hello-world
```

## Step 2. Clone a repository
Use the gcloud source repos clone command to clone the contents of the Google Cloud repository into a local Git repository:
```
gcloud source repos clone hello-world
```
## Step 3. Create a "Hello, World!" script
In this step you will create a Python script that prints Hello, World! in a browser window.
1. Go to your hello-world repository.
Using a text editor, **create a file named main.py**, and then paste the following code:
```
#!/usr/bin/env python

import webapp2

class MainHandler(webapp2.RequestHandler):
    def get(self):
        self.response.write('Hello, World!')

app = webapp2.WSGIApplication([
    ('/', MainHandler)
], debug=True)

```
## Step 4. Create an app.yaml file
Create an app.yaml file that contains the configuration information you need to deploy your code to App Engine.
1. Go to your hello-world repository.
2. Using a text editor, **create a file named app.yaml**, and then paste the following configuration information:
```
runtime: python27
api_version: 1
threadsafe: yes

handlers:
- url: .*
  script: main.app

libraries:
- name: webapp2
  version: "2.5.2"
```
## Step 5. Push to Cloud Source Repositories
Push the files you just created into Cloud Source Repositories.
In a terminal window, go to your hello-world directory:
```
cd hello-world
```

Add the files by typing in the terminal:
```
git add .
```
Commit the files to the repository with a comment describing the history of this action:
```
git commit -m "Add Hello World app to Cloud Source Repositories"
```
Using the git push command, add the contents of the local Git repository to Cloud Source Repositories:
```
git push origin master
```
Git pushes the files from the master branch to the origin remote. Output similar to the following is displayed:
```
Counting objects: 21, done.
Delta compression using up to 6 threads.
Compressing objects: 100% (20/20), done.
Writing objects: 100% (21/21), 9.76 KiB | 0 bytes/s, done.
Total 21 (delta 5), reused 0 (delta 0)
remote: Storing objects: 100% (21/21), done.
remote: Processing commits: 100% (6/6), done.
To https://source.developers.google.com/p/example-project-1244/r/repo-name
* [new branch]      master -> master
```
## Step 6. View files in the repository
In the Google Cloud Console, **open Cloud Source Repositories**.
**Click the name** of the hello-world repository that you created.
Go to the files you pushed to the repository.
The GCP Console shows the files in the master branch at the most recent commit.
In the Files list, **click a file to view its contents**.

## Step 7. Enable the App Engine Admin API.
In this step we will deploy your app
In a terminal window, go to the directory containing the repository:
```
cd hello-world
```
Deploy the sample app:
```
gcloud app deploy app.yaml
```
Verify that your app is running:
```
gcloud app browse
```
The browser displays the message Hello, World!
## Step 8. Update your app
In a terminal window, **use a text editor to update the main.py** file by pasting the following code:
```
#!/usr/bin/env python

import webapp2

class MainHandler(webapp2.RequestHandler):
    def get(self):
        self.response.write('Goodbye, Moon!')

app = webapp2.WSGIApplication([
    ('/', MainHandler)
], debug=True)
```
Add the file so Git can commit it:
```
git add main.py
```

Commit the file with a comment describing the history of this action:
```
git commit -m "Update main.py to say Goodbye Moon"
```
Push the file to Cloud Source Repositories:
```
git push origin master
```
Now try to run it in your browser. What message does it display? 
You should still see Hello, World because despite our edit and commit we have not triggered a build and deploy. To do that continue with the next steps:
## Redeploy your app
In a terminal window, enter the following command:
```
gcloud app deploy app.yaml
```
Open your app:
```
gcloud app browse
```
The browser displays the message Goodbye, Moon!
Congratulations! You have just created and successfully fully utilized a Cloud Respository
**Clean up**
To avoid incurring charges to your Google Cloud account for the resources used in this page, follow these steps.
##Disable your app
In the Google Cloud Console, go to the App Engine Settings page.
**Click Disable application** and follow the instructions.
Disabling your app takes effect immediately. 
**Confirm that your app is disabled by visiting the URL of your app**, for example, http://[YOUR_PROJECT_ID].appspot.com/, where [YOUR_PROJECT_ID] is the name of your Google Cloud project ID. If your app is disabled, an HTTP 404 Not Found status code is returned.
**Delete the repository**
In the GCP Console, open the All repositories page for Cloud Source Repositories.
Hold the pointer over the repository you want to delete and click Settings settings.
The General settings page opens.
**Click Delete this repository**.
The Remove repository dialog opens.
Type the name of the repository you want to delete.
**Click Delete**.
