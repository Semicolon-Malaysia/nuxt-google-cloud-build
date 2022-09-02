

# Google Cloud Build Tutorial for Nuxt.js


## Application Configuration

In the nuxt configuration file, set the following config:
* target: "server" **(required)**
* ssr: "true" **(if using ssr, can be false too)**

In the server configuration, make sure that the app is hosted at "0.0.0.0" and port 8080.



## Cloudbuild.json

Add a cloudbuild.json to the root path your application.
Set the service name as your Google CLoud service name.
The image name can be anything.

````json
{
  "steps": [
    {
      "name": "node:12",
      "entrypoint": "npm",
      "args": ["run", "create-env"],
      "env": [
        "APP_NAME=${_APP_NAME}",
        "APP_PORT=${_APP_PORT}",
        "APP_URL=${_APP_URL}",
        "APP_HOST=${_APP_HOST}",
        "APP_META_DESCRIPTION=${_APP_META_DESCRIPTION}",
        "GA_MEASUREMENT_ID=${_GA_MEASUREMENT_ID}",
        "GA_STREAM_ID=${_GA_STREAM_ID}"
      ]
    },
    {
      "name": "gcr.io/cloud-builders/docker",
      "args": ["build", "-t", "gcr.io/{{service_name}}/{{image_name}}", "."]
    },
    {
      "name": "gcr.io/cloud-builders/docker",
      "args": ["push", "gcr.io/{{service_name}}/{{image_name}}"]
    },
    {
      "name": "gcr.io/cloud-builders/gcloud",
      "args": [
        "run",
        "deploy",
        "{{service_name}}",
        "--image",
        "gcr.io/{{service_name}}/{{image_name}}",
        "--region",
        "asia-southeast1",
        "--platform",
        "managed"
      ]
    }
  ],
  "options": { "logging": "CLOUD_LOGGING_ONLY" },
  "images": ["gcr.io/{{service_name}}/{{image_name}}"],
  "timeout": "1200s"
}

````
 ## Dockerfile
Add a Dockerfile to the root path of your application.

    FROM node:12
    
    WORKDIR /src/app
    
    COPY package*.json ./
    RUN npm install
    
    COPY . .
    
    #RUN mv .env.production .env
    RUN npm run build
    
    RUN npm cache clean --force
    
    EXPOSE 8080
    CMD [ "npm", "start" ]

## Setup Google Cloud Service
### Google Cloud Account
1. Create a Google Cloud account if you dont already have one.
2. Enable Cloud Build, Cloud Run, Billing, and IAM Authentication APIs.
3. Create a service account and grant Cloud Run Invoker, Cloud Build Service Account access.

### Cloud Build
1. Create a new Cloud Build Trigger.
2. Connect to repository and follow the instructions accordingly.
3. For the Container configuration, make sure that the port is set to 8080.
4. Add all the env variables neede for your app in the container variable section.
5. Create and run the trigger!

### Cloud Run
1. When the app has finished building, a Cloud Run service will be created automatically.
2. To make the app public, allUsers must be given access to the service. Change the container authentication accordingly.
3. To connect the container to your website, follow the 'add domain' instructions on the dashboard.
