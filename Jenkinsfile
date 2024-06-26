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