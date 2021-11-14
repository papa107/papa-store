# Interactive Tutorial 10.1-  GitOps-style continuous delivery with Cloud Build

## Overview
This tutorial uses two Git repositories:
* app repository: contains the source code of the application itself
* env repository: contains the manifests for the Kubernetes Deployment

When you push a change to the app repository, the Cloud Build pipeline runs tests, builds a container image, and pushes it to Container Registry. After pushing the image, Cloud Build updates the Deployment manifest and pushes it to the env repository. This triggers another Cloud Build pipeline that applies the manifest to the GKE cluster and, if successful, stores the manifest in another branch of the env repository.

We keep the app and env repositories separate because they have different lifecycles and uses. The main users of the app repository are actual humans and this repository is dedicated to a specific application. The main users of the env repository are automated systems (such as Cloud Build), and this repository might be shared by several applications. The env repository can have several branches that each map to a specific environment (you only use production in this tutorial) and reference a specific container image, whereas the app repository does not.

When you finish this tutorial, you will have built a system where you can easily:
* Distinguish between failed and successful deployments by looking at the Cloud Build history,
* Access the manifest currently used by looking at the production branch of the env repository,
* Rollback to any previous version by re-executing the corresponding Cloud Build build.

This tutorial uses Cloud Source Repositories to host Git repositories, but you can achieve the same results with other third-party products such as GitHub, Bitbucket, or GitLab.
This pipeline does not implement a validation mechanism before the deployment. If you use GitHub, Bitbucket, or GitLab, you can modify the pipeline to use a Pull Request for this purpose.
While we recommend Spinnaker to the teams who want to implement advanced deployment patterns (blue/green, canary analysis, multi-cloud, etc.), its feature set might not be needed for a successful CI/CD strategy for smaller organizations and projects. In this tutorial, you learn how to create a CI/CD pipeline fit for applications hosted on GKE with tooling.
For simplicity, this tutorial uses a single environment —production— in the env repository, but you can extend it to deploy to multiple environments if needed.

**Objectives**
* Create Git repositories in Cloud Source Repositories.
* Create a container image with Cloud Build and store it in the Container Registry.
* Create a CI pipeline.
* Create a CD pipeline.
* Test the CI/CD pipeline.

**Before You Begin**

Set your Project_ID in Cloud Shell
```
gcloud config set project [PROJECT_ID]
```
In Cloud Shell, enable the required APIs.
```
gcloud services enable container.googleapis.com \
cloudbuild.googleapis.com \ 
sourcerepo.googleapis.com \
containeranalysis.googleapis.com
```
## Step 1: Build the Infrastructure
In Cloud Shell, create a GKE cluster that you will use to deploy the sample application of this tutorial.
Create a cluster named hello-cloudbuild:
```
gcloud container clusters 
create  hello-cloudbuild \
 --region us-central1
```
If you have never used Git in Cloud Shell, configure it with your name and email address. Git will use those to identify you as the author of the commits you will create in Cloud Shell.
```
git config --global user.email "[YOUR_EMAIL_ADDRESS]"
git config --global user.name "[YOUR_NAME]"
```
Run the gcloud credential helper. This will connect your Google Cloud user account to Cloud Source Repositories.
```
git config --global credential.helper gcloud.sh
```
## Step 2 Creating the Git repositories in Cloud Source Repositories
In this section, you create the two Git repositories (app and env) used in this tutorial, and initialize the app one with some sample code.
In Cloud Shell, create the two Git repositories.
```
gcloud source repos create hello-cloudbuild-app
gcloud source repos create hello-cloudbuild-env
```

Clone the sample code from GitHub.
```
cd ~
git clone https://github.com/GoogleCloudPlatform/gke-gitops-tutorial-cloudbuild \
Hello-cloudbuild-app
```

## Step 3. Configure Cloud Source Repositories as a remote.
```
cd ~/hello-cloudbuild-app
PROJECT_ID=$(gcloud config get-value project)
git remote add google \
    "https://source.developers.google.com/p/${PROJECT_ID}/r/hello-cloudbuild-app"
```
The code you cloned contains a "Hello World" application.
app.py
```
from flask import Flask
app = Flask('hello-cloudbuild')

@app.route('/')
def hello():
  return "Hello World!\n"

if __name__ == '__main__':
  app.run(host = '0.0.0.0', port = 8080)
```
## Step 4. Creating a container image with Cloud Build
The code you cloned contains the following Dockerfile.
Dockerfile
```
FROM python:3.7-slim
RUN pip install flask
WORKDIR /app
COPY app.py /app/app.py
ENTRYPOINT ["python"]
CMD ["/app/app.py"]
With this Dockerfile, you can create a container image with Cloud Build and store it in the Container Registry.
In Cloud Shell, create a Cloud Build build based on the latest commit with the following command.
cd ~/hello-cloudbuild-app
COMMIT_ID="$(git rev-parse --short=7 HEAD)"
gcloud builds submit --tag="gcr.io/${PROJECT_ID}/hello-cloudbuild:${COMMIT_ID}" .
```
Cloud Build streams the logs generated by the creation of the container image to your terminal when you execute this command.

After the build finishes, verify that your new container image is available in the Container Registry.

## Step 5. Creating the continuous integration pipeline
In this section, you configure Cloud Build to automatically run a small unit test, build the container image, and then push it to the Container Registry. Pushing a new commit to Cloud Source Repositories automatically triggers this pipeline. The cloudbuild.yaml file included in the code is the pipeline's configuration.

Cloudbuild.yaml
```
# This step runs the unit tests on the app
- name: 'python:3.7-slim'
  id: Test
  entrypoint: /bin/sh
  args:
  - -c
  - 'pip install flask && python test_app.py -v'

# This step builds the container image.
- name: 'gcr.io/cloud-builders/docker'
  id: Build
  args:
  - 'build'
  - '-t'
  - 'gcr.io/$PROJECT_ID/hello-cloudbuild:$SHORT_SHA'
  - '.'

# This step pushes the image to Container Registry
# The PROJECT_ID and SHORT_SHA variables are automatically
# replaced by Cloud Build.
- name: 'gcr.io/cloud-builders/docker'
  id: Push
  args:
  - 'push'
  - 'gcr.io/$PROJECT_ID/hello-cloudbuild:$SHORT_SHA'
```

## Step 6. Open the Cloud Build Triggers page.

**Click Create trigger**.
Fill out the following options:
* In the Name field, type hello-cloudbuild.
* Under Event, select Push to a branch.
* Under Source, select hello-cloudbuild-app as your Repository and ^master$ as your Branch.
* Under Build configuration, select Cloud Build configuration file.
* In the Cloud Build configuration file location field, type cloudbuild.yaml after the /.
* Click Create to save your build trigger.

## Step 7 push the application code to Cloud Source Repositories

In Cloud Shell, push the application code to Cloud Source Repositories to trigger the CI pipeline in Cloud Build.
```
cd ~/hello-cloudbuild-app
git push google master
```
**Open the Cloud Build console**.

Your recently run and finished builds appear. You can click on a build to follow its execution and examine its logs.

## Step 8 Creating the continuous delivery pipeline

Cloud Build is also used for the continuous delivery pipeline. The pipeline runs each time a commit is pushed to the candidate branch of the hello-cloudbuild-env repository. The pipeline applies the new version of the manifest to the Kubernetes cluster and, if successful, copies the manifest over to the production branch. This process has the following properties:
* The candidate branch is a history of the deployment attempts.
* The production branch is a history of the successful deployments.
* You have a view of successful and failed deployments in Cloud Build.
* You can rollback to any previous deployment by re-executing the corresponding build in Cloud Build. A rollback also updates the production branch to truthfully reflect the history of deployments.
* You will modify the continuous integration pipeline to update the candidate branch of the hello-cloudbuild-env repository, triggering the continuous delivery pipeline.

Note: You can extend the system described in this tutorial to manage several environments. The easiest way to achieve this is to have a pair of branches for each environment env: a candidate-env branch and an env branch.

## Step 9 Granting Cloud Build access to GKE

To deploy the application in your Kubernetes cluster, Cloud Build needs the Kubernetes Engine Developer Identity and Access Management Role.

In Cloud Shell, execute the following command:
```
PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} --format='get(projectNumber)')"
gcloud projects add-iam-policy-binding ${PROJECT_NUMBER} \
    --member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
    --role=roles/container.developer
```

## Step 10 Initializing the hello-cloudbuild-env repository

You need to initialize the hello-cloudbuild-env repository with two branches (production and candidate) and a Cloud Build configuration file describing the deployment process.
In Cloud Shell, clone the hello-cloudbuild-env repository and create the production branch.
```
cd ~
gcloud source repos clone hello-cloudbuild-env
cd ~/hello-cloudbuild-env
git checkout -b production
```
Copy the cloudbuild-delivery.yaml file available in the hello-cloudbuild-app repository and commit the change.
```
cd ~/hello-cloudbuild-env
cp ~/hello-cloudbuild-app/cloudbuild-delivery.yaml ~/hello-cloudbuild-env/cloudbuild.yaml
git add .
git commit -m "Create cloudbuild.yaml for deployment"
```
The cloudbuild-delivery.yaml file describes the deployment process to be run in Cloud Build. It has two steps:
* Cloud Build applies the manifest on the GKE cluster.
* If successful, Cloud Build copies the manifest on the production branch.

## Step 11 Create a candidate branch

Push both branches for them to be available in Cloud Source Repositories.
```
git checkout -b candidate
git push origin production
git push origin candidate
```
Grant the Source Repository Writer IAM role to the Cloud Build service account for the hello-cloudbuild-env repository.
```
PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} \
 --format='get(projectNumber)')"
cat >/tmp/hello-cloudbuild-env-policy.yaml <<EOF
bindings:
- members:
  - serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com
  role: roles/source.writer
EOF
gcloud source repos set-iam-policy \ hello-cloudbuild-env /tmp/hello-cloudbuild-env-policy.yaml
```
 
## Step 12 Creating the trigger for the continuous delivery pipeline

In this section, you configure Cloud Build to be triggered by a push to the candidate branch of the hello-cloudbuild-env repository.
**Open the Triggers** page of Cloud Build.
**Click Create trigger**.
Fill out the following options:
* In the Name field, type hello-cloudbuild-deploy.
* Under Event, select Push to a branch.
* Under Source, select hello-cloudbuild-env as your Repository and ^candidate$ as your Branch.
* Under Configuration, select Cloud Build configuration file (yaml or json).
* In the Cloud Build configuration file location field, type cloudbuild.yaml after the /.
**Click Create**.

## Step 13 Modifying the continuous integration pipeline to trigger the continuous delivery pipeline

In this section, you add some steps to the continuous integration pipeline that generates a new version of the Kubernetes manifest and push it to the hello-cloudbuild-env repository to trigger the continuous delivery pipeline.
Replace the cloudbuild.yaml file with the extended example in the cloudbuild-trigger-cd.yaml file.
```
cd ~/hello-cloudbuild-app
cp cloudbuild-trigger-cd.yaml cloudbuild.yaml
```
The cloudbuild-trigger-cd.yaml is an extended version of the cloudbuild.yaml file. It adds steps to generate the new Kubernetes manifest and trigger the continuous delivery pipeline.
Note: This pipeline uses a sed to render the manifest template. In reality, you might benefit from using a dedicated tool such as kustomize or skaffold. These tools give you more control over the rendering of the manifest templates.
Cloudbuild-trigger-cd.yaml
```
# This step clones the hello-cloudbuild-env repository
- name: 'gcr.io/cloud-builders/gcloud'
 id: Clone env repository
  entrypoint: /bin/sh
  args:
  - '-c'
  - |
    gcloud source repos clone hello-cloudbuild-env && \
    cd hello-cloudbuild-env && \
    git checkout candidate && \
    git config user.email $(gcloud auth list --filter=status:ACTIVE --format='value(account)')

# This step generates the new manifest
- name: 'gcr.io/cloud-builders/gcloud'
  id: Generate manifest
  entrypoint: /bin/sh
  args:
  - '-c'
  - |
     sed "s/GOOGLE_CLOUD_PROJECT/${PROJECT_ID}/g" kubernetes.yaml.tpl | \
     sed "s/COMMIT_SHA/${SHORT_SHA}/g" > hello-cloudbuild-env/kubernetes.yaml

# This step pushes the manifest back to hello-cloudbuild-env
- name: 'gcr.io/cloud-builders/gcloud'
  id: Push manifest
  entrypoint: /bin/sh
  args:
  - '-c'
  - |
    set -x && \
    cd hello-cloudbuild-env && \
    git add kubernetes.yaml && \
    git commit -m "Deploying image gcr.io/${PROJECT_ID}/hello-cloudbuild:${SHORT_SHA}
    Built from commit ${COMMIT_SHA} of repository hello-cloudbuild-app
    Author: $(git log --format='%an <%ae>' -n 1 HEAD)" && \
    git push origin candidate
```

## Step 14 Commit the modifications and push them to Cloud Source Repositories.
```
cd ~/hello-cloudbuild-app
git add cloudbuild.yaml
git commit -m "Trigger CD pipeline"
git push google master
```
This triggers the continuous integration pipeline in Cloud Build.
## Step 15 Examine the continuous integration build.

Go to Cloud Build

Your recently run and finished builds for the hello-cloudbuild-app repository appear. You can click on a build to follow its execution and examine its logs. 

The last step of this pipeline pushes the new manifest to the hello-cloudbuild-env repository, which triggers the continuous delivery pipeline.
Examine the continuous delivery build.

## Step 16 Testing the complete pipeline

The complete CI/CD pipeline is now configured. In this section, you test it from end to end.

Go to the GKE Services page.

The list contains a single service called hello-cloudbuild created by the recently completed continuous delivery build.
**Click on the endpoint** for the hello-cloudbuild service. "Hello World!" appears. If there is no endpoint, or if you see a load balancer error, you might have to wait a few minutes for the load balancer to be completely initialized. Click Refresh to update the page if needed.
In Cloud Shell, replace "Hello World" by "Hello Cloud Build", both in the application and in the unit test.
```
cd ~/hello-cloudbuild-app
sed -i 's/Hello World/Hello Cloud Build/g' app.py
sed -i 's/Hello World/Hello Cloud Build/g' test_app.py
```
**Commit and push the change to Cloud Source Repositories**.
```
git add app.py test_app.py
git commit -m "Hello Cloud Build"
git push google master
```
This triggers the full CI/CD pipeline.

After a few minutes, reload the application in your browser. "Hello Cloud Build!" appears.

## Step 17 Testing the rollback
In this section, you rollback to the version of the application that said "Hello World!".

**Open the Cloud Build console** for the hello-cloudbuild-env repository.

Click on the second most recent build available.

**Click Rebuild**.

When the build is finished, reload the application in your browser. "Hello World!" appears again.

**Congratulations!** You have just built a GitOps-style continuous delivery pipeline using Cloud Build.
 
**Clean up**
To avoid incurring charges to your Google Cloud account for the resources used in this tutorial, either delete the project that contains the resources, or keep the project and delete the individual resources.
**Caution**: Deleting a project has the following effects:
* Everything in the project is deleted. 
* If you used an existing project for this tutorial, when you delete it, you also delete any other work you've done in the project.
* Custom project IDs are lost. When you created this project, you might have created a custom project ID that you want to use in the future. 
* To preserve the URLs that use the project ID, such as an appspot.com URL, delete selected resources inside the project instead of deleting the whole project.
* If you plan to explore multiple tutorials and quickstarts, reusing projects can help you avoid exceeding project quota limits.
  
In the Cloud Console, go to the Manage resources page.

In the project list, select the project that you want to delete, and then click Delete.

  In the dialog, type the project ID, and then click Shut down to delete the project.

 ** OR YOU CAN Delete the Resources**

If you want to keep the Google Cloud project you used in this tutorial, delete the individual resources:
Delete the local Git repositories.
```
cd ~
rm -rf ~/hello-cloudbuild-app
rm -rf ~/hello-cloudbuild-env
```
Delete the Git repositories in Cloud Source Repositories.
```
gcloud source repos delete hello-cloudbuild-app --quiet
gcloud source repos delete hello-cloudbuild-env --quiet
```
Delete the Cloud Build Triggers.
Open the Triggers page of Cloud Build.
For each trigger, click More, then Delete.

Delete the images in Container Registry.
```
gcloud beta container images list-tags \
    gcr.io/${PROJECT_ID}/hello-cloudbuild \
    --format="value(tags)" | \
    xargs -I {} gcloud beta container images delete \
    --force-delete-tags --quiet \
    gcr.io/${PROJECT_ID}/hello-cloudbuild:{}
```
Remove the permission granted to Cloud Build to connect to GKE.
```
PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} \
    --format='get(projectNumber)')"
gcloud projects remove-iam-policy-binding ${PROJECT_NUMBER} \
    --member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
    --role=roles/container.developer
```
Delete the GKE cluster.
```
gcloud container clusters delete hello-cloudbuild \
 --region us-central1
```
