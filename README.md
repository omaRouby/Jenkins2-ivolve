# README.md

## Jenkins Pipeline to Build and Deploy a Python Application to OpenShift using Shared Library

This README provides a detailed guide on setting up a Jenkins pipeline to build a Python application, containerize it using Docker, push it to DockerHub, and deploy it to an OpenShift cluster. The pipeline will use a shared Jenkins library for reusability and maintainability.

### Prerequisites

1. **Jenkins Server**: Jenkins should be installed and running.
2. **DockerHub Account**: A DockerHub account to push the Docker image.
3. **OpenShift Cluster**: An OpenShift cluster to deploy the application.
4. **Python Application**: A Python application with a `Dockerfile` (Provided in this Repository) .
5. **Jenkins Shared Library**: A Jenkins shared library containing reusable pipeline code.

### Steps

#### Step 1: Configure DockerHub Credentials in Jenkins

1. **Add DockerHub Credentials**:
    - Go to Jenkins Dashboard > Manage Jenkins > Manage Credentials.
    - Add a new credential with your DockerHub username and password.
![](https://github.com/omaRouby/Jenkins2-ivolve/blob/main/pictures/dockerhub-cred.png)
#### Step 2: Set Up OpenShift Credentials in Jenkins

1. **Add OpenShift Credentials**:
    - Go to Jenkins Dashboard > Manage Jenkins > Manage Credentials.
    - Add a new credential with your OpenShift SA token.
![](https://github.com/omaRouby/Jenkins2-ivolve/blob/main/pictures/sa-token.png)
#### Step 3: Create Jenkins Shared Library

1. **Set Up Shared Library Repository**:
    - Create a new Git repository for your shared library if you don't have one.

2. **Structure Your Shared Library**:
    - Follow the standard structure for a Jenkins shared library. Create the necessary directories and files.

3. **Example Shared Library**:
    - Ensure your shared library includes the necessary scripts for building, pushing, and deploying the application.

#### Step 4: Configure Shared Library in Jenkins

1. **Add Shared Library**:
    - Go to Jenkins Dashboard > Manage Jenkins > Configure System.
    - Scroll down to the "Global Pipeline Libraries" section.
    - Add a new library with the name of your shared library, the default version ( `main`), and the repository URL.
    - The Shard Library Used for this Project https://github.com/omaRouby/share-library-python.git

#### Step 5: Create Jenkins Pipeline Job

1. **Create a New Pipeline Job**:
    - Go to Jenkins Dashboard > New Item.
    - Enter a name for your job and select "Pipeline".

2. **Configure Pipeline Script**:
    - In the pipeline configuration, select "Pipeline script from SCM".
    - Choose "Git" and enter the repository URL and branch for your Python application repository.
    - In the `Jenkinsfile` section, use the shared library:

  ```groovy
@Library('shared-library-python') _

pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "omarrouby/python-appx"
        DOCKERHUB_CREDENTIALS_ID = "dockerhub"
        OPENSHIFT_PROJECT = "omarrouby"
        OPENSHIFT_CREDENTIALS_ID = "sa-token"
        CLUSTER_URL = "https://api.ocp-training.ivolve-test.com:6443"
    }

    stages {
        stage('Build and Push Docker Image') {
            steps {
                script {
                    def fullImageName = "${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    buildAndPushImage(fullImageName, DOCKERHUB_CREDENTIALS_ID)
                }
            }
        }

        stage('Edit new image in deployment.yaml file') {
            steps {
                script {
                    dir('openshift') {
                        def fullImageName = "${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                        editNewImage(fullImageName)
                    }
                }
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                script {
                    deployToOpenShift(OPENSHIFT_CREDENTIALS_ID, OPENSHIFT_PROJECT, CLUSTER_URL)
                }
            }
        }
    }

    post {
        success {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline succeeded"
        }
        failure {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline failed"
        }
    }
}
   
    )
   ```

#### Step 6: Run the Pipeline

1. **Trigger the Pipeline**:
    - Go to the pipeline job and click "Build Now".
    - Monitor the pipeline execution in the console output.
![](https://github.com/omaRouby/Jenkins2-ivolve/blob/main/pictures/stages.png)
### Detailed Explanation

1. **DockerHub**:
    - The pipeline logs into DockerHub using stored credentials, builds a Docker image for the Python application, and pushes it to the DockerHub repository.

2. **OpenShift Deployment**:
    - The pipeline logs into the OpenShift cluster using stored credentials, imports the Docker image from DockerHub, and triggers a deployment.

### The Pipeline Output
- ![](https://github.com/omaRouby/Jenkins2-ivolve/blob/main/pictures/pipeline-success.png)
- **Navigate TO The Route ULR Created By OpenShift**
- ![](https://github.com/omaRouby/Jenkins2-ivolve/blob/main/pictures/routes.png)
- ![](https://github.com/omaRouby/Jenkins2-ivolve/blob/main/pictures/route-page.png)

By following these steps, you can set up a robust CI/CD pipeline using Jenkins to build, push, and deploy a Python application to an OpenShift cluster, leveraging a shared Jenkins library for reusability and maintainability.
