pipeline {
    agent any
    environment {
        GCR_CREDENTIALS_ID = 'jenkins-gcr'
        IMAGE_NAME = 'claire-test-build-1'
        GCR_URL = 'gcr.io/lbg-mea-15'
        PROJECT_ID = 'lbg-mea-15'
        CLUSTER_NAME = 'claire-cluster'
        LOCATION = 'europe-west2-c'
        CREDENTIALS_ID = '4633d7f5-9cbb-46f5-9e2c-554859fa2e5a'
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
        stage('Staging Deploy to GKE'){
            steps {
                script {
                    //deploy to GKE using Jenkins K8 Engine Plugin
                    step([
                        $class: 'KubernetesEngineBuilder', 
                        projectId: env.PROJECT_ID, 
                        clusterName: env.CLUSTER_NAME, 
                        location: env.LOCATION, 
                        namespace: 'staging', 
                        manifestPattern: 'kubernetes/deployment.yaml', 
                        credentialsId: env.CREDENTIALS_ID, 
                        verifyDeployments: true])
                }
            }
        }
        stage('Quality Check') {
            steps {
                sh '''
                sleep 50
                gcloud config set account claire-jenkins@lbg-mea-15.iam.gserviceaccount.com
                // gcloud config set account 549579320723-compute@developer.gserviceaccount.com
                export STAGING_IP=\$(kubectl get svc -o json --namespace staging | jq '.items[] | select(.metadata.name == "flask-service") | .status.loadBalancer.ingress[0].ip' | tr -d '"')
                pip3 install requests
                python3 lbg.test.py
                '''
            }
        }


        stage('Deploy to GKE'){
            steps {
                script {
                    //deploy to GKE using Jenkins K8 Engine Plugin
                    step([
                        $class: 'KubernetesEngineBuilder', 
                        projectId: env.PROJECT_ID, 
                        clusterName: env.CLUSTER_NAME,
                        location: env.LOCATION, 
                        namespace: 'prod', 
                        manifestPattern: 'kubernetes/deployment.yaml', 
                        credentialsId: env.CREDENTIALS_ID, 
                        verifyDeployments: true])
                }
            }
        }
    }
}
