# Interactive Tutorial 5.2 
In this interactive tutorial you will build and deploy a Container in Cloud Run. In this tutorial we will be learning hands-on how to apply what we have learned so far to build and deploy an application container to Cloud Run.
Cloud Run is a managed compute platform that enables you to run stateless containers that are invocable via HTTP requests. Cloud Run is serverless: it abstracts away all infrastructure management, so you can focus on what matters most — building great applications.
It is built from Knative, letting you choose to run your containers either fully managed with Cloud Run, or in your Google Kubernetes Engine cluster with Cloud Run on GKE.
The goal of this tutorial is for you to build a container image and deploy it to Cloud Run.
## Step 1. Setup and requirements
Set up your Project and environment.
**Sign in to Cloud Console** and **create a new project** or reuse an existing one. 
Next, you'll need to enable billing in Cloud Console in order to use Google Cloud resources.
Step 1.1 **Open Cloud Shell**
Step 1.2  **Enable the APIs**
From Cloud Shell, enable the Cloud Build and Cloud Run APIs:
```
gcloud services enable cloudbuild.googleapis.com run.googleapis.com
```
This should produce a successful message similar to this one:
```
Operation "operations/acf.cc11852d-40af-47ad-9d59-477a12847c9e" finished successfully.
```
## Step 2. Write the sample application
In this step you'll build a simple Flask-based Python application responding to HTTP requests. 
Step 2.1 To build your application, use Cloud Shell to create a new directory named helloworld-python and change directory into it:
```
mkdir ~/helloworld-python
cd ~/helloworld-python
```
Step 2.2 Using the Cloud Shell web editor **click on the "Open Editor" ** (pen-shaped icon), **create a file named app.py** and paste the following code into it:
```
from flask import Flask, request

app = Flask(__name__)

@app.route("/", methods=["GET"])
def hello():
    """ Return a friendly HTTP greeting. """
    who = request.args.get("who", "World")
    return f"Hello {who}!\n"
if __name__ == "__main__":
    # Used when running locally only. When deploying to Cloud Run,
    # a webserver process such as Gunicorn will serve the app.
    app.run(host="localhost", port=8080, debug=True)
```
This code creates a basic web server responding to HTTP GET requests with a friendly message. Your app is now ready to be containerized, tested, and uploaded to Container Registry.
## Step 3. Containerize your app and upload it to Container Registry
Step 3.1 To containerize the sample app, **create a new file** named Dockerfile in the same directory as the source files, and copy the following content:
```
# Use an official lightweight Python image.
# https://hub.docker.com/_/python
FROM python:3.9-slim

# Install production dependencies.
RUN pip install Flask gunicorn

# Copy local code to the container image.
WORKDIR /app
COPY . .

# Service must listen to $PORT environment variable.
# This default value facilitates local development.
ENV PORT 8080

# Run the web service on container startup. Here we use the gunicorn
# webserver, with one worker process and 8 threads.
# For environments with multiple CPU cores, increase the number of workers
# to be equal to the cores available.
CMD exec gunicorn --bind 0.0.0.0:$PORT --workers 1 --threads 8 --timeout 0 app:app
```
You can define any container image conforming to the Cloud Run Container Contract.
Step 3.2 Define the PROJECT_ID and DOCKER_IMG environment variables which will be used throughout the next steps and make sure they have the correct values:
```
PROJECT_ID=$(gcloud config get-value project)
DOCKER_IMG="gcr.io/$PROJECT_ID/helloworld-python"
echo $PROJECT_ID
echo $DOCKER_IMG
```
Step 3.3 Now, build your container image using Cloud Build, by running the following command from the directory containing the Dockerfile:
```
gcloud builds submit --tag $DOCKER_IMG
```
Cloud Build is a service that executes your builds on GCP. It executes a series of build steps, where each build step is run in a Docker container to produce your application container (or other artifacts) and push it to Cloud Registry, all in one command.
Once pushed to the registry, you will see a SUCCESS message containing the image name. 
The image is stored in the Artifact Registry and can be re-used if desired.
Step 3.4 You can list all the container images associated with your current project using this command:
```
gcloud container images list
```
Step 3.5 Before deploying, run and test the application locally from Cloud Shell, you can start it using these standard docker commands:
```
docker pull $DOCKER_IMG
docker run -p 8080:8080 $DOCKER_IMG
```

Step 3.6 If the docker command cannot pull the remote container image then try running this:
```
gcloud auth configure-docker
```
In the Cloud Shell window, **click on the "Web preview" icon and select "Preview on port 8080":**
This should open a browser window showing the "Hello World!" message. You can also simply use from another Cloud Shell session: 
```
curl localhost:8080
```
 When you're done, you can stop your docker run command with ctrl+c.
## Step 4. Deploy to Cloud Run
Cloud Run is regional, which means the infrastructure that runs your Cloud Run services is located in a specific region and is managed by Google to be redundantly available across all the zones within that region. Define the region you'll use for your deployment, for example:
REGION="asia-south2"
Step 4.1 Deploy your containerized application to Cloud Run with the following command:
```
gcloud run deploy helloworld-python \
  --image $DOCKER_IMG \
  --platform managed \
  --region $REGION \
  --allow-unauthenticated
```
Note:
You can also define a default region with:
```
 gcloud config set run/region $REGION.
```
The --allow-unauthenticated option makes the service publicly available. To avoid unauthenticated requests, use --no-allow-unauthenticated instead.
To check all options, use gcloud run deploy --help.
Then wait a few moments until the deployment is complete. On success, the command line displays the service URL:
```
Deploying container to Cloud Run service [helloworld-python] in project [PROJECT_ID...
✓ Deploying new service... Done.                                   
  ✓ Creating Revision... Revision deployment finished. Waiting for health check...
  ✓ Routing traffic...
  ✓ Setting IAM Policy...
Done.
Service [helloworld-python] revision [helloworld-python-...] has been deployed
and is serving 100 percent of traffic.
Service URL: https://helloworld-python-....a.run.app
```
You can also retrieve your service URL:
```
SERVICE_URL=$( \
  gcloud run services describe helloworld-python \
  --platform managed \
  --region $REGION \
  --format "value(status.url)" \
)
echo $SERVICE_URL
```
This should display something like:
https://helloworld-python-....a.run.app

Step 4.2 You can now visit your deployed container by opening the service URL in a web browser:
You can also call your service from Cloud Shell:
```
curl $SERVICE_URL?who=me
```
**Congratulations!** 
