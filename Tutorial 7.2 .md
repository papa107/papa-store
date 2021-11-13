# Interactive Tutorial 7.2  Building your first Kubernetes App

##Overview

In this tutorial you will use Cloud Shell as it comes preinstalled with the gcloud command-line tool and the kubectl command-line tool. The gcloud tool provides the primary command-line interface for Google Cloud, and kubectl provides the primary command-line interface for running commands against Kubernetes clusters.

Step 1 Launch Cloud Shell

To Launch Cloud Shell: Type in the browser: 
```
console.cloud.google.com
```
OR

From the upper-right corner of the console, click the **Activate Cloud Shell** button: 

A Cloud Shell terminal session opens inside a frame in the bottom half of the console. This is the terminal we will configure to use to run gcloud and kubectl commands.

## Step 2 Set default settings for the gcloud tool

To make entering some of the commands easier you are going to configure your Cloud Shell terminal session to use the following default settings: your default project, compute zone, and compute region. Configuring these default settings makes it easier to run gcloud commands, because gcloud requires that you specify the project and location in which you want to work. If you do not configure our own default settings then you would have to specify these settings with flags, such as --project, --zone, --region, and --cluster, in your individual gcloud commands. 
When you create GKE resources after configuring your default project, zone, and region, the resources are automatically created in that project, zone, and region.

In Cloud Shell, perform the following steps:

Step 1. Configure the Cloud Shell environment

Set the default project. Replace PROJECT_ID with your project ID:
```
gcloud config set project PROJECT_ID
```

Set the default zone. Replace COMPUTE_ZONE with your compute zone, such as asia-south2-a
```
gcloud config set compute/zone COMPUTE_ZONE
```

Set the default region: Replace COMPUTE_REGION with your compute region, such as asia-south2.
```
gcloud config set compute/region COMPUTE_REGION
```

## Step 3. Enable the APIs

Go to the Cloud Console and enable the APIs for:
Compute Engine
Kubernetes Engine

## Step 4. Create a GKE cluster

A cluster consists of at least one cluster control plane machine and multiple worker machines called nodes. You will use Compute Engine virtual machine (VM) instances as the nodes that will run the Kubernetes processes. You build the nodes that run the processes necessary to make them part of the cluster then you will deploy your application to the cluster.
To create clusters in GKE, you need to choose a mode of operation: Standard or Autopilot. If you use the Standard mode, your cluster is zonal (in this tutorial). If you use the Autopilot mode, your cluster is regional.

4.1 Create a one-node Standard cluster named hello-cluster:
```	
gcloud container clusters create hello-cluster --num-nodes=1
```

It might take several minutes to finish creating the cluster.


4.2 Get authentication credentials for the cluster
After creating your cluster, you need to get authentication credentials to interact with the cluster:
```
gcloud container clusters get-credentials hello-cluster
```

This command configures kubectl to use the cluster you created.

4.3 Deploy an application to the cluster
Now that you have created a cluster, you can deploy a containerized application to it. For this tutorial, you can deploy the example web application, hello-app.
GKE uses Kubernetes objects to create and manage your cluster's resources. Kubernetes provides the Deployment object for deploying stateless applications like web servers. Service objects define rules and load balancing for accessing your application from the internet. You will use a tool called kubectl to help interact with the Kubernetes API.

## Step 5 Create the Deployment
To run hello-app in your cluster, you need to deploy the application by running the following command:
```
kubectl create deployment hello-server \
    --image=us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
```

This Kubernetes command, kubectl create deployment, creates a Deployment named hello-server. The Deployment's Pod runs the hello-app container image.

In this command:

--image specifies a container image to deploy. In this case, the command pulls the example image from an Artifact Registry repository, us-docker.pkg.dev/google-samples/containers/gke/hello-app. :1.0 indicates the specific image version to pull. If you don't specify a version, the image with the default tag latest is used.

## Step 6. Expose the Deployment
After deploying the application, you need to expose it to the internet so that users can access it. You can expose your application by creating a Service, a Kubernetes resource that exposes your application to external traffic.

To expose your application, run the following kubectl expose command:
```
kubectl expose deployment hello-server --type LoadBalancer --port 80 --target-port 8080
```

Passing in the --type LoadBalancer flag creates a Compute Engine load balancer for your container. 
The --port flag initializes public port 80 to the internet and
 the --target-port flag routes the traffic to port 8080 of the application.

This command creates a Kubernetes API object called a deployment. A deployment is an abstraction that manages the life cycle of an application. You set the desired number of app instances for the deployment to manage, and then it will make sure the correct number of instances, or replicas, are kept running.

So if we increase the number of replicas that we want, the deployment will see that there's currently not enough replicas and spin up another one.
That even works when a node crashes. If the node goes down, the current state is once again different from the desired state, then Kubernetes will schedule another replica for us.

## Step 7. Inspect and view the application

7.1 Inspect the running Pods by using kubectl get pods:
```
kubectl get pods
```

You should see one hello-server Pod running on your cluster.


7.2 Inspect the hello-server Service by using kubectl get service:
```
kubectl get service hello-server
```

You have just created a service. The service creates an endpoint that we can use to access the running app instances. In this case, we have multiple app instances. So this service will loadbalance incoming requests between the two running pods.
For any container inside of the cluster they can connect to my service using the service's name.
Either way, the service keeps track of wherever the pod is running.
This is a good example of how Kubernetes removes the need to manually keep track of where your containers are running. Even if a pod were to go down, once a new one comes back online, the service will automatically update its list of endpoints to target the new pod.
So Kubernetes objects, like deployments and services, automatically ensure that we have the right number of app instances running through pods, and that we can always reach them.

7.3 From this command's output, copy the Service's external IP address from the EXTERNAL-IP column.
Note: You might need to wait several minutes before the Service's external IP address populates. If the application's external IP is <pending>, run the kubectl command again.

5.4 View the application from your web browser by using the external IP address with the exposed port:
*Note in my case the IP is 34.131.209.53 Yours will be  different so use your IP address.
http://34.131.209.53

**Congratulations!** - You have just deployed a containerized web application to GKE.

**Clean up**
To avoid incurring charges to your Google Cloud account for the resources used in this page, follow these steps.

Delete the application's Service by running kubectl delete:
```
kubectl delete service hello-server
```

This command deletes the Compute Engine load balancer that you created when you exposed the Deployment.

Delete your cluster by running gcloud container clusters delete:
```
gcloud container clusters delete hello-cluster
```
