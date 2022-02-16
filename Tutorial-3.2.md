# Interactive Tutorial 3.2 
You will first build the image using a Dockerfile, which is the Docker configuration file, and then build the same image using the Cloud Build configuration file.
Note: This quickstart shows you how to build an image with docker, but Cloud Build supports most build tools and programming languages.
Enable the Cloud Build and the Artifact Registry APIs.
If you've already installed Cloud SDK previously, make sure you have the latest available version by running 
```
gcloud components update
```

## Step 1. Prepare Your source files
In this tutorial you will need some sample source code to package into a container image. In this section, you'll create a simple shell script and a Dockerfile. A Dockerfile is a text document that contains instructions for Docker to build an image.
1. **Open a cloud terminal** window.
2. **Create a new directory** named tutorial-docker and then navigate into it:
```
mkdir tutorial-docker
cd tutorial-docker
```
3. **Create a file** named tutorial.sh with the following contents:
```
echo "Hello, world! The time is $(date)."
```
4. **Create a file** named Dockerfile with the following contents:
```
FROM alpine
COPY tutorial.sh /
CMD ["/tutorial.sh"]
```
5. Run the following command to make tutorial.sh executable:
```
chmod +x tutorial.sh
```
Step 2. Create a Docker repository in Artifact Registry
1. **Create a new Docker repository** named tutorial-docker-repo in the location us-central1 with the description "Docker repository" is using windows remove the '\' in the command:
```
gcloud artifacts repositories create tutorial-docker-repo --repository-format=docker \  --location=us-central1 --description="Docker repository"
```
2. Verify that your repository was created:
```
gcloud artifacts repositories list
```
Step 3. Build using Dockerfile
Cloud Build allows you to build a Docker image using a Dockerfile. You don't require a separate Cloud Build config file.
To build using a Dockerfile:
Get your Cloud project ID by running the following command:
```
gcloud config get-value project
```
2. Run the following command from the directory containing tutorial.sh and Dockerfile, where project-id is your Cloud project ID:
```
gcloud builds submit --tag us-central1-docker.pkg.dev/project-id/quickstart-docker-repo/quickstart-image:tag1
```
Note: If your project ID contains a colon, replace the colon with a forward slash.
After the build is complete, you will see an output similar to the following:
```
SUCCESS
```
**Congratulations!**
You've just built a Docker image named tutorial-image using a Dockerfile and pushed the image to Artifact Registry. (But your not done yet,)
## Step 4. Build using a build config file
In this section you will use a Cloud Build config file to build the same Docker image as above. The build config file instructs Cloud Build to perform tasks based on your specifications.
In the same directory that contains tutorial.sh and the Dockerfile, 
**create a file** named cloudbuild.yaml. This file is your build config file. At build time, Cloud Build automatically replaces $PROJECT_ID with your project ID. enter the following contents into the file:
```
steps:
- name: 'gcr.io/cloud-builders/docker'
 args: [ 'build', '-t', 'us-central1-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1', '.' ]
images:
- 'us-central1-docker.pkg.dev/$PROJECT_ID/quickstart-docker-repo/quickstart-image:tag1'
```
Start the build by running the following command:
```
gcloud builds submit --config cloudbuild.yaml
```
When the build is complete, you will see an output similar to the following:
```
DONE
------------------------------------------------------------------------------------------------------------------------------------
ID                                    CREATE_TIME                DURATION  SOURCE   IMAGES     STATUS
545cb89c-f7a4-4652-8f63-579ac974be2e  2020-11-05T18:16:04+00:00  16S   SUCCESS

```
**Congratulations!**
You've just built tutorial-image using the build config file and pushed the image to Artifact Registry.
## Step 5. View build details
**Open the Cloud Build page** in the Google Cloud Console.
2. Select your project and then **Click Open**.
You will see the Build history page
3. Click on your latest build.
You will see the Build details page.
4. To view the artifacts of your build, under Build Summary, click Build Artifacts.
You can download your build log and view your image details in the Artifact Registry from here.
## Clean up
To avoid incurring charges to your Google Cloud account for the resources used in this page, follow these steps.
**Open the Artifact Registry page** in the Google Cloud Console.
Select your project and click Open.
**Select tutorial-docker-repo**.
**Click Delete**.
You have now deleted the repository.
