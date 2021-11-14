# Interactive Tutorial 8.1: Running Microservices in Google Kubernetes Engine (GKE)
## Overview
In this tutorial we will run microservices in Google Kubernetes Engine (GKE). Kubernetes is a platform to manage, host, scale, and deploy containers. Containers are a portable way of packaging and running code. They are well suited to the microservices pattern, where each microservice can run in its own container.
For this tutorial, we will deploy an existing monolithic application to a Google Kubernetes Engine cluster, then break it down into microservices!

What you'll learn
* How to break down a Monolith to Microservices
* How to create a Google Kubernetes Engine cluster
* How to create a Docker image
* How to deploy Docker images to Kubernetes

## Step 1: Set the Environment

Step 1.1 Open Cloud Shell and some environment variables as default, this will make entering commands easier
as you will not have to keep entering your Project_ID, Zone and Region.
```
echo $GOOGLE_CLOUD_PROJECT
```
Command output
<PROJECT_ID>

Step 2. Finally, set the default zone and project configuration, use your own.
```
gcloud config set compute/zone asia-south2-a
```
You can choose your own or use the US defaults as this isn’t going to be testing the latency of our app in any way.

## 3. Clone Source Repository
We use an existing monolithic application of an imaginary ecommerce website, with a simple welcome page, a products page and an order history page. We will just need to clone the source from our git repo, so we can focus on breaking it down into microservices and deploying to Google Kubernetes Engine (GKE).
Run the following commands to clone the git repo to your Cloud Shell instance and change to the appropriate directory. We will also install the NodeJS dependencies so we can test our monolith before deploying. It may take a few minutes for this script to run.
```
cd ~
git clone https://github.com/googlecodelabs/monolith-to-microservices.git
cd ~/monolith-to-microservices
./setup.sh
```
This will clone our Github repo, change to the directory and install the dependencies needed to run our application locally. It may take a few minutes for this script to run.

## 4. Create a GKE Cluster
Step 4.1 Before we do anything else we have to enable the apis that we will use in this tutorial. In order to ensure the proper API's are enabled we Run the following command:
```
gcloud services enable container.googleapis.com
```
This command enables the containers api so we can use Google Kubernetes Engine.

4.2  Next we need to build a GKE cluster in which to deploy our monolith and eventually our microservices. Run the command below to create a GKE cluster named book-website with 3 nodes.
```
gcloud container clusters create book-website  --num-nodes 3
```
It can seem like ever for the cluster to be created but be patient and when the command has completed, run the following command: and see the cluster's three worker VM instances:
```
gcloud compute instances list
```

You can also view your Kubernetes cluster and related information in the Google Cloud console. Click the menu button in the top left, scroll down to Kubernetes Engine and click Clusters. You should see your cluster listed and named book-website.
Note:
If you have been following along and you have created a cluster with the gcloud container clusters create command listed above, the next step is not necessary.

Step 4.3 If you have created your cluster using the GCP console or are using an existing Google Kubernetes Engine cluster, you will need to run the following command to retrieve cluster credentials and configure kubectl command-line tool with them:
```
gcloud container clusters get-credentials book-website
```
## Step 5. Deploy and Access the Monolith
Step 5.1 We are first going to deploy our monolith website app as we need to get a monolith application up and running before we can start to demonstrate how to break it down into microservices architecture. Run the following script to deploy a monolith application to our GKE cluster:
```
cd ~/monolith-to-microservices
./deploy-monolith.sh
```

Step 5.2 To find the external IP address for our monolith application, run the following command.
```
kubectl get service monolith
```
You should see output similar to the following:
NAME         CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
monolith     10.3.251.122    203.0.113.0     80:30877/TCP     3d

NOTE: Behind the scenes an external load balancer and IP are being provisioned for this so it will take a few moments. If your output lists the external IP as <pending> give it a few minutes and try again.
Once you've determined the external IP address for your monolith, copy the IP address. Point your browser to this URL (such as http://203.0.113.0) to check if your monolith is accessible.
Take a note of the IP or benchmark the link in your browser  as you will continue to use it throughout this tutorial. Nonetheless it saves you running this same command again.
Rebuild Monolith Config Files

Step 5.3 Now we have to rebuild the monolith app as we have changes to be applied.
```
npm run build:monolith
```
## Step 6 Create Docker Container with Google Cloud Build

In order to deploy our app to our Kubernetes cluster we have to first put it into a container.
```
cd ~/monolith-to-microservices/monolith
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 .
```
When the cloud build process is complete and it has been pushed to the Container Registry we are ready to deploy our app. 

To deploy container from the Container Registry to KGE we use the following command:
```
kubectl set image deployment/monolith monolith=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0
```
You can verify your application is now hitting the new Orders microservice by going to the monolith application in your browser and navigating to the Orders page. 
All the order ID's should end in a suffix -MICROSERVICE :

You should see the welcome page for the monolithic website. The welcome page is a static page that will be served up by the Frontend microservice later on. You now have your monolith fully running on Kubernetes!

## Step 7 Migrate Products to Microservice

Step 7.1 Create New Products Microservice
You will continue to break out our services by migrating the Products service next. You will follow the same process as the previous step. Run the following commands to build a Docker container, deploy your container and expose it to via a Kubernetes service.

7.2 Create Docker Container with Google Cloud Build
```
cd ~/monolith-to-microservices/microservices/src/products
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/products:1.0.0 .
```

7.3 Deploy Container to GKE
```
kubectl create deployment products --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/products:1.0.0
```

7.4 Expose GKE Container
```
kubectl expose deployment products --type=LoadBalancer --port 80 --target-port 8082
```

7.5 Find the public IP of our Products services the same way we did for our Orders service with the following command:
```
kubectl get service products
```

Output:
NAME         CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
products     10.3.251.122    203.0.113.0     80:30877/TCP     3d

Save the IP address for the next step when we reconfigure our monolith to point to our new Products microservice.

7.6 Reconfigure Monolith
Use the nano editor to replace the local URL with the IP address of our new Products microservices:
```
cd ~/monolith-to-microservices/react-app
nano .env.monolith
```

When the editor opens, your file should look like this:
```  
REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders

REACT_APP_PRODUCTS_URL=/service/products
```
Replace the REACT_APP_PRODUCTS_URL to the new format while replacing the Product microservice IP address so it matches below:
```
REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders

REACT_APP_PRODUCTS_URL=http://<PRODUCTS_IP_ADDRESS>/api/products
```

Press CTRL+O, press ENTER, then CTRL+X to save the file in the nano editor.

You can test your new microservice by navigating to the URL you just set in this file. The webpage should return a JSON response from our Products microservice.
Next, we will need to rebuild our monolith frontend and repeat the build process to build the container for the monolith and redeploy to our GKE cluster. Run the following commands complete these steps:

7.7 Rebuild Monolith Config Files
```
npm run build:monolith
```

7.8 Create Docker Container with Google Cloud Build
```
cd ~/monolith-to-microservices/monolith
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:3.0.0 .
```

7.9 Deploy Container to GKE
```
kubectl set image deployment/monolith monolith=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:3.0.0
```
You can verify your application is now hitting the new Products microservice by going to the monolith application in your browser and navigating to the Products page. All the product names should be prefixed by MS- as shown below:

## 8. Migrate Frontend to Microservice
The last step in the migration process is to move the Frontend code to a microservice and shut down the monolith! After this step is completed, we will have successfully migrated our monolith to a microservices architecture!

8.1 Create New Frontend Microservice
You will follow the same procedure as the last two steps but this time you will create a new frontend microservice.
However, when we rebuilt our monolith we updated our config to point to our monolith, but now we need to use the same config for our Frontend microservice. So we will have to copy the config across so the service traders seamlessly. To do this you will Run the following commands to copy the monolith config files to the Frontend microservice codebase:
```
cd ~/monolith-to-microservices/react-app
cp .env.monolith .env
npm run build
```
Once that is completed, You are back on track so you can follow the same process as the previous steps. Run the following commands to build a Docker container, deploy your container and expose it to via a Kubernetes service.

8.2 Create Docker Container with Google Cloud Build
```
cd ~/monolith-to-microservices/microservices/src/frontend
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/frontend:1.0.0 .
```

8.3 Deploy Container to GKE
```
kubectl create deployment frontend --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/frontend:1.0.0
```

8.4 Expose GKE Container
```
kubectl expose deployment frontend --type=LoadBalancer --port 80 --target-port 8080
```

Delete The Monolith

8.5 Now that all our services are running as microservices, we can delete our monolith application! Note, in an actual migration, this would also entail DNS changes, etc to get our existing domain names to point to the new frontend microservices for our application. Run the following commands to delete our monolith:
```
kubectl delete deployment monolith
kubectl delete service monolith
```
## 9. Test Your Work
To verify everything is working, your old IP address from your monolith service should not work and your new IP address from your frontend service should host the new application. To see a list of all the services and IP addresses, use the following command:
```
kubectl get services
```

Your output should look similar to the following:
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
frontend     LoadBalancer   10.39.246.135   35.227.21.154    80:32663/TCP   12m
kubernetes   ClusterIP      10.39.240.1     <none>           443/TCP        18d
orders       LoadBalancer   10.39.243.42    35.243.173.255   80:32714/TCP   31m
products     LoadBalancer   10.39.250.16    35.243.180.23    80:32335/TCP   21m

Once you've determined the external IP address for your Frontend microservice, copy the IP address. Point your browser to this URL (such as http://203.0.113.0) to check if your frontend is accessible. 

## 10. Migration with node pools
Instead of upgrading the “active” node pool as you would with a rolling update, you can create a fresh node pool, wait for all the nodes to be running, and then migrate workloads over one node at a time.
For example, if a Kubernetes cluster has three VMs and you can see the nodes with the following command:
```  
kubectl get nodes
```
NAME                                        STATUS  AGE
gke-cluster-1-default-pool-7d6b79ce-0s6z    Ready   3h
gke-cluster-1-default-pool-7d6b79ce-9kkm    Ready   3h
gke-cluster-1-default-pool-7d6b79ce-j6ch    Ready   3h

10.1 Then you would need to create a new node pool. To create the new node pool with the name “pool-two”, you would run the following command: 
```
gcloud container node-pools create pool-two
```

You can also use the Console GUI to create a new node pool if you prefer.

Now if you check the nodes, you will notice there are three more nodes with the new pool name:

NAME                                        STATUS  AGE
gke-cluster-1-pool-two-9ca78aa9–5gmk        Ready   1m
gke-cluster-1-pool-two-9ca78aa9–5w6w        Ready   1m
gke-cluster-1-pool-two-9ca78aa9-v88c        Ready   1m
gke-cluster-1-default-pool-7d6b79ce-0s6z    Ready   3h
gke-cluster-1-default-pool-7d6b79ce-9kkm    Ready   3h
gke-cluster-1-default-pool-7d6b79ce-j6ch    Ready   3h

However, the pods are still on the old nodes! Let’s move them over.

10.2 Drain the old pool
The first thing we have to do is to drain the old pool so that there is a graceful transition when we move work to the new node pool. To do this we will move over one node at a time in a rolling fashion.
First, we will cordon off each of the old nodes. This will prevent new pods from being scheduled onto them to do this we use the command:
```
kubectl cordon <node_name>
```

10.3 Once all the old nodes are cordoned, pods can only be scheduled onto the new node pool. This means you can start to remove pods from the old node pool, and Kubernetes automatically schedules them on the new node pool.

Warning: Make sure your pods are managed by a ReplicaSet, Deployment, StatefulSet, or something similar. Standalone pods won’t be rescheduled!

Run the following command to drain each node. This deletes all the pods on that node.
```
kubectl drain <node_name> --force
```

After you drain a node, make sure the new pods are up and running before moving on to the next one.

If you have any issues during the migration, uncordon the old pool and then cordon and drain the new pool. The pods get rescheduled back to the old pool.

10.4 Delete the old pool
Once all the pods are safely rescheduled, it is time to delete the old pool.

Replace “default-pool” with the pool you want to delete.
```
gcloud container node-pools delete default-pool
```

You have just successfully updated all your nodes!

**Congratulations!** You have completed the Tutorial.
