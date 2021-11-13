# Interactive Tutorial 7.3  A Declarative Approach to Configuring And Deploying to Kubernetes 

## Overview
In this lab we want to do things declaratively by having a file that describes what we want the desired state to be and then we will let Kubernetes do all the hard work. This means that Kubernetes will figure out how to make that desired state work.
Having our state defined declaratively in files allows us to make changes to our deployments by just editing the file describing it. We can also track those changes through source control and make use of great tools for deploying applications at scale, like continuous integration and continuous deployment tools.

In this tutorial you will set up the Cloud Shell environment in the same way as you did in tutorial 7.1 and export the project_id, zone and region into Cloud Shell as defaults.

## Step 1. Create the Kubernetes cluster

1.1 Create a one-node Standard cluster named hello-cluster:
```	
gcloud container clusters create hello-cluster --num-nodes=1
```

It might take several minutes to finish creating the cluster.


1.2 Get authentication credentials for the cluster
After creating your cluster, you need to get authentication credentials to interact with the cluster:
```
gcloud container clusters get-credentials hello-cluster
```

This command configures kubectl to use the cluster you created.

1.3 Make a working dir and download the source files
```

git clone https://github.com/papa107/declarative-patterns.git
```

Using the Cloud Shell terminal navigate to the new directory tutorial_apps run LS here to list the files, you will see some subdirectories that hold the files you will need to create that microservice application in Kubernetes. Navigate to the tutorial_apps/io15/kubernetes directory. You will work from this directory for all exercises in the tutorial.

1.4 Open deployments/hello.yml in Cloud Shell Editor 

View the Code
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 3
  selector:
    matchLabels: 
      app: "hello"
  template:
    metadata:
      labels:
        app: hello
        track: stable
        version: 1.0.0
    spec:
      containers:
        - name: hello
          image: "askcarter/hello:1.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: "10Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1

```

This is the configuration file we will edit to deploy our app to kubernetes. 

* You are giving that deployment a name.
* You are telling Kubernetes the number of replicas of the pods that will be in this deployment.
* You are defining the container image 
* You have defined a liveness probe and a readiness probe 

The liveness and readiness probes are health check tools that Kubernetes provides to help ensure that your application is up and ready to receive traffic.

All of the information about what we want this deployment to look like is here in this file and this is what we mean by declaring our desired state. And if we wanted to make a change, like changing the image that we're going to be running in this deployment, we could do that in this file and we could track that change through source control.

1.5 Open services/frontend.yaml in Cloud Shell Editor 

```yaml
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  selector:
    app: "hello"
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80

```
In tutorial 7.1 you had to create a service to be able to connect to our app and this is what you are doing in this file, you are creating a Kubernetes service. 
This is creating another Kubernetes service that will serve connections to your application, which will be deployed in its own deployment.
If you consider the service.yml contents you see that the service definition is pretty simple. 
* It has a name for the service. 
* It has a selector which tells the service how to find the back ends that it is serving, and
* it has some port information that declares how to reach the application.

## 2.0 Deploy the hello microservices to Kubernetes cluster

```
kubectl apply -f deployments/hello.yaml -f services/hello.yaml
```

You can run several deployments at the same time by just adding another ‘-f’ and the deployment files name.

2.1 Deploy the Hello microservice to Kubernetes cluster
This application uses an Nginx server, which will act as your front end. But first we will need to create a service for it using the same template for the hello service. You give it a name, a selector and define the ports.

When you have finished viewing the code run the command:
```
kubectl apply -f deployments/frontend.yaml -f services/frontend.yaml
```
## 2.2 Examine the rest of the Frontend Configuration File
Note the Volumes being mounted in line 23. You should see that two external volumes are being mounted: ng-frontend-conf and tls-certs. Starting at line 28: you should find these volumes identified as being a secret (tls-certs) and a configMap (nginx-frontend-conf) volumes respectively.

 In Kubernetes and Docker containers we can think of a volume as an external directory, possibly with some data in it, which is accessible to the containers in a pod. Volumes are a convenient way to access data that is external to the container.

To use a volume, you need to specify the volumes to provide for the Pod in the frontend.yaml file in .spec.volumes (see line 28) and also declare where to mount those volumes into containers in .spec.containers[*].volumeMounts. (see line 23) 
A process in a container sees a filesystem view composed from their Docker image and any mounted volumes. The Docker image is at the root of the filesystem hierarchy with Volumes mounted at the specified paths within the image. Volumes can not mount onto other volumes or have hard links to other volumes. Each Container in the Pod's configuration must independently specify where to mount each volume.
Now let's see what these volumes are used for?

## 3.0 Configuring the Nginx Container
You now have a front-end consisting of a container running Nginx but there is some configuration information that you want to pass into your Nginx server and we can do this using a Kubernetes tool called ConfigMaps.

3.1 Create a configMaps for the nginx frontend.
A config map lets you map your configuration information to your application running in a Kubernetes deployment. The configuration information that is needed for your Nginx front end is located in nginx/frontend.conf.
So you can see here that this is a bunch of configuration information for my Nginx front end.
However this conf file is not in yaml format so we will need to create a ConfigMap that uses this as an input.
```
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf 
```

In this command Configmap is the type of Kubernetes object that you want to create. You have given it a name and provided a source file, which is nginx/frontend.conf.

3.2 Pass confidential information into the container using secrets.
If we wanted to run this application in production we would need to use SSL/TLS to enable HTTPS but to do that we need to pass some TLS certificates.

You don’t want to use a configMap for this config map because that information is passed in plain text. You need the certs to be passed encrypted and Kubernetes has a tool for this called Secrets.
```
kubectl create secret generic tls-certs --from-file tls/
```

We use the kubectl command to create a secret in a very similar way to creating a ConfigMaps, except they're for secret information.
To create a secret we provide a secrets type, in this case we have chosen the all-purpose generic type and we give it a name, tls-certs and we provide a source file location, tls/

## 4.0 Deploy the Auth microservice
The Auth module is the last piece of the microservice application. And of course, I'm going to need a service service/auth.yml and something for that service to serve, which will be a deployment one again and that's going to be deployment/auth.yml as well.
```
kubectl apply -f deployments/auth.yaml -f services/auth.yaml
```

That is the whole microservice architecture for the app deployed to kubernetes. 

## 5.0 Testing the App is deployed in Kubernetes

To test the app has deployed okay we can run kubectl get services, or just svc for short. In Kubernetes run:
```
kubectl get services
```
5.1  Get the External IP address
In this step you want to obtain the Kubernetes EXTERNAL_IP address so you need to get this information for the front end.
In this case Kubernetes has created an actual load balancer resource in Google Cloud, and that external load balancer resource has created an external IP for you so that anyone will actually be able to use it to reach the application that you created.

5.2 Export the EXTERNAL_IP
Copy and export that IP as a variable just to make the following commands a little bit easier.
```
Export EXTERNAL_IP
```
So now I can run a curl command to test to make sure that my application is running correctly.
And if you'll remember earlier, we incorporated some security capabilities into this application.

## 6.0 Test the App from the Public IP
Because we set up TLS and we set up some security measures. we're going to have to connect to this application using HTTPS.
To do this run:
```
Curl -k https://$EXTERNAL_IP
```

## 7.0 Examine the Health of the Cluster
In the step we will examine the health checks we configured in the yaml config file. 

In the yaml config we configured Liveness and a Readiness Probes so lets see if they are working as planned.

7.1 Examine the logs by running the command:
```
kubectl logs deployments/hello -c hello
```

7.2 Note the output from the logs then close using ctrl/c

**Congratulations!** You now know the application is up and running properly on Kubernetes.
