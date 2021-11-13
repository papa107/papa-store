# Creating a Custom Shell Docker image

## Step 1. To create your own custom Docker image follow these steps:

Step 1.1 In a Cloud Shell terminal tab, run the following command to create a boilerplate custom image in a repository hosted by Cloud Source Repositories:
```
cloudshell env create-custom-image NEW_REPO_NAME
```

Step 1.2 Open a new empty Dockerfile using the following command:
```
cd $HOME/NEW_REPO_NAME && cloudshell edit Dockerfile
```

This convoluted command simply changes the working directory to our new source repository and opens a new dockerfile.

Step 1.3 Now you can edit the Dockerfile and add any additional packages you want made available in your Cloud Shell container. For example: The following snippet  required Codeserver and Higo to be installed at startup.
```
FROM gcr.io/cloudshell-images/cloudshell:latest
 
ENV CODESERVER_VERSION="2.1665-vsc1.39.2"
ENV HUGO_VERSION="0.88.1"
```

Note: The first line in your Dockerfile, FROM gcr.io/cloudshell-images/cloudshell:latest, references the base Cloud Shell image and should not be removed.

## Step 2.  Once you have finished editing you can Build your image locally by running:
```
cloudshell env build-local
```

Step 2.1 Once the build completes you can proceed to Test your image locally and verify that your installed packages are present by running:
```
cloudshell env run
```

Step 2.2 To exit testing, run:
```
exit
```

## Step 3. You now want to Commit your code changes locally by checking-in your code:
```
git commit -a -m "Initial custom environment check-in."
```

Step 3.1 Push your code changes to Cloud Source Repositories:
```
git push origin master
```

Step 3.2 Finally, push your custom image to Container Registry:
```
cloudshell env push
```

## Step 4. Using the Console Browser open Artifact Repository and click on Container Registry to view your new customized Container.

Note the menu options in Container Registry allows us to deploy to Cloud Run, GKE and GCE.

**Congratulations!** You have just created your own Shell Docker image
