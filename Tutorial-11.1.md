# Interactive Tutorial 11.1  Building a CI/CD in Anthos

##Overview

**Objectives**
* Deploy the reference architecture infrastructure.
* Explore the infrastructure.
* Explore the code repositories and pipelines.
* Explore an example application landing zone.
 
## Step 1 Deploying the Architecture

Exploring the infrastructure
In this section, you explore the main components of the CI/CD system, including the infrastructure, code repositories, and a sample landing zone.
In the Google Cloud Console, **go to the Kubernetes clusters page**.

This page lists the clusters that are used for the development (dev-us-west1), shared tools (gitlab), and application environments (staging-us-west2, prod-us-central1, prod-us-east1):

Development cluster
The development cluster (dev-us-west1) gives your developers access to a namespace that they can use to iterate on their applications. We recommend that teams use tools like Skaffold that provide an iterative workflow by actively monitoring the code under development and reapplying it to the development environments as changes are made. This iteration loop is similar to hot reloading, but instead of being programming language-specific, the loop works with any application that you can build with a Docker image. You can run the loop inside a Kubernetes cluster.
 
Shared tools cluster
Any software delivery system uses a mixture of tools to support the software development lifecycle. Following the principle of least privilege, the reference implementation provides a dedicated cluster for storing tools (gitlab). We recommend that you deploy each tool to its own namespace.
In this reference implementation, you use GitLab for source code management and continuous integration. You install GitLab in the tools cluster by using the GitLab on GKE Terraform module.

Application environment clusters
Also included in the reference architecture are clusters (staging-us-west2, prod-us-central1, prod-us-east1) to run your applications for both pre-production (staging) and production deployments. You should deploy your applications to at least one cluster in each environment. For geo-redundancy or high-availability (HA) systems, we recommend that you add multiple clusters to each environment. For all clusters where applications are deployed, it's ideal to use regional clusters. This approach insulates your applications from zone-level failures and any interruptions caused by cluster or node pool upgrades.
We recommend that you use Anthos Config Management to sync the configuration of cluster resources such as namespaces, quotas, and RBAC. For more details on how to manage those resources, see Configuration and policy repositories later in this document.

Exploring the code repositories
In this section, you explore the code repositories.

**Log in to the GitLab instance**
In Cloud Shell, **get the GitLab URL**:
```
echo "https://gitlab.endpoints.${PROJECT_ID}.cloud.goog"
```
Copy the URL because you need it for a later step.

Retrieve the GitLab User and Password, which are stored in Secrets Manager:
```
export GITLAB_USER=$(gcloud secrets versions access latest --secret="gitlab-user")
export GITLAB_PASSWORD=$(gcloud secrets versions access latest --secret="gitlab-password")

echo "User: ${GITLAB_USER}"
echo "Password: ${GITLAB_PASSWORD}"
```
Copy these credentials because you need them for a later step.

In a web browser, go to the GitLab URL that you copied earlier.
Using the User and Password credentials that you copied, log in to your GitLab instance.

The Projects page for your GitLab instance is displayed.
Explore the operator repositories

The operator, starter, and configuration repositories are where operators and platform administrators define the common best practices for building on and operating the platform. These repositories are all located in the platform-admins group.
In the GitLab instance, **click Groups**, and then **select Your Groups**.

**Click platform-admins**.
A list of repositories is displayed:

## Step 2 Applying the reference architecture
Now that you've explored the reference architecture, you can explore a developer workflow that is based on this implementation. 
If you've never used Git in Cloud Shell, configure Git with your name and email address. Git uses this information to identify you as the author of the commits that you create in Cloud Shell.
```
git config --global user.email "GIT_EMAIL_ADDRESS"
git config --global user.name "GIT_USERNAME"
```
Replace the following:
GIT_EMAIL_ADDRESS: the email address associated with your Git account
GIT_USERNAME: the username associated with your Git account

Install kustomize.
Adding a new feature to the application
When you develop a new feature, you need to quickly deploy your changes into a development sandbox in order to test and iterate on them. 
In this tutorial, you use Skaffold to actively monitor your changes and deploy them to a development sandbox.
Skaffold generates a configuration file named skaffold.yaml. 
This file defines the Docker images and Kubernetes manifests that you use to deploy the application. When you run skaffold dev, the continuous development loop begins. As you make changes to the application, Skaffold automatically rebuilds the necessary Docker images and deploys the latest version of your development code to the development cluster.

## Step 3 Deploying your change to the staging cluster
After you successfully push your changes into the feature branch of the application code repository, you can deploy them to the staging cluster. To deploy to the staging cluster, merge your changes into the main branch of the application repository. This action triggers a CI process to test the code, render the Kubernetes manifests, and push the rendered manifests into the staging branch of the application's configuration repository. When the CI process pushes the manifests into the application's configuration repository, a CD job starts, which deploys the manifests to the staging cluster.
To deploy your changes to the staging cluster, do the following:
In a web browser, **go to GitLab**, and **log in** using the URL and username and password from the reference architecture.
**Click Groups**, and then **select Your Groups**.
**Click hello-world-golang** to go to the application code repository.
To see your changes in the repository, select your feature branch from the Branches list:

In the adjacent pane, click Merge Requests, and then click Create merge request to create a merge request.
Select the Delete source branch when merge request is accepted option, and then click Submit merge request.
Click Merge.
In the menu, click CI/CD to view the execution of the CI pipeline.
Click Running to get more details.
Note: When the pipeline is completed, the updated manifests have been pushed into the hello-world-golang-env repository. That push triggered a CD job to deploy your code changes to the staging cluster.
Click Groups, and then select Your Groups.
Click hello-world-golang.
Click hello-world-golang-env to view the application configuration repository.
In the menu, click CI/CD to view the execution of the CD pipeline.
Note: When the pipeline is completed, the code changes have been deployed to the staging cluster.
View the changes on the staging cluster
In Cloud Shell, get credentials to the staging cluster:
gcloud container clusters --region us-west2 get-credentials staging-us-west2
Rename your context:
kubectx staging=gke_${PROJECT_ID}_us-west2_staging-us-west2
Switch to your staging cluster context:
kubectx staging
Create a port forward:
kubectl port-forward svc/hello-world-golang-app -n hello-world-golang 8080:8080
On the Cloud Shell toolbar, click preview Web Preview, and then click Preview on port 8080.
The output is the following:
My new feature!
 In Cloud Shell, press CTRL+C to end the port forward.
Promoting to the production clusters
After you verify your changes in the staging cluster, you are ready to promote the changes to the production clusters. The hello-world-golang-env repository contains a branch for each application environment. Updates to the application configuration automatically trigger the CD pipelines, which deploy the application to the environments associated with the branch—for example, updates to the staging branch trigger a deployment to the staging environment. Storing the application configuration in Git and automating the deployment of the applications form the foundation of the GitOps process. Storing the manifests in Git improves the auditability of configuration changes and deployments.
To trigger the deployment to production, you merge the application configuration from the staging branch into the main branch of the hello-world-golang-env repository.
In GitLab, click Groups, and then select Your Groups.
Click hello-world-golang.
Click hello-world-golang-env.
In the adjacent pane, click Merge Requests, and then click Create merge request to create a merge request.
Unselect Delete source branch when merge request is accepted.
Click Submit merge request.
Click Merge.
In the menu, click CI/CD to view the pipeline execution.
After the deployment to the prod-us-central1 cluster completes, click play_arrow Play to approve the rollout to the prod-us-east1 cluster.
View the changes on a production cluster
In Cloud Shell, get credentials to the prod-us-central1 cluster:
gcloud container clusters get-credentials prod-us-central1
Rename your context:
kubectx prod-central=gke_${PROJECT_ID}_us-central1_prod-us-central1
Switch to your prod-central cluster context:
kubectx prod-central
Create a port forward:
kubectl port-forward svc/hello-world-golang-app -n hello-world-golang 8080:8080
On the Cloud Shell toolbar, click preview Web Preview, and then click Preview on port 8080.
The output is the following:
My new feature!
In Cloud Shell, press CTRL+C to end the port forward.
Clean up
To avoid incurring charges to your Google Cloud account for the resources used in this tutorial.
Delete the project
