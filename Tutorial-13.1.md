# Interactive Tutorial 13-1 Validating Apps Against Policy
 
## Overview
This tutorial demonstrates how to validate your apps against the company policies in a CI pipeline.
Validating your app is useful if you are a developer building a CI pipeline for an app, or a platform engineer building a CI pipeline template for multiple app teams.

Policies are an important part of the security and compliance of an organization. Policy Controller, which is part of Anthos Config Management, allows your organization to manage those policies centrally and declaratively for all your clusters. As a developer, you can take advantage of the centralized and declarative nature of those policies. You can use those characteristics to validate your app against those policies as early as possible in your development workflow. Learning about policy violations in your CI pipeline instead of during the deployment has two main advantages: it lets you shift left on security, and it tightens the feedback loop, reducing the time and cost necessary to fix those violations.

This tutorial uses Cloud Build as a CI tool and a sample GitHub repository containing policies for demonstrations.

Resources
This tutorial uses several Kubernetes tools. 

The tools that you use in this tutorial include the following:

* Policy Controller: Policy Controller is a Google Cloud product that is part of Anthos Config Management. 
* GitHub: In this tutorial, we use GitHub to host the Git repositories: one for a sample app, and one for Anthos Config Management.
* Cloud Build: Cloud Build is Google Cloud's CI solution. In this tutorial, we use it to run the validation tests.
* Kustomize: Kustomize is a customization tool for Kubernetes configurations.
* Kpt: Kpt is a tool to build workflows for Kubernetes configurations. Kpt lets you fetch, display, customize, update, validate, and apply Kubernetes configurations.

Pipeline
The CI pipeline we use in this tutorial runs in Cloud Build, and the commands are run in a directory containing a copy of the sample app repository. 

The pipeline starts by generating the final Kubernetes configurations with Kustomize. 

Next, it fetches the constraints that we want to validate against from the Anthos Config Management repository using kpt. 

Finally, it uses kpt to validate the Kubernetes configurations against those constraints. 

To achieve this last step, we use a specific config function called gatekeeper-validate that performs this validation. 

In this tutorial, you trigger the CI pipeline manually, but in reality you would configure it to run after a git push to your Git repository.

Objectives
* Run a CI pipeline for a sample app with Cloud Build.
* Observe that the pipeline fails because of a policy violation.
* Modify the sample app repository to comply with the policies.
* Run the CI pipeline again successfully.

Note: You don't need an Anthos entitlement to run this tutorial. However, you do need one to run Anthos Config Management in a cluster.

Before you begin
* Select or create a Google Cloud project.
* To execute the commands listed in this tutorial, open Cloud Shell:
In Cloud Shell, run:
```
gcloud config get-value project
```
If the command does not return the ID of the project that you just selected, configure Cloud Shell to use your project: Replace PROJECT_ID with your project ID.
```
gcloud config set project PROJECT_ID
```

In Cloud Shell, enable the required Cloud Build API:
```
gcloud services enable cloudbuild.googleapis.com
```

## Step 1. Validating the sample app configurations

In this section, you run a CI pipeline with Cloud Build for a sample app repository that we provide. This pipeline validates the Kubernetes configuration available in that sample app repository against constraints available in a sample Anthos Config Management repository.

Step 1.1 To validate the app configurations:

In Cloud Shell, clone the sample app repository:
```
git clone https://github.com/GoogleCloudPlatform/anthos-config-management-samples.git
```

## Step 2. Run the CI pipeline with Cloud Build. 

2.1 Run the pipeline the logs of the build are displayed directly in Cloud Shell.
```
cd anthos-config-management-samples/ci-app/app-repo
gcloud builds submit .
```

The pipeline that you run is defined in the following file.
ci-app/app-repo/cloudbuild.yaml
```
steps:
- id: 'Prepare config'
  # This step builds the final manifests for the app
  # using kustomize and the configuration files
  # available in the repository.
  name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
  entrypoint: '/bin/sh'
  args: ['-c', 'mkdir hydrated-manifests && kubectl kustomize config/prod > hydrated-manifests/prod.yaml']
- id: 'Download policies'
  # This step fetches the policies from the Anthos Config Management repository
  # and consolidates every resource in a single file.
  name: 'gcr.io/kpt-dev/kpt'
  entrypoint: '/bin/sh'
  args: ['-c', 'kpt pkg get https://github.com/GoogleCloudPlatform/anthos-config-management-samples.git/ci-app/acm-repo/cluster@1.0.0 constraints
                  && kpt fn source constraints/ hydrated-manifests/ > hydrated-manifests/kpt-manifests.yaml']
- id: 'Validate against policies'
  # This step validates that all resources comply with all policies.
  name: 'gcr.io/config-management-release/policy-controller-validate'
  args: ['--input', 'hydrated-manifests/kpt-manifests.yaml']
```
Step 2.2 Analyse the code 
While the pipeline is running take the time to review the yaml file. In the cloubuild.yaml file the 2nd step:
```
- id: 'Download policies'
  # This step fetches the policies from the Anthos Config Management repository
  # and consolidates every resource in a single file.
  name: 'gcr.io/kpt-dev/kpt'
  entrypoint: '/bin/sh'
  args: ['-c', 'kpt pkg get 
```
We see that the kpt pkg get command is needed and used to download both constraint templates and constraints before they are used in step 3:
```
- id: 'Validate against policies'
  # This step validates that all resources comply with all policies.
```
After a few minutes, observe that the pipeline fails with the following error:
```
[...]
Step #2 - "Validate against policies": [RUNNING] "gcr.io/kpt-fn/gatekeeper:v0"
Step #2 - "Validate against policies": [FAIL] "gcr.io/kpt-fn/gatekeeper:v0"
Step #2 - "Validate against policies":   Results:
Step #2 - "Validate against policies":     [ERROR] Deployment objects should have an 'owner' label indicating who created them. violatedConstraint: deployment-must-have-owner in object "apps/v1/Deployment/nginx-deployment" in file "prod.yaml"
Step #2 - "Validate against policies":   Stderr:
Step #2 - "Validate against policies":     "[error] apps/v1/Deployment/nginx-deployment : Deployment objects should have an 'owner' label indicating who created them."
Step #2 - "Validate against policies":     "violatedConstraint: deployment-must-have-owner"
Step #2 - "Validate against policies":     ""
Step #2 - "Validate against policies":   Exit code: 1
[...]
```
## Step 3 View the Logs in the console
3.1 Note that the constraint that the configuration is violating is defined in the following file. It's a Kubernetes custom resource called K8sRequiredLabels.
ci-app/acm-repo/cluster/deployment-must-have-owner.yaml
```
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: deployment-must-have-owner
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    labels:
      - key: "owner"
    message: "Deployment objects should have an 'owner' label indicating who created them."
```
For the constraint template corresponding to this constraint, see requiredlabels.yaml on GitHub.

3.2 Build the full Kubernetes configuration yourself, and observe that the owner label is indeed missing. 

To build the configuration:
```
kubectl kustomize config/prod
```

3.3 Fix the app to comply with company policies

In this section, you will fix the policy violation using Kustomize:

In Cloud Shell, add a commonLabels section to the base Kustomization file:
```
cat <<EOF >> config/base/kustomization.yaml
commonLabels:
  owner: myself
EOF
```

3.4 Build the full Kubernetes configuration, and observe that the owner label is now present:
```
kubectl kustomize config/prod
```

3.5 Rerun the CI pipeline with Cloud Build:
```
gcloud builds submit .
```

The pipeline now succeeds with the following output:
```
[...]
Step #2 - "Validate against policies": [RUNNING] "gcr.io/kpt-fn/gatekeeper:v0"
Step #2 - "Validate against policies": [PASS] "gcr.io/kpt-fn/gatekeeper:v0"
[...]
```
Tip: You can run and debug Cloud Build pipelines locally without having to trigger an actual build or pushing to your Git repository using cloud- build-local.

**Congratulations!** You have completed the last tutorial in this book.

Cleaning up

As this is the last interactive tutorial in the book you can safely delete the project.
In the project list, select the project that you want to delete, and then click Delete.
In the dialog, type the project ID, and then click Shut down to delete the project.
