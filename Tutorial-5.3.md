# Interactive Tutorial 5.3 Running A Google Translate API on Cloud Run

## Step 1. Enable Translation API

In this section, you'll learn how to enable Google APIs in general. For our sample app, you'll enable the Cloud Translation API, the Cloud Run API, and Cloud Artifact Registry API.
Introduction
Regardless of which Google API you want to use in your application, they must be enabled. The following example shows two ways to enable the Cloud Translate API.  After you learn how to enable one Cloud API, you'll be able to enable other APIs because the process consistently applies to all the other APIs in the catalogue.

**Option 1: From the Cloud Console**
You can also enable the Translate API in the console under API and Services, From the Cloud Console, and then go to API Manager and select Library.

Alternatively you could click on the ENABLE APIS and SERVICES link either way they will take you to the same location. If you wanted to enable the Cloud Translate API, start by entering "Translate" in the search bar, and anything that matches what you've entered so far will appear:

While its monthly quota isn't listed in the overall "Always Free" tier summary page, the Translation API's pricing page states all users get a fixed amount of translated characters monthly. You shouldn't incur any charges from the API if you stay below that threshold. If there are any other Google Cloud related charges, they will be discussed at the end in the "Clean up" section.
Click on the button to enable the API

**Option 2: From Cloud Shell or your command-line interface**
While enabling APIs from the Cloud Console is easy it can be a bit cumbersome to navigate if you are to be enabling several APIs at once. Also developers are notoriously  keen on sticking to the shell command line. That might be fine for experienced developers but for beginners using the command line to activate the APIs it is not ideal as you will need to look up an API's "service name." 
The API’s Service-name looks like a URL: SERVICE_NAME.googleapis.com and you can find these in the Supported products chart, or you can query for them using the Google Discovery API.
For example, this command enables the Cloud Translate API:
```
gcloud services enable translate.googleapis.com
```
 
Step 1.1 Enable all 4 APIs you require for the tutorial. Type in the Cloud Terminal:
```
gcloud services enable translate.googleapis.com run.googleapis.com cloudfunctions.googleapis.com appengine.googleapis.com
```
Note: If this command results in errors, make sure that the current Project ID matches your codelab Project ID. Use the following command to find the current Project ID being used by Cloud Shell:
gcloud config list

If the Project ID is not correct, issue the following command to specify the correct Project ID:
gcloud config set project <PROJECT_ID>

Replace <PROJECT_ID> with the correct Project ID.

Step 1.2 Copy the files from the Github Repository
```
git clone https://github.com/papa107/cloud-translation-serverless-python.git
```
Step 1.3 Open Cloud Editor and Navigate to Dir /cloud-translation-serverless-python

## Step 2. Viewing the Code
In this tutorial the app is a simple Google Translate derivative that prompts users to enter text in English and receive the equivalent translation of that text in Spanish. 
Step 1. Open the Shell Editor and highlight the folder  ‘Cloud-translation-serverless-python’ in File click on Open Workspace, and Select  ‘Cloud-translation-serverless-python’. This will readjust your workspace to remove all the clutter.
Step 2. Open the python main.py file so we can view its contents and explore how the python code works. The code will look like what is shown below, with the commented lines about licensing, etc cropped. 
```
from flask import Flask, render_template, request
import google.auth
from google.cloud import translate
 
app = Flask(__name__)
_, PROJECT_ID = google.auth.default()
TRANSLATE = translate.TranslationServiceClient()
PARENT = 'projects/{}'.format(PROJECT_ID)
SOURCE, TARGET = ('en', 'English'), ('es', 'Spanish')
 
 
@app.route('/', methods=['GET', 'POST'])
def translate(gcf_request=None):
    """
    main handler - show form and possibly previous translation
    """
 
    # Flask Request object passed in for Cloud Functions
    # (use gcf_request for GCF but flask.request otherwise)
    local_request = gcf_request if gcf_request else request
 
    # reset all variables (GET)
    text = translated = None
 
    # form submission and if there is data to process (POST)
    if local_request.method == 'POST':
        text = local_request.form['text'].strip()
        if text:
            data = {
                'contents': [text],
                'parent': PARENT,
                'target_language_code': TARGET[0],
            }
            # handle older call for backwards-compatibility
            try:
                rsp = TRANSLATE.translate_text(request=data)
            except TypeError:
                rsp = TRANSLATE.translate_text(**data)
            translated = rsp.translations[0].translated_text
 
    # create context & render template
    context = {
        'orig':  {'text': text, 'lc': SOURCE},
        'trans': {'text': translated, 'lc': TARGET},
    }
    return render_template('index.html', **context)
 
 
if __name__ == '__main__':
    import os
    app.run(debug=True, threaded=True, host='0.0.0.0',
            port=int(os.environ.get('PORT', 8080)))
 ```
 
The imports bring in Flask functionality, the google.auth module, and the Cloud Translation API client library.
 
The global variables represent the Flask app, the Cloud project ID, the Translation API client, the parent "location path" for Translation API calls, and the source and target languages. In this case, it's English (en) and Spanish (es), but feel free to change these values to other language codes supported by the Cloud Translation API.
 
The large ‘if’ block at the bottom is used in the tutorial for running this app locally—it utilizes the Flask development server to serve our app. This section is also here for the Cloud Run deployment tutorials in case the web server isn't bundled into the container. You are asked to enable bundling the server in the container, but in case you overlook this, the app code falls back to using the Flask development server. (It is not an issue with App Engine or Cloud Functions because those are sourced-based platforms, meaning Google Cloud provides and runs a default web server.)

## Step 3. Cloud Run (Python 3 via Cloud Buildpacks)

Step 3.1 Navigate to the Dir: /cloud-translation-serverless-python (id you are not already there) and  DELETE the files Dockerfile, credentials.json, and lib

Step 3.2 Open Requirments.txt and make the following changes:
```
gunicorn>=19.10.0
flask>=1.1.2
google-cloud-translate>=2.0.1
```
Step 3.3 Open Procfile and make the following changes
```
#web: python main.py
web: gunicorn -b :$PORT -w 2 main:app
```
 
## Step 4. Deploy your translation service to Cloud Run 

Step 4.1 Run the following command:
```
gcloud run deploy translate --source . --allow-unauthenticated --platform managed
```
 
Remember with Buildpacks for Cloud Run there is no need to write Yaml build files or Dockerfiles
The output should look as follows, and provide some prompts for next steps:
Now that your app is available globally around the world, you should be able to reach it at the URL containing your project ID as shown in the deployment output: Service URL: 
## Step 4. Open Cloud Run to verify the service is deployed and active and note the URL.

Translate something to see it work!
