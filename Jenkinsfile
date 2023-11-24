pipeline {
    agent any
    environment {
        GCR_CREDENTIALS_ID = 'jenkins-gcr'
        IMAGE_NAME = 'claire-test-build-1'
        GCR_URL = 'gcr.io/lbg-mea-15'
    }
    stages {
        stage('Build and push to GCR') {
            steps {
                //Authenticate with Google Cloud
                withCredentials([file(credentialsId: GCR_CREDENTIALS_ID, variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                }
                //COnfigure Docker to use gcloud as a credential helper
                sh 'gcloud auth configure-docker --quiet'
                //Build the Docker image
                sh "docker build -t ${GCR_URL}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                //PUsh the Docker image to GCR
                sh "docker push ${GCR_URL}/${IMAGE_NAME}:${BUILD_NUMBER}"
            }

        }
    }
}
