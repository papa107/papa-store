# Interactive Tutorial 2.2  

In this interactive turorial you will be **Creating a Custom Cloud Shell Docker Image**

Cloud Shell supports the customization of the default Cloud Shell experience by creating a Docker image that functions as a custom Cloud Shell environment with your specified additional packages and custom configurations. The only caveats are that the custom Docker image must be based on the base Cloud Shell image and hosted in GCP’s Container Registry.

## Step 1. Creating the Docker Image
Create your own custom Cloud Shell Docker image by following these instructions:
In a Cloud Shell run the following command to create a boilerplate custom image in a repository hosted by Cloud Source Repositories:
```
cloudshell env create-custom-image NEW_REPO_NAME
```
2. Open your new Dockerfile by following the instructions printed to your command line:
```
cd $HOME/NEW_REPO_NAME && cloudshell edit Dockerfile
```

3. (OPTIONAL) Add any additional packages you want made available in your Cloud Shell experience below the first line. For example:
```
FROM gcr.io/cloudshell-images/cloudshell:latest
RUN apt-get -y install *name of package*
```
The first line in your Dockerfile, ‘FROM gcr.io/cloudshell-images/cloudshell:latest’, references the base Cloud Shell image and should not be removed.

## Step 2. Build your image 

Build your image locally by running:
```
cloudshell env build-local
```

Test your image locally and verify that your installed packages are present by running:
```
cloudshell env run
```

To exit testing, **type: exit**


Commit your code changes locally:
```
git commit -a -m "Initial custom environment check-in."
```

Push your code changes to Cloud Source Repositories:
```
git push origin master
```

Finally, push your custom image to Container Registry:
```
cloudshell env push
```

**Congratulations!** You have just created your own code repository, cloned your shell image and created a docker container and pushed it into Google Registry. 
