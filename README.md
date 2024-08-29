# FlaskWebGUI Project

## Overview

**FlaskWebGUI** is a simple Python web application that uses Flask for the backend and FlaskWebGUI to provide a desktop-like window for the web app. The application is containerized with Docker and is designed to be deployed on Kubernetes using Helm. This README will guide you through the process of setting up a Jenkins pipeline to build, push, and deploy the application.

## Project Structure

```
.
├── Dockerfile
├── helm/
│   ├── charts/
│   ├── Chart.yaml
│   ├── my-values.yaml
│   ├── templates/
│   │   ├── deployment.yaml
│   │   ├── _helpers.tpl
│   │   ├── hpa.yaml
│   │   ├── ingress.yaml
│   │   ├── NOTES.txt
│   │   ├── serviceaccount.yaml
│   │   ├── service.yaml
│   │   └── tests/
│   │       └── test-connection.yaml
│   └── values.yaml
├── Jenkinsfile
├── main.py
├── requirements.txt
└── templates/
    ├── index.html
    └── some_page.html
```

- **`main.py`**: The main Flask application file.
- **`templates/`**: HTML files for rendering the web pages.
- **`Dockerfile`**: Docker configuration to containerize the application.
- **`helm/`**: Helm chart for deploying the application on Kubernetes.
- **`Jenkinsfile`**: Jenkins pipeline script for building, pushing, and deploying the Docker image.

## Prerequisites

Before setting up the Jenkins pipeline, ensure you have the following:

- Jenkins installed and configured with Kubernetes.
- A Docker Hub account with access tokens or login credentials.
- A Kubernetes cluster accessible from Jenkins (for example, a KIND or a managed cluster).

## Setting Up Jenkins Pipeline

### Step 1: Create a Jenkins Pipeline

1. **Log in to Jenkins**: Access your Jenkins dashboard.

2. **Create a New Pipeline**:
   - Go to "New Item" in Jenkins.
   - Enter a name for your pipeline (e.g., `FlaskWebGUI-Pipeline`).
   - Select "Pipeline" as the project type and click "OK".

3. **Configure the Pipeline**:
   - In the pipeline configuration, go to the "Pipeline" section.
   - Choose "Pipeline script from SCM".
   - Set "SCM" to "Git" and enter your repository URL:
     ```plaintext
     https://github.com/DanyalTaghipor/webgui
     ```
   - Set the branch to "main".

4. **Add Docker Hub Credentials**:
   - In Jenkins, go to "Manage Jenkins" -> "Manage Credentials".
   - Add a new credential of type "Username with password".
   - Use your Docker Hub username and password/token.
   - Set the ID to `docker`.

### Step 2: Review the Jenkinsfile

The `Jenkinsfile` in your project repository is pre-configured with the necessary steps to build, push, and deploy the Docker image. Here's a brief explanation of the stages:

- **Checkout**: Clones the project repository from GitHub.
- **Build Docker Image**: Builds a Docker image using the `Dockerfile` in the project.
- **Push Docker Image**: Pushes the built image to Docker Hub.
- **Deploy to Kubernetes**: Uses Helm to deploy the application to the Kubernetes cluster.

### Step 3: Set Up Environment Variables

In your Jenkins pipeline, make sure the following environment variables are correctly configured:

- `DOCKER_IMAGE`: This should be set to your Docker Hub repository, e.g., `danytgh/webgui`.

The `Jenkinsfile` will automatically pull the repository, build the image, and push it to Docker Hub. It will then deploy the application to your Kubernetes cluster using Helm.

### Step 4: Run the Pipeline

- After setting up the pipeline, click "Build Now" to run the pipeline.
- Monitor the build process in the Jenkins console output.
- If the pipeline succeeds, your Flask web application will be available on the Kubernetes cluster.

## Accessing the Application

Once deployed, the application will be accessible through the configured Ingress or service. Check your Kubernetes cluster’s configuration to find the appropriate URL or IP address.

## Cleanup

To remove the deployed application and clean up resources:

1. **Uninstall the Helm Release**:

   ```sh
   helm uninstall webgui
   ```
