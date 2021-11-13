# Interactive Tutorial 7.1 
## Creating a GKE Cluster
In this introductory tutorial we will explore Google Kubernetes Engine (GKE) and learn hands-on how it provides a managed environment for deploying, managing, and scaling your containerized applications using Google Cloud infrastructure. 

Step 1: set defaults - Project_ID, compute zone
First thing you must do is to export to the Cloud Shell your default Project_ID,  compute zone and region. For example, us-central1-a is a zone in the us-central1 region.
To set your default compute zone run the following command:
```
gcloud config set compute/zone (Your Zone)
```
Expected output: Updated property [compute/zone].

Step 2: Create a GKE cluste
To create a cluster, run the following command, replacing [CLUSTER-NAME] with the name you choose for the cluster (for example: my-cluster):
```
gcloud container clusters create [CLUSTER-NAME]
```
You can ignore any warnings in the output. Please be patient as It might take several minutes to finish creating the cluster.

Step 3: Get authentication credentials for the cluster
After creating your cluster, you need to obtain authentication credentials in order to interact with it.
To authenticate the cluster, run the following command, replacing [CLUSTER-NAME] with the name of your cluster:
```
gcloud container clusters get-credentials [CLUSTER-NAME]
```

Step 4: Deploy an application to the cluster
Now that you have a cluster built you can go ahead and deploy a containerized application to the cluster. For this lab, you'll run hello-app in your cluster.
To create a new Deployment hello-server from the hello-app container image, run the following command:
```
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```
Expected output: deployment.apps/hello-server created

Step 5. To create a Kubernetes Service, which is a Kubernetes resource that lets you expose your application to external traffic, run the following command:
```
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```
 Note: In this command:
--port specifies the port that the container exposes.
type="LoadBalancer" creates a Compute Engine load balancer for your container.

Step 6. To inspect the hello-server Service, run the following command:
```
kubectl get service
```
Note: It might take a minute for an external IP address to be generated. Try running the command again.

Step 7. To view the application from your web browser, open a new tab and enter the following address, replacing [EXTERNAL IP] with the EXTERNAL-IP for hello-server.
http://[EXTERNAL-IP]:8080

**Congratulations!** You have just created your first KGE Cluster and deployed an application.
